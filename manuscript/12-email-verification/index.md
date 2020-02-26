# Email Verification

In your application, users can employ an email/password combination, but also social logins to get access to your service or product. Often, the email address associated with the social logins is confirmed by the social platform (Google, Facebook, Twitter) and you know this email address really exists. But what about the email address used with the password? Because users are sometimes unwilling to provide real email addresses, they'll simply make one up, so you can't provide them with further information via email or to integrate them with third-parties where a valid email address is required. In this section, I will show you how to confirm user email addresses before they can access your application. After an email verification with a double opt-in send by email, users are authorized to use your application.

Because the Firebase API already provides this functionality, we can add it to our Firebase class to make it available for our React application. Provide an optional redirect URL that is used to navigate to the application after email confirmation:

{title="src/components/Firebase/firebase.js",lang="javascript"}
~~~~~~~
...

class Firebase {
  ...

  // *** Auth API ***

  ...

# leanpub-start-insert
  doSendEmailVerification = () =>
    this.auth.currentUser.sendEmailVerification({
      url: process.env.REACT_APP_CONFIRMATION_EMAIL_REDIRECT,
    });
# leanpub-end-insert

  ...
}

export default Firebase;
~~~~~~~

You can inline this URL, but also put it into your *.env* file(s). I prefer environment variables for development (*.env.development*) and production (*.env.production*). The development environment receives the localhost URL:

{title=".env.development",lang="javascript"}
~~~~~~~
...

# leanpub-start-insert
REACT_APP_CONFIRMATION_EMAIL_REDIRECT=http://localhost:3000
# leanpub-end-insert
~~~~~~~

And the production environment receives an actual domain:

{title=".env.production",lang="javascript"}
~~~~~~~
...

# leanpub-start-insert
REACT_APP_CONFIRMATION_EMAIL_REDIRECT=https://mydomain.com
# leanpub-end-insert
~~~~~~~

That's all we need to do for the API. The best place to guide users through the email verification is during email and password sign-up:

{title="src/components/SignUp/index.js",lang="javascript"}
~~~~~~~
...

class SignUpFormBase extends Component {
  ...

  onSubmit = event => {
    ...

    this.props.firebase
      .doCreateUserWithEmailAndPassword(email, passwordOne)
      .then(authUser => {
        // Create a user in your Firebase realtime database
        return this.props.firebase.user(authUser.user.uid).set({
          username,
          email,
          roles,
        });
      })
# leanpub-start-insert
      .then(() => {
        return this.props.firebase.doSendEmailVerification();
      })
# leanpub-end-insert
      .then(() => {
        this.setState({ ...INITIAL_STATE });
        this.props.history.push(ROUTES.HOME);
      })
      .catch(error => {
        ...
      });

    event.preventDefault();
  };

  ...
}

...
~~~~~~~

Users will receive a verification email when they register for your application. To find out if a user has a verified email, you can retrieve this information from the authenticated user in your Firebase class:

{title="src/components/Firebase/firebase.js",lang="javascript"}
~~~~~~~
...

class Firebase {
  ...

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
# leanpub-start-insert
              emailVerified: authUser.emailVerified,
              providerData: authUser.providerData,
# leanpub-end-insert
              ...dbUser,
            };

            next(authUser);
          });
      } else {
        fallback();
      }
    });

    ...
}

export default Firebase;
~~~~~~~

To protect your routes from users who have no verified email address, we will do it with a new higher-order component in *src/components/Session/withEmailVerification.js* that has access to Firebase and the authenticated user:

{title="src/components/Session/withEmailVerification.js",lang="javascript"}
~~~~~~~
import React from 'react';

import AuthUserContext from './context';
import { withFirebase } from '../Firebase';

const withEmailVerification = Component => {
  class WithEmailVerification extends React.Component {
    render() {
      return (
        <AuthUserContext.Consumer>
          {authUser => <Component {...this.props} />}
        </AuthUserContext.Consumer>
      );
    }
  }

  return withFirebase(WithEmailVerification);
};

