# Authorization (1): General Authorization and Route Protection

So far, all of your application's routes are accessible by everyone. It doesn't matter whether the user is authenticated or not authenticated. For instance, when you sign out on the home or account page, there is no redirect, even though these pages should be only accessible for authenticated users. There is no reason to show a non authenticated user the account or home page in the first place, because these are the places where a user accesses sensitive information. In this section, so you will implement a protection for these routes called authorization. The protection is a **broad-grained authorization**, which checks for authenticated users. If none is present, it redirects from a protected to a public route; else, it will do nothing. The condition is defined as:

{title="Code Playground",lang="javascript"}
~~~~~~~
const condition = authUser => authUser != null;

// short version
const condition = authUser => !!authUser;
~~~~~~~

In contrast, a more **fine-grained authorization** could be a role-based or permission-based authorization:

{title="Code Playground",lang="javascript"}
~~~~~~~
// role-based authorization
const condition = authUser => authUser.role === 'ADMIN';

// permission-based authorization
const condition = authUser => authUser.permissions.canEditAccount;
~~~~~~~

Fortunately, we implement it in a way that lets you define the authorization condition (predicate) with flexibility, so that you can use a more generalized authorization rule, permission-based or role-based authorizations.

Like the `withAuthentication` higher-order component, there is a `withAuthorization` higher-order component to shield the authorization business logic from your components. It can be used on any component that needs to be protected with authorization (e.g. home page, account page). Let's start to add the higher-order component in a new file:

{title="src/components/Session/withAuthorization.js",lang="javascript"}
~~~~~~~
import React from 'react';

const withAuthorization = () => Component => {
  class WithAuthorization extends React.Component {
    render() {
      return <Component {...this.props} />;
    }
  }

  return WithAuthorization;
};

export default withAuthorization;
~~~~~~~

So far, the higher-order component is not doing anything but taking a component as input and returning it as output. However, the higher-order component should be able to receive a condition function passed as parameter. You can decide if it should be a broad or fine-grained (role-based, permission-based) authorization rule. Second, it has to decide based on the condition whether it should redirect to a public page (public route), because the user isn't authorized to view the current protected page (protected route). Let's paste the implementation details for the higher-order component and go through it step-by-step:

{title="src/components/Session/withAuthorization.js",lang="javascript"}
~~~~~~~
import React from 'react';
# leanpub-start-insert
import { withRouter } from 'react-router-dom';
import { compose } from 'recompose';
# leanpub-end-insert

# leanpub-start-insert
import { withFirebase } from '../Firebase';
import * as ROUTES from '../../constants/routes';
# leanpub-end-insert

# leanpub-start-insert
const withAuthorization = condition => Component => {
# leanpub-end-insert
  class WithAuthorization extends React.Component {
# leanpub-start-insert
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
# leanpub-end-insert

    render() {
      return (
        <Component {...this.props} />
      );
    }
  }

# leanpub-start-insert
  return compose(
    withRouter,
    withFirebase,
  )(WithAuthorization);
# leanpub-end-insert
};

export default withAuthorization;
~~~~~~~

The render method displays the passed component (e.g. home page, account page) that should be protected by this higher-order component. We will refine this later. The real authorization logic happens in the `componentDidMount()` lifecycle method. Like the `withAuthentication()` higher-order component, it uses the Firebase listener to trigger a callback function every time the authenticated user changes. The authenticated user is either a `authUser` object or `null`. Within this function, the passed `condition()` function is executed with the `authUser`. If the authorization fails, for instance because the authenticated user is `null`, the higher-order component redirects to the sign in page. If it doesn't fail, the higher-order component does nothing and renders the passed component (e.g. home page, account page). To redirect a user, the higher-order component has access to the history object of the Router using the in-house `withRouter()` higher-order component from the React Router library.

Remember to export the higher-order component from your session module:

{title="src/components/Session/index.js",lang="javascript"}
~~~~~~~
import AuthUserContext from './context';
import withAuthentication from './withAuthentication';
# leanpub-start-insert
import withAuthorization from './withAuthorization';
# leanpub-end-insert

# leanpub-start-insert
export { AuthUserContext, withAuthentication, withAuthorization };
# leanpub-end-insert
~~~~~~~

In the next step, you can use the higher-order component to protect your routes (e.g. /home and /account) with authorization rules using the passed `condition()` function.  To keep it simple, the following two components are only protected with a broad authorization rule that checks if the `authUser` is not `null`. First, enhance the HomePage component with the higher-order component and define the authorization condition for it:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
import React from 'react';

# leanpub-start-insert
import { withAuthorization } from '../Session';
# leanpub-end-insert

