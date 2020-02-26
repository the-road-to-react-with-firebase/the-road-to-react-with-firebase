# Authorization (2): Roles and Permissions

So far, you've used broad authorization rules that check user authentication, where the dedicated authorization higher-order component redirects them to the login page if the user is not authenticated. In this section, you will apply a more specific authorization mechanism. It works with roles (e.g. Admin, Author) assigned to a user, but also with permissions (e.g. user is allowed to write an article). Again, if the user doesn't fit a role for the authorization condition in your authorization higher-order component, the user will be redirected. Let's revisit the *src/components/Session/withAuthorization.js* higher-order component we have implemented thus far:

{title="src/components/Session/withAuthorization.js",lang="javascript"}
~~~~~~~
import React from 'react';
import { withRouter } from 'react-router-dom';
import { compose } from 'recompose';

import AuthUserContext from './context';
import { withFirebase } from '../Firebase';
import * as ROUTES from '../../constants/routes';

const withAuthorization = condition => Component => {
  class WithAuthorization extends React.Component {
    componentDidMount() {
      this.listener = this.props.firebase.auth.onAuthStateChanged(
        authUser => {
          if (!condition(authUser)) {
            this.props.history.push(ROUTES.SIGN_IN);
          }
        },
      );
    }

    componentWillUnmount() {
      this.listener();
    }

    render() {
      return (
        <AuthUserContext.Consumer>
          {authUser =>
            condition(authUser) ? <Component {...this.props} /> : null
          }
        </AuthUserContext.Consumer>
      );
    }
  }

  return compose(
    withRouter,
    withFirebase,
  )(WithAuthorization);
};

export default withAuthorization;
~~~~~~~

The Firebase listener always gives us the recent authenticated user. We don't know if the user is null though, so we deployed broader authorization rules for a few routes (e.g. */home*) of the application. If a user is not authenticated, we redirect the user to the login page.

{title="Code Playground",lang="javascript"}
~~~~~~~
const condition = authUser => !!authUser;
~~~~~~~

Now we can go a step further and check for a user's role or permission:

{title="Code Playground",lang="javascript"}
~~~~~~~
const condition = authUser =>
  authUser && !!authUser.roles[ROLES.ADMIN];
~~~~~~~

Assigning properties like an object of roles to authenticated users is a straightforward task. However, as we learned in previous sections, authenticated users are managed internally by Firebase. We are not able to alter user properties, so we manage them in Firebase's realtime database. If you go to your Firebase project's dashboard, you can see that users are managed in the Authentication and Database tabs. We introduced the latter to keep track of the users and assign them additional properties ourselves.

This section is split up into three parts:

* Assign a user on sign up a roles (e.g. admin role) property.
* Merge the authenticated user and database user so they can be authorized with their roles in the authorization higher-order component.
* Showcase a role authorization for one of our routes (e.g. only allowed for admin users).

Firebase has an official way to introduce roles to your authenticated user. I am not very comfortable with it, though, because it uses lots of Firebase internals and increases the effect of vendor lock-ins. Instead, I prefer storing roles directly into the user entities in the Firebase database. This way, you'll have an easier time migrating away from Firebase if you decide to roll out a backend application with a database.

## Database Users with Roles

We'll use multiple roles because users may have more than one role in the application or system. For instance, a user could be an admin, but also an author with access to admin and author areas. Let's assign a `roles` property to our users when they are created in the realtime database on sign-up. First, we'll track a checkbox state for this type of role in our component:

{title="src/components/SignUp/index.js",lang="javascript"}
~~~~~~~
const INITIAL_STATE = {
  username: '',
  email: '',
  passwordOne: '',
  passwordTwo: '',
# leanpub-start-insert
  isAdmin: false,
# leanpub-end-insert
  error: null,
};
~~~~~~~

Next, add a checkbox to toggle the user role in the UI:

{title="src/components/SignUp/index.js",lang="javascript"}
~~~~~~~
class SignUpFormBase extends Component {

  ...

# leanpub-start-insert
  onChangeCheckbox = event => {
    this.setState({ [event.target.name]: event.target.checked });
  };
# leanpub-end-insert

