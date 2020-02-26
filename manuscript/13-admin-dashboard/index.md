# Admin Dashboard

Before we dive deeper into Firebase's realtime database and the domain-related business logic of our application, it makes sense to invest more time into React Router. So far, we have split up our application into top-level routes to manage our whole authentication flow with login, logout, and registration. Additionally, we protected top-level routes with authorization that checks for authenticated users, confirmed email addresses, and admin users.

In this section, we'll implement more specific routing for the admin page. So far, this page only shows a list of users, retrieved from the Firebase realtime database. Basically, it is the overview of our users. However, a list of users alone doesn't help that much, and a detail page would be much more useful. Then, it would be possible to trigger further actions for the user on the detail page instead of the overview page. To start, define a new child route:

{title="src/constants/routes.js",lang="javascript"}
~~~~~~~
export const LANDING = '/';
export const SIGN_UP = '/signup';
export const SIGN_IN = '/signin';
export const HOME = '/home';
export const ACCOUNT = '/account';
export const PASSWORD_FORGET = '/pw-forget';
export const ADMIN = '/admin';
# leanpub-start-insert
export const ADMIN_DETAILS = '/admin/:id';
# leanpub-end-insert
~~~~~~~

The `:id` is a placeholder for a user identifier to be used later. If you want to be more specific, you could have used `/admin/users/:id` as well. Perhaps later you'll want to manage other entities on this admin page. For instance, the admin page could have a list of users and a list of books written by them, where it would make sense to have detail pages for users (`/admin/users/:userId`) and books (`/admin/books/:bookId`).

Next, extract all the functionality from the AdminPage component. You will lift this business logic down to another component in the next step. In this step, introduce two sub routes for the admin page and match the UserList and UserItem components to it. The former component is already there, the latter component will be implemented soon.

{title="src/components/Admin/index.js",lang="javascript"}
~~~~~~~
import React, { Component } from 'react';
# leanpub-start-insert
import { Switch, Route, Link } from 'react-router-dom';
# leanpub-end-insert
import { compose } from 'recompose';

import { withFirebase } from '../Firebase';
import { withAuthorization, withEmailVerification } from '../Session';
import * as ROLES from '../../constants/roles';
# leanpub-start-insert
import * as ROUTES from '../../constants/routes';
# leanpub-end-insert

# leanpub-start-insert
const AdminPage = () => (
# leanpub-end-insert
  <div>
    <h1>Admin</h1>
    <p>The Admin Page is accessible by every signed in admin user.</p>

# leanpub-start-insert
    <Switch>
      <Route exact path={ROUTES.ADMIN_DETAILS} component={UserItem} />
      <Route exact path={ROUTES.ADMIN} component={UserList} />
    </Switch>
# leanpub-end-insert
  </div>
# leanpub-start-insert
);
# leanpub-end-insert
~~~~~~~

The UserList component receives all the business logic that was in the AdminPage. Also, it receives the `Base` suffix because we enhance it in the next step with a higher-order component to make the Firebase instance available.

{title="src/components/Admin/index.js",lang="javascript"}
~~~~~~~
# leanpub-start-insert
class UserListBase extends Component {
  constructor(props) {
    super(props);

    this.state = {
      loading: false,
      users: [],
    };
  }

  componentDidMount() {
    this.setState({ loading: true });

    this.props.firebase.users().on('value', snapshot => {
      const usersObject = snapshot.val();

      const usersList = Object.keys(usersObject).map(key => ({
        ...usersObject[key],
        uid: key,
      }));

      this.setState({
        users: usersList,
        loading: false,
      });
    });
  }

  componentWillUnmount() {
    this.props.firebase.users().off();
  }

  render() {
    ...
  }
}
# leanpub-end-insert
~~~~~~~

Further, the UserList component renders a Link component from the React Router package, which is used to navigate users from the user list (overview) to the user item (detail) route. The mapping for the route and the component was completed in the AdminPage component.

{title="src/components/Admin/index.js",lang="javascript"}
~~~~~~~
class UserListBase extends Component {
  ...

# leanpub-start-insert
  render() {
    const { users, loading } = this.state;
# leanpub-end-insert

# leanpub-start-insert
    return (
# leanpub-end-insert
      <div>
# leanpub-start-insert
        <h2>Users</h2>
        {loading && <div>Loading ...</div>}
# leanpub-end-insert
        <ul>
          {users.map(user => (
            <li key={user.uid}>
              <span>
                <strong>ID:</strong> {user.uid}
              </span>
              <span>
                <strong>E-Mail:</strong> {user.email}
              </span>
              <span>
                <strong>Username:</strong> {user.username}
              </span>
# leanpub-start-insert
              <span>
                <Link to={`${ROUTES.ADMIN}/${user.uid}`}>
                  Details
                </Link>
              </span>
# leanpub-end-insert
            </li>
          ))}
        </ul>
      </div>
# leanpub-start-insert
    );
  }
# leanpub-end-insert
}
~~~~~~~

Remember, the UserList receives access to the Firebase instance, and the AdminPage doesn't need it anymore.

{title="src/components/Admin/index.js",lang="javascript"}
~~~~~~~
...

const condition = authUser =>
  authUser && !!authUser.roles[ROLES.ADMIN];

# leanpub-start-insert
const UserList = withFirebase(UserListBase);
# leanpub-end-insert

export default compose(
  withEmailVerification,
  withAuthorization(condition),
)(AdminPage);
~~~~~~~

Last but not least, render a basic UserItem component.

{title="src/components/Admin/index.js",lang="javascript"}
~~~~~~~
...

# leanpub-start-insert
const UserItem = ({ match }) => (
  <div>
    <h2>User ({match.params.id})</h2>
  </div>
);
# leanpub-end-insert
~~~~~~~