const HomePage = () => (
  <div>
    <h1>Home Page</h1>
# leanpub-start-insert
    <p>The Home Page is accessible by every signed in user.</p>
# leanpub-end-insert
  </div>
);

# leanpub-start-insert
const condition = authUser => !!authUser;
# leanpub-end-insert

# leanpub-start-insert
export default withAuthorization(condition)(HomePage);
# leanpub-end-insert
~~~~~~~

Second, enhance the AccountPage component with the higher-order component and define the authorization condition. It similar to the previous usage:

{title="src/components/Account/index.js",lang="javascript"}
~~~~~~~
import React from 'react';

import { PasswordForgetForm } from '../PasswordForget';
import PasswordChangeForm from '../PasswordChange';
# leanpub-start-insert
import { withAuthorization } from '../Session';
# leanpub-end-insert

const AccountPage = () => (
  <div>
    <h1>Account Page</h1>
    <PasswordForgetForm />
    <PasswordChangeForm />
  </div>
);

# leanpub-start-insert
const condition = authUser => !!authUser;
# leanpub-end-insert

# leanpub-start-insert
export default withAuthorization(condition)(AccountPage);
# leanpub-end-insert
~~~~~~~

The protection of both pages/routes is almost done. One refinement can be made in the `withAuthorization` higher-order component using the authenticated user from the context:

{title="src/components/Session/withAuthorization.js",lang="javascript"}
~~~~~~~
import React from 'react';
import { withRouter } from 'react-router-dom';
import { compose } from 'recompose';

# leanpub-start-insert
import AuthUserContext from './context';
# leanpub-end-insert
import { withFirebase } from '../Firebase';
import * as ROUTES from '../../constants/routes';

const withAuthorization = condition => Component => {
  class WithAuthorization extends React.Component {
    componentDidMount() {
      this.listener = this.props.firebase.auth.onAuthStateChanged(authUser => {
        if (!condition(authUser)) {
          this.props.history.push(ROUTES.SIGN_IN);
        }
      });
    }

    componentWillUnmount() {
      this.listener();
    }

    render() {
      return (
# leanpub-start-insert
        <AuthUserContext.Consumer>
          {authUser =>
            condition(authUser) ? <Component {...this.props} /> : null
          }
        </AuthUserContext.Consumer>
# leanpub-end-insert
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

The improvement in the render method was needed to avoid showing the protected page before the redirect happens. You want to show nothing if the authenticated user doesn't meet the condition's criteria. Then it's fine if the listener is too late to redirect the user, because the higher-order component didn't show the protected component.

Both routes are protected now, so we can render properties of the authenticated user in the AccountPage component without a null check for the authenticated user. You know the user should be there, otherwise the higher-order component would redirect to a public route.

{title="src/components/Account/index.js",lang="javascript"}
~~~~~~~
import React from 'react';

# leanpub-start-insert
import { AuthUserContext, withAuthorization } from '../Session';
# leanpub-end-insert
import { PasswordForgetForm } from '../PasswordForget';
import PasswordChangeForm from '../PasswordChange';

const AccountPage = () => (
# leanpub-start-insert
  <AuthUserContext.Consumer>
    {authUser => (
# leanpub-end-insert
      <div>
# leanpub-start-insert
        <h1>Account: {authUser.email}</h1>
# leanpub-end-insert
        <PasswordForgetForm />
        <PasswordChangeForm />
      </div>
# leanpub-start-insert
    )}
  </AuthUserContext.Consumer>
# leanpub-end-insert
);

const condition = authUser => !!authUser;

export default withAuthorization(condition)(AccountPage);
~~~~~~~

You can try it by signing out from your application and trying to access the */account* or */home* routes. Both should redirect you to the */signin* route. It should also redirect you automatically when you stay on one of the routes while you sign out.

You can imagine how this technique gives control over authorizations, not just by broader authorization rules, but more specific role-based and permission-based authorizations. For instance, an admin page available for users with the admin role could be protected as follows:

{title="Code Playground",lang="javascript"}
~~~~~~~
import React from 'react';

import * as ROLES from '../../constants/roles';

const AdminPage = () => (
  <div>
    <h1>Admin</h1>
    <p>
      Restricted area! Only users with the admin role are authorized.
    </p>
  </div>
);

const condition = authUser =>
  authUser && !!authUser.roles[ROLES.ADMIN];

export default withAuthorization(condition)(AdminPage);
~~~~~~~

Don't worry about this yet, because we'll implement a role-based authorization for this application later. For now, you have successfully implemented a full-fledged authentication mechanisms with Firebase in React, added neat features such as password reset and password change, and protected routes with dynamic authorization conditions.

### Exercises:

* Research yourself how a role-based or permission-based authorization could be implemented.
* Confirm your [source code for the last section](http://bit.ly/2Vop4SL)