  render() {
    const {
      username,
      email,
      passwordOne,
      passwordTwo,
# leanpub-start-insert
      isAdmin,
# leanpub-end-insert
      error,
    } = this.state;

    ...

    return (
      <form onSubmit={this.onSubmit}>
        ...
# leanpub-start-insert
        <label>
          Admin:
          <input
            name="isAdmin"
            type="checkbox"
            checked={isAdmin}
            onChange={this.onChangeCheckbox}
          />
        </label>
# leanpub-end-insert
        <button disabled={isInvalid} type="submit">
          Sign Up
        </button>

        {error && <p>{error.message}</p>}
      </form>
    );
  }
}

...
~~~~~~~

We don't want to grant any user the power to sign up as admin, but we'll keep it simple for now, and you can decide which circumstances prompt you to assign roles to users later. Next, add the roles property to your user when they are created in the database. Since we need an object of roles, we'll initialize it as an empty object and add conditional roles to it:

{title="src/components/SignUp/index.js",lang="javascript"}
~~~~~~~
import React, { Component } from 'react';
import { Link, withRouter } from 'react-router-dom';
import { compose } from 'recompose';

import { withFirebase } from '../Firebase';
import * as ROUTES from '../../constants/routes';
# leanpub-start-insert
import * as ROLES from '../../constants/roles';
# leanpub-end-insert

...

class SignUpFormBase extends Component {
  ...

  onSubmit = event => {
# leanpub-start-insert
    const { username, email, passwordOne, isAdmin } = this.state;
    const roles = {};
# leanpub-end-insert

# leanpub-start-insert
    if (isAdmin) {
      roles[ROLES.ADMIN] = ROLES.ADMIN;
    }
# leanpub-end-insert

    this.props.firebase
      .doCreateUserWithEmailAndPassword(email, passwordOne)
      .then(authUser => {
        // Create a user in your Firebase realtime database
        return this.props.firebase
          .user(authUser.user.uid)
          .set({
            username,
            email,
# leanpub-start-insert
            roles,
# leanpub-end-insert
          })
      })
      .then(() => {
        this.setState({ ...INITIAL_STATE });
        this.props.history.push(ROUTES.HOME);
      })
      .catch(error => {
        this.setState({ error });
      });

    event.preventDefault();
  };

  ...
}

...
~~~~~~~

Next, collect all your roles in the *src/constants/roles.js* file we just imported in the previous step. It can be used to assign roles as you did before, but also to protect routes later on.

{title="src/constants/roles.js",lang="javascript"}
~~~~~~~
# leanpub-start-insert
export const ADMIN = 'ADMIN';
# leanpub-end-insert
~~~~~~~

You should be able to create users with admin privileges now. Next we'll cover how to merge this user from our database with the authenticated user from the Firebase authentication.

## How to merge authenticated user with database user?

Since we need to check the roles only in the authorization higher-order component, it's best to merge the authentication user and database user in this component before checking for its privileges (roles, permissions).

{title="src/components/Session/withAuthorization.js",lang="javascript"}
~~~~~~~
...

const withAuthorization = condition => Component => {
  class WithAuthorization extends React.Component {
    componentDidMount() {
      this.listener = this.props.firebase.auth.onAuthStateChanged(
        authUser => {
# leanpub-start-insert
          if (authUser) {
            this.props.firebase
              .user(authUser.uid)
              .once('value')
              .then(snapshot => {
                const dbUser = snapshot.val();

                // default empty roles
                if (!dbUser.roles) {
                  dbUser.roles = {};
                }

                // merge auth and db user
                authUser = {
                  uid: authUser.uid,
                  email: authUser.email,
                  ...dbUser,
                };

                if (!condition(authUser)) {
                  this.props.history.push(ROUTES.SIGN_IN);
                }
              });
          } else {
            this.props.history.push(ROUTES.SIGN_IN);
          }
# leanpub-end-insert
        },
      );
    }

    ...
  }

  ...
};

...
~~~~~~~

When the authenticated user changes, the function within the listener is called. If the user is null, it redirects. If the user is not null, we will get the database user with the help of the authenticated user's unique identifier, and then we merge everything from the database user with the unique identifier and email from the authenticated user. You may need more properties from the authenticated user later, but at this point the unique identifier and the email are sufficient. As we did before, we are running our conditional check to see if the user is authorized or not. However, now we have all the properties from the database user (e.g. roles) at our disposal. If the conditions are not met, we redirect the user. If the conditions are met, the user stays on the page component enhanced by the authorization higher-order component.

