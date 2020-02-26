# Password Management

Let's take a step back from the higher-order components, React Context API, and session handling. In this section, we will implement two additional features available in the Firebase authentication API, the ability to retrieve (password forget) and change a password.

## Password Forget

Let's start by implementing the password forget feature. Since you already implemented the interface in your Firebase class, you can use it in components. The following file adds most of the password reset logic in a form again. We already used a couple of those forms before, so it shouldn't be different now. Add this in the *src/components/PasswordForget/index.js* file:

{title="src/components/PasswordForget/index.js",lang="javascript"}
~~~~~~~
import React, { Component } from 'react';
import { Link } from 'react-router-dom';

import { withFirebase } from '../Firebase';
import * as ROUTES from '../../constants/routes';

const PasswordForgetPage = () => (
  <div>
    <h1>PasswordForget</h1>
    <PasswordForgetForm />
  </div>
);

const INITIAL_STATE = {
  email: '',
  error: null,
};

class PasswordForgetFormBase extends Component {
  constructor(props) {
    super(props);

    this.state = { ...INITIAL_STATE };
  }

  onSubmit = event => {
    const { email } = this.state;

    this.props.firebase
      .doPasswordReset(email)
      .then(() => {
        this.setState({ ...INITIAL_STATE });
      })
      .catch(error => {
        this.setState({ error });
      });

    event.preventDefault();
  };

  onChange = event => {
    this.setState({ [event.target.name]: event.target.value });
  };

  render() {
    const { email, error } = this.state;

    const isInvalid = email === '';

    return (
      <form onSubmit={this.onSubmit}>
        <input
          name="email"
          value={this.state.email}
          onChange={this.onChange}
          type="text"
          placeholder="Email Address"
        />
        <button disabled={isInvalid} type="submit">
          Reset My Password
        </button>

        {error && <p>{error.message}</p>}
      </form>
    );
  }
}

const PasswordForgetLink = () => (
  <p>
    <Link to={ROUTES.PASSWORD_FORGET}>Forgot Password?</Link>
  </p>
);

export default PasswordForgetPage;

const PasswordForgetForm = withFirebase(PasswordForgetFormBase);

export { PasswordForgetForm, PasswordForgetLink };
~~~~~~~

The code is verbose, but it it's no different from the sign up and sign in forms from previous sections. The password forget uses a form to submit the information (email address) needed by the Firebase authentication API to reset the password. A class method (onSubmit) ensures the information is send to the API. It also resets the form's input field on a successful request, and shows an error on an erroneous request. The form is validated before it is submitted as well. The file implements a password forget link as a component which isn't used directly in the form component. It is similar to the SignUpLink component that we used on in the SignInPage component. This link is the same, and it's still usable. If a user forgets the password after sign up, the password forget page uses the link in the *src/components/SignIn/index.js* file:

{title="src/components/SignIn/index.js",lang="javascript"}
~~~~~~~
import React, { Component } from 'react';
import { withRouter } from 'react-router-dom';
import { compose } from 'recompose';

import { SignUpLink } from '../SignUp';
# leanpub-start-insert
import { PasswordForgetLink } from '../PasswordForget';
# leanpub-end-insert
import { withFirebase } from '../Firebase';
import * as ROUTES from '../../constants/routes';

const SignInPage = () => (
  <div>
    <h1>SignIn</h1>
    <SignInForm />
# leanpub-start-insert
    <PasswordForgetLink />
# leanpub-end-insert
    <SignUpLink />
  </div>
);


...
~~~~~~~

The password forget page is already matched in the App component, so you can drop the PasswordForgetLink component in the sign in page and know the mapping between route and component is complete. Start the application and reset your password. It doesn't matter if you are authenticated or not. Once you send the request, you should get an email from Firebase to update your password.

## Password Change

Next we'll add the password change feature, which is also in your Firebase interface. You only need a form component to use it. Again, the form component isn't any different from the sign in, sign up, and password forget forms. In the *src/components/PasswordChange/index.js* file add the following component:

{title="src/components/PasswordChange/index.js",lang="javascript"}
~~~~~~~
import React, { Component } from 'react';

import { withFirebase } from '../Firebase';

const INITIAL_STATE = {
  passwordOne: '',
  passwordTwo: '',
  error: null,
};

class PasswordChangeForm extends Component {
  constructor(props) {
    super(props);

    this.state = { ...INITIAL_STATE };
  }

  onSubmit = event => {
    const { passwordOne } = this.state;

    this.props.firebase
      .doPasswordUpdate(passwordOne)
      .then(() => {
        this.setState({ ...INITIAL_STATE });
      })
      .catch(error => {
        this.setState({ error });
      });

    event.preventDefault();
  };

  onChange = event => {
    this.setState({ [event.target.name]: event.target.value });
  };

  render() {
    const { passwordOne, passwordTwo, error } = this.state;

    const isInvalid =
      passwordOne !== passwordTwo || passwordOne === '';

    return (
      <form onSubmit={this.onSubmit}>
        <input
          name="passwordOne"
          value={passwordOne}
          onChange={this.onChange}
          type="password"
          placeholder="New Password"
        />
        <input
          name="passwordTwo"
          value={passwordTwo}
          onChange={this.onChange}
          type="password"
          placeholder="Confirm New Password"
        />
        <button disabled={isInvalid} type="submit">
          Reset My Password
        </button>

        {error && <p>{error.message}</p>}
      </form>
    );
  }
}

export default withFirebase(PasswordChangeForm);
~~~~~~~

The component updates its local state using `onChange` handlers in the input fields. It validates the state before submitting a request to change the password by enabling or disabling the submit button, and it shows again an error message when a request fails.

So far, the PasswordChangeForm is not matched by any route, because it should live on the Account page. The Account page could serve as the central place for users to manage their account, where it shows the PasswordChangeForm and PasswordResetForm, accessible by a standalone route. You already created the *src/components/Account/index.js* file and matched the route in the App component. You only need to implement it:

{title="src/components/Account/index.js",lang="javascript"}
~~~~~~~
import React from 'react';

import { PasswordForgetForm } from '../PasswordForget';
import PasswordChangeForm from '../PasswordChange';

const AccountPage = () => (
  <div>
    <h1>Account Page</h1>
    <PasswordForgetForm />
    <PasswordChangeForm />
  </div>
);

export default AccountPage;
~~~~~~~

The Account page doesn't have any business logic. It uses the password forget and password change forms in a central place. In this section, your user experience improved significantly with the password forget and password change features, handling scenarios where users have trouble remembering passwords.

### Exercises:

* Consider ways to protect the Account page and make it accessible only for authenticated users.
* Confirm your [source code for the last section](http://bit.ly/2VqWgt4)