export default withEmailVerification;
~~~~~~~

Add a function in this file that checks if the authenticated user has a verified email and an email/password sign in on associated with it. If the user has only social logins, it doesn't matter if the email is not verified.

{title="src/components/Session/withEmailVerification.js",lang="javascript"}
~~~~~~~
# leanpub-start-insert
const needsEmailVerification = authUser =>
  authUser &&
  !authUser.emailVerified &&
  authUser.providerData
    .map(provider => provider.providerId)
    .includes('password');
# leanpub-end-insert
~~~~~~~

If this is true, don't render the component passed to this higher-order component, but a message that reminds users to verify their email addresses.

{title="src/components/Session/withEmailVerification.js",lang="javascript"}
~~~~~~~
...

const withEmailVerification = Component => {
  class WithEmailVerification extends React.Component {
# leanpub-start-insert
    onSendEmailVerification = () => {
      this.props.firebase.doSendEmailVerification();
    }
# leanpub-end-insert

    render() {
      return (
        <AuthUserContext.Consumer>
          {authUser =>
# leanpub-start-insert
            needsEmailVerification(authUser) ? (
              <div>
                <p>
                  Verify your E-Mail: Check you E-Mails (Spam folder
                  included) for a confirmation E-Mail or send
                  another confirmation E-Mail.
                </p>

                <button
                  type="button"
                  onClick={this.onSendEmailVerification}
                >
                  Send confirmation E-Mail
                </button>
              </div>
            ) : (
              <Component {...this.props} />
            )
# leanpub-end-insert
          }
        </AuthUserContext.Consumer>
      );
    }
  }

  return withFirebase(WithEmailVerification);
};

export default withEmailVerification;
~~~~~~~

Optionally, we can provide a button to resend a verification email to the user. Let's improve the user experience. After the button is clicked to resend the verification email, users should receive feedback, and be prohibited from sending another email. First, add a local state to the higher-order component that tracks whether the button was clicked:

{title="src/components/Session/withEmailVerification.js",lang="javascript"}
~~~~~~~
...

const withEmailVerification = Component => {
  class WithEmailVerification extends React.Component {
# leanpub-start-insert
    constructor(props) {
      super(props);

      this.state = { isSent: false };
    }
# leanpub-end-insert

    onSendEmailVerification = () => {
      this.props.firebase
        .doSendEmailVerification()
# leanpub-start-insert
        .then(() => this.setState({ isSent: true }));
# leanpub-end-insert
    };

    ...
  }

  return withFirebase(WithEmailVerification);
};

export default withEmailVerification;
~~~~~~~

Second, show another message with a conditional rendering if a user has sent another verification email:

{title="src/components/Session/withEmailVerification.js",lang="javascript"}
~~~~~~~
...

const withEmailVerification = Component => {
  class WithEmailVerification extends React.Component {

    ...

    render() {
      return (
        <AuthUserContext.Consumer>
          {authUser =>
            needsEmailVerification(authUser) ? (
              <div>
# leanpub-start-insert
                {this.state.isSent ? (
                  <p>
                    E-Mail confirmation sent: Check you E-Mails (Spam
                    folder included) for a confirmation E-Mail.
                    Refresh this page once you confirmed your E-Mail.
                  </p>
                ) : (
# leanpub-end-insert
                  <p>
                    Verify your E-Mail: Check you E-Mails (Spam folder
                    included) for a confirmation E-Mail or send
                    another confirmation E-Mail.
                  </p>
# leanpub-start-insert
                )}
# leanpub-end-insert

                <button
                  type="button"
                  onClick={this.onSendEmailVerification}
# leanpub-start-insert
                  disabled={this.state.isSent}
# leanpub-end-insert
                >
                  Send confirmation E-Mail
                </button>
              </div>
            ) : (
              <Component {...this.props} />
            )
          }
        </AuthUserContext.Consumer>
      );
    }
  }

  return withFirebase(WithEmailVerification);
};