You should be able to navigate from the user list (overview) to the user item (detail) component on the admin page now. We are fetching the user list on the admin page, without specific user data for a single user for the UserItem component on the detail perspective. The identifier for the user is available from the browser's URL through the React Router. You can extract it from the component's props to fetch a user from Firebase's realtime database:

{title="src/components/Admin/index.js",lang="javascript"}
~~~~~~~
# leanpub-start-insert
class UserItemBase extends Component {
  constructor(props) {
    super(props);

    this.state = {
      loading: false,
      user: null,
    };
  }

  componentDidMount() {
    this.setState({ loading: true });

    this.props.firebase
      .user(this.props.match.params.id)
      .on('value', snapshot => {
        this.setState({
          user: snapshot.val(),
          loading: false,
        });
      });
  }

  componentWillUnmount() {
    this.props.firebase.user(this.props.match.params.id).off();
  }

  render() {
    ...
  }
}
# leanpub-end-insert
~~~~~~~

Don't forget to make Firebase accessible in the props of the UserItem component again via our higher-order component:

{title="src/components/Admin/index.js",lang="javascript"}
~~~~~~~
...

const UserList = withFirebase(UserListBase);
# leanpub-start-insert
const UserItem = withFirebase(UserItemBase);
# leanpub-end-insert

...
~~~~~~~

Last but not least, render again the user information. This time it's not a whole list of users, but only a single user entity:

{title="src/components/Admin/index.js",lang="javascript"}
~~~~~~~
class UserItemBase extends Component {
  ...

  render() {
# leanpub-start-insert
    const { user, loading } = this.state;

    return (
      <div>
        <h2>User ({this.props.match.params.id})</h2>
        {loading && <div>Loading ...</div>}

        {user && (
          <div>
            <span>
              <strong>ID:</strong> {user.uid}
            </span>
            <span>
              <strong>E-Mail:</strong> {user.email}
            </span>
            <span>
              <strong>Username:</strong> {user.username}
            </span>
          </div>
        )}
      </div>
    );
# leanpub-end-insert
  }
}
~~~~~~~

When you navigate to a user detail perspective, you can see the id from the props is rendered immediately, because it's available from React Router to fetch user details from the Firebase database. However, since you already have the information about the user in the UserList component that links to your UserItem component, you can pass this information through React Router's Link:

{title="src/components/Admin/index.js",lang="javascript"}
~~~~~~~
class UserListBase extends Component {
  ...

  render() {
    const { users, loading } = this.state;

    return (
      <div>
        <h2>Users</h2>
        {loading && <div>Loading ...</div>}
        <ul>
          {users.map(user => (
            <li key={user.uid}>
              ...
              <span>
                <Link
# leanpub-start-insert
                  to={{
                    pathname: `${ROUTES.ADMIN}/${user.uid}`,
                    state: { user },
                  }}
# leanpub-end-insert
                >
                  Details
                </Link>
              </span>
            </li>
          ))}
        </ul>
      </div>
    );
  }
}
~~~~~~~

Then use it in the UserItem component as default local state:

{title="src/components/Admin/index.js",lang="javascript"}
~~~~~~~
class UserItemBase extends Component {
  constructor(props) {
    super(props);

    this.state = {
      loading: false,
      user: null,
# leanpub-start-insert
      ...props.location.state,
# leanpub-end-insert
    };
  }

  componentDidMount() {
# leanpub-start-insert
    if (this.state.user) {
      return;
    }
# leanpub-end-insert

    this.setState({ loading: true });

    this.props.firebase
      .user(this.props.match.params.id)
      .on('value', snapshot => {
        this.setState({
          user: snapshot.val(),
          loading: false,
        });
      });
  }

  ...
}
~~~~~~~

If users navigate from the UserList to the UserItem component, they should arrive immediately. If they enter the URL by hand in the browser or with a Link component that doesn't pass them to the UserItem component, the user needs to be fetched from the Firebase database. Since you have a page for each individual user on your admin dashboard now, you can add more specific actions. For instance, sometimes a user can't login and isn't sure how to proceed, which is the perfect time to send a reset password email to them as admin. Let's add a button to send a password reset email to a user.

{title="src/components/Admin/index.js",lang="javascript"}
~~~~~~~
class UserItemBase extends Component {
  ...

# leanpub-start-insert
  onSendPasswordResetEmail = () => {
    this.props.firebase.doPasswordReset(this.state.user.email);
  };
# leanpub-end-insert

  render() {
    const { user, loading } = this.state;

    return (
      <div>
        <h2>User ({this.props.match.params.id})</h2>
        {loading && <div>Loading ...</div>}

        {user && (
          <div>
            ...
            <span>
              <strong>Username:</strong> {user.username}
            </span>
# leanpub-start-insert
            <span>
              <button
                type="button"
                onClick={this.onSendPasswordResetEmail}
              >
                Send Password Reset
              </button>
            </span>
# leanpub-end-insert
          </div>
        )}
      </div>
    );
  }
}
~~~~~~~

Note: If you want to dig deeper into deleting users from Firebase's authentication, how to resend verification emails, or how to change email addresses, study Firebase's Admin SDK.

This section has shown you how to implement more specific routes with React Router and how to interact with the Firebase database on each individual route. You can also use React Router's advanced features to pass information as props to the other components like we did for the user.

### Exercises:

* Learn more about [React Router](https://reacttraining.com/react-router/web/guides/quick-start)
* Read more about [Firebase's Admin SDK](https://firebase.google.com/docs/auth/admin/)
* Confirm your [source code for the last section](http://bit.ly/2VnfIqw)