We've covered merging the user in the authorization higher-order component. Next, we'll consider the authentication higher-order component that provides authentication for all our components. Maybe we want to show something to the user based on their level of authentication like a link to admin page in the navigation. Let's implement the merging in this higher-order component, too:

{title="src/components/Session/withAuthentication.js",lang="javascript"}
~~~~~~~
...

const withAuthentication = Component => {
  class WithAuthentication extends React.Component {
    componentDidMount() {
      this.listener = this.props.firebase.auth.onAuthStateChanged(
        authUser => {
# leanpub-start-insert
          if (authUser) {
            this.props.firebase
              .user(authUser.uid)
              .once('value')
              .then(snapshot => {
                const dbUser = snapshot.val();

                // default empty roles
                if (!dbUser.roles) {
                  dbUser.roles = {};
                }

                // merge auth and db user
                authUser = {
                  uid: authUser.uid,
                  email: authUser.email,
                  ...dbUser,
                };

                this.setState({ authUser });
              });
          } else {
            this.setState({ authUser: null });
          }
# leanpub-end-insert
        },
      );
    }

    ...
  }

  ...
};

...
~~~~~~~

Now the authenticated user is provided via React's Context API extended with the database user to all our components. As you may have noticed, the implementation was quite repetitive to the authorization higher-order component. The only thing we changed was the local state usage instead of the redirects. Let's see how we can extract the common implementation to our Firebase class without touching the implementation details with the local state (authentication higher-order component) and the redirection (authorization higher-order component).

{title="src/components/Firebase/firebase.js",lang="javascript"}
~~~~~~~
...

class Firebase {
  ...

  // *** Auth API ***

  ...

# leanpub-start-insert
  // *** Merge Auth and DB User API *** //

  onAuthUserListener = (next, fallback) =>
    this.auth.onAuthStateChanged(authUser => {
      if (authUser) {
        this.user(authUser.uid)
          .once('value')
          .then(snapshot => {
            const dbUser = snapshot.val();

            // default empty roles
            if (!dbUser.roles) {
              dbUser.roles = {};
            }

            // merge auth and db user
            authUser = {
              uid: authUser.uid,
              email: authUser.email,
              ...dbUser,
            };

            next(authUser);
          });
      } else {
        fallback();
      }
    });
# leanpub-end-insert

  // *** User API ***

  ...
}

export default Firebase;
~~~~~~~

Observe which common implementation details were taken from the two higher-order components. The only thing changed in this extracted function are the `next()` and `fallback()` callback functions. This is where we can implement the specific implementation details of every higher-order component (local state for authentication, redirect for authorization) that uses this new method. Let's use this function first in the authentication higher-order component and provide both callback functions as "next" and "fallback":

{title="src/components/Session/withAuthentication.js",lang="javascript"}
~~~~~~~
...

const withAuthentication = Component => {
  class WithAuthentication extends React.Component {
    ...

    componentDidMount() {
# leanpub-start-insert
      this.listener = this.props.firebase.onAuthUserListener(
        authUser => {
          this.setState({ authUser });
        },
        () => {
          this.setState({ authUser: null });
        },
      );
# leanpub-end-insert
    }

    ...
  }

  ...
};

...
~~~~~~~

The higher-order component uses this new method from the Firebase instance, providing only two callback functions as parameters. The first callback function is used for the merged user; the second is used when the authenticated user is null. Let's use the new method in the authorization higher-order component as well:

{title="src/components/Session/withAuthorization.js",lang="javascript"}
~~~~~~~
...

const withAuthorization = condition => Component => {
  class WithAuthorization extends React.Component {
    ...

    componentDidMount() {
# leanpub-start-insert
      this.listener = this.props.firebase.onAuthUserListener(
        authUser => {
          if (!condition(authUser)) {
            this.props.history.push(ROUTES.SIGN_IN);
          }
        },
        () => this.props.history.push(ROUTES.SIGN_IN),
      );
# leanpub-end-insert
    }

    ...
  }

  ...
};