export default withEmailVerification;
~~~~~~~

Lastly, make the new higher-order component available in your Session folder's *index.js* file:

{title="src/components/Session/index.js",lang="javascript"}
~~~~~~~
import AuthUserContext from './context';
import withAuthentication from './withAuthentication';
import withAuthorization from './withAuthorization';
# leanpub-start-insert
import withEmailVerification from './withEmailVerification';
# leanpub-end-insert

export {
  AuthUserContext,
  withAuthentication,
  withAuthorization,
# leanpub-start-insert
  withEmailVerification,
# leanpub-end-insert
};
~~~~~~~

Send a confirmation email once a user signs up with a email/password combination. You also have a higher-order component used for authorization and optionally resending a confirmation email. Next, secure all pages/routes that should be only accessible with a confirmed email. Let's begin with the home page:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
import React from 'react';
# leanpub-start-insert
import { compose } from 'recompose';
# leanpub-end-insert

# leanpub-start-insert
import { withAuthorization, withEmailVerification } from '../Session';
# leanpub-end-insert

const HomePage = () => (
  <div>
    <h1>Home Page</h1>
    <p>The Home Page is accessible by every signed in user.</p>
  </div>
);

const condition = authUser => !!authUser;

# leanpub-start-insert
export default compose(
  withEmailVerification,
  withAuthorization(condition),
)(HomePage);
# leanpub-end-insert
~~~~~~~

Next the admin page:

{title="src/components/Admin/index.js",lang="javascript"}
~~~~~~~
import React, { Component } from 'react';
import { compose } from 'recompose';

import { withFirebase } from '../Firebase';
# leanpub-start-insert
import { withAuthorization, withEmailVerification } from '../Session';
# leanpub-end-insert
import * as ROLES from '../../constants/roles';

...

const condition = authUser =>
  authUser && !!authUser.roles[ROLES.ADMIN];

export default compose(
# leanpub-start-insert
  withEmailVerification,
# leanpub-end-insert
  withAuthorization(condition),
  withFirebase,
)(AdminPage);
~~~~~~~

And the account page:

{title="src/components/Account/index.js",lang="javascript"}
~~~~~~~
import React, { Component } from 'react';
# leanpub-start-insert
import { compose } from 'recompose';
# leanpub-end-insert

import {
  AuthUserContext,
  withAuthorization,
# leanpub-start-insert
  withEmailVerification,
# leanpub-end-insert
} from '../Session';
import { withFirebase } from '../Firebase';
import { PasswordForgetForm } from '../PasswordForget';
import PasswordChangeForm from '../PasswordChange';

...

const condition = authUser => !!authUser;

# leanpub-start-insert
export default compose(
  withEmailVerification,
  withAuthorization(condition),
)(AccountPage);
# leanpub-end-insert
~~~~~~~

All the sensitive routes for authenticated users now require a confirmed email. Finally, your application can be only used by users with real email addresses.

### Exercises:

* Familiarize yourself with the new flow by deleting your user from the Authentication and Realtime Databases and sign up again.
  * For example, sign up with a social login instead of the email/password combination, but activate the email/password sign in method later on the account page.
  * This is in general a good way to purge the database to start from a clean slate if anything feels buggy.
* Implement the "Send confirmation E-Mail" button in a way that it's not shown the first time a user signs up; otherwise the user may be tempted to click the button right away and receives a second confirmation E-Mail.
* Read more about [Firebase's verification E-Mail](https://firebase.google.com/docs/auth/web/manage-users)
* Read more about [additional configuration for the verification E-Mail](https://firebase.google.com/docs/auth/web/passing-state-in-email-actions)
* Confirm your [source code for the last section](http://bit.ly/2Vqtfxt)