...
~~~~~~~

We extracted the business logic that merges authentication user and database user to the Firebase class, made it available as method on the class instance, and used it in both higher-order components. Since we can provide two callback functions to this new method, which are executed based on whether the user is authenticated or not, each higher-order component gains control over what happens afterward. The authorization higher-order component cares about redirects, while the authentication higher-order component cares about storing the merged user in the local state and distributing it to other components via React's Context Provider component.

## Authorize a Firebase User based on a Role

Now we need to check whether our role-based authorization is working after all. The admin page is the best place to deploy a role-based authorization, because so far it is open for everyone and not secured.

{title="src/components/Admin/index.js",lang="javascript"}
~~~~~~~
import React, { Component } from 'react';
# leanpub-start-insert
import { compose } from 'recompose';
# leanpub-end-insert

import { withFirebase } from '../Firebase';
# leanpub-start-insert
import { withAuthorization } from '../Session';
import * as ROLES from '../../constants/roles';
# leanpub-end-insert

class AdminPage extends Component {
  ...

  render() {
    const { users, loading } = this.state;

    return (
      <div>
        <h1>Admin</h1>
# leanpub-start-insert
        <p>
          The Admin Page is accessible by every signed in admin user.
        </p>
# leanpub-end-insert

        {loading && <div>Loading ...</div>}

        <UserList users={users} />
      </div>
    );
  }
}

...

# leanpub-start-insert
const condition = authUser =>
  authUser && !!authUser.roles[ROLES.ADMIN];
# leanpub-end-insert

# leanpub-start-insert
export default compose(
  withAuthorization(condition),
  withFirebase,
)(AdminPage);
# leanpub-end-insert
~~~~~~~

If you try to access the admin page as non-authenticated user or as non-admin user without the necessary role, it redirects you to the login page. If you access it as user with admin privileges, you should be able to see the desired content. The authorization higher-order component made this possible.

Next, we'll secure the route to the admin page in the Navigation component. That's where the (merged) authenticated user from the authentication higher-order component comes in. Using React's Context in this higher-order component, you can access the extended authenticated user:

{title="src/components/Navigation/index.js",lang="javascript"}
~~~~~~~
...

import { AuthUserContext } from '../Session';
import SignOutButton from '../SignOut';
import * as ROUTES from '../../constants/routes';
# leanpub-start-insert
import * as ROLES from '../../constants/roles';
# leanpub-end-insert

const Navigation = () => (
  <AuthUserContext.Consumer>
    {authUser =>
      authUser ? (
# leanpub-start-insert
        <NavigationAuth authUser={authUser} />
# leanpub-end-insert
      ) : (
        <NavigationNonAuth />
      )
    }
  </AuthUserContext.Consumer>
);

# leanpub-start-insert
const NavigationAuth = ({ authUser }) => (
# leanpub-end-insert
  <ul>
    <li>
      <Link to={ROUTES.LANDING}>Landing</Link>
    </li>
    <li>
      <Link to={ROUTES.HOME}>Home</Link>
    </li>
    <li>
      <Link to={ROUTES.ACCOUNT}>Account</Link>
    </li>
# leanpub-start-insert
    {!!authUser.roles[ROLES.ADMIN] && (
      <li>
        <Link to={ROUTES.ADMIN}>Admin</Link>
      </li>
    )}
# leanpub-end-insert
    <li>
      <SignOutButton />
    </li>
  </ul>
);

const NavigationNonAuth = () => (
  <ul>
    <li>
      <Link to={ROUTES.LANDING}>Landing</Link>
    </li>
    <li>
      <Link to={ROUTES.SIGN_IN}>Sign In</Link>
    </li>
  </ul>
);

export default Navigation;
~~~~~~~

You have successfully assigned roles to your users, merged authentication user with the database user in your higher-order components, and applied authorization rules with redirects and conditional rendering in your application. Everything that is added to the database user will be available when you launch your application, and you've set properties for the Firebase user.

### Exercises:

* Walk through a scenario where the role-based authorization could be replaced with a permission-based authorization (e.g. `authUser.permissions.canEditUser`).
* Confirm your [source code for the last section](http://bit.ly/2VneMT2)
