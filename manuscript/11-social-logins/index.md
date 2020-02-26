# Social Logins

So far, you have used a email/password combination to authenticated with the application. Firebase offers more than this sign in method. If you take a closer look at their documentation, you can find social sign in methods for Google, Facebook, Twitter and others. In this section, I want to show you how to use these social logins to give users access to your application. It removes lots of friction to use your application, because not everyone wants to create a new account from scratch. Rather people tend more and more to use social logins for services and products.

Note: The following sections show API keys, secrets, URIs and other sensitive data that you shouldn't share with other people. They should be kept secret. That's why all the sensitive data shown in the following sections is fake.

Firebase has the restriction to allow only one E-Mail address per user. If you try to use another sign in method next to the default email/password sign in method, you may see the following error: *"An account already exists with the same email address but different sign in credentials. Sign in using a provider associated with this email address."* It's because your E-Mail address from your Google account may be the same as for Facebook account or your default email/password combination. In order to overcome this behavior, only for this section though, you can deactivate it in your Firebase dashboard on the Authentication tab. There you can allow more than one account for the same E-Mail address:

![](images/firebase-one-account-per-email_1024.jpg)

Keep in mind that we will revert this configuration later though, because you don't want to create a dedicated user account for each social login in the end. It would mean that someone creating content with their Facebook social login wouldn't own the content with their Google social login anymore, because it's a different account. However, let's create the social logins this way first and see how we can merge them into one account later.

## Troubleshoot

There are a few errors that could show up for while setting up Google, Facebook or Twitter social logins for your application. First, understand the error message yourself and try to figure out the fix for it. However, I want to document a few things I have noticed myself and how I fixed them. If you encounter any of these issues, check again this troubleshooting area. Let's see what kind of errors we have and how to fix them:

**Info:** *The current domain is not authorized for OAuth operations. This will prevent signInWithPopup, signInWithRedirect, linkWithPopup and linkWithRedirect from working. Add your domain (localhost) to the OAuth redirect domains list in the Firebase console -> Auth section -> Sign in method tab.*

On your Firebase dashboard, you will find an Authentication tab to get a list of all your authenticated users, sign up methods and other configuration. Click the Authentication tab and scroll down to "Authorised domains" and add "localhost" there. Then your development domain should be authorized to perform the Auth operations with third-parties.

![](images/firebase-google-social-login-authorized-domains_1024.jpg)

It's a mandatory configuration for most of Firebase's sign in methods. However, it can be that this alone doesn't help and you need to perform further configuration. Therefore, visit [Google's Developer Console](https://console.developers.google.com/apis/credentials) and select your Firebase project in the top-level navigation and navigate to "Credentials" afterward.

There you will see configuration for "API keys" and "OAuth 2.0 client IDs". In "API keys", edit "Browser key (auto created by Google Service)" and add localhost and the `authDomain` from your project's configuration in "Accept requests from these HTTP referrers (websites)".

![](images/firebase-google-developer-console-api-keys_1024.jpg)

Next, in "OAuth 2.0 client IDs", edit "Web client (auto created by Google Service)" and add localhost and the `authDomain` from your project's configuration in "Authorised JavaScript origins".

![](images/firebase-google-developer-console-oauth-client-ids_1024.jpg)

It can take some time until the changes are propagated through Google's services (e.g. Firebase). But then all third-parties should be authorised to access your Firebase project.

## Google Social Login

Before we can start to code the social login for Google with Firebase in React, we need to enable it as sign in method on our Firebase project's dashboard. You can find all your sign in methods under the "Authentication" tab.

![](images/firebase-enable-google-social-login_1024.jpg)

Afterward, we are able to implement the social login in our code. In the Firebase class that's our interface between our React application and the Firebase API, add the Google Authentication Provider and the class method to sign in with Google by using the provider:

{title="src/components/Firebase/firebase.js",lang="javascript"}
~~~~~~~
...

class Firebase {
  constructor() {
    app.initializeApp(config);

    this.auth = app.auth();
    this.db = app.database();

# leanpub-start-insert
    this.googleProvider = new app.auth.GoogleAuthProvider();
# leanpub-end-insert
  }

  // *** Auth API ***

  doCreateUserWithEmailAndPassword = (email, password) =>
    this.auth.createUserWithEmailAndPassword(email, password);

  doSignInWithEmailAndPassword = (email, password) =>
    this.auth.signInWithEmailAndPassword(email, password);

# leanpub-start-insert
  doSignInWithGoogle = () =>
    this.auth.signInWithPopup(this.googleProvider);
# leanpub-end-insert

  doSignOut = () => this.auth.signOut();

  ...
}

export default Firebase;
~~~~~~~

On your sign in page, add a new component for a sign in with Google next to your email/password sign in:

{title="src/components/SignIn/index.js",lang="javascript"}
~~~~~~~
...

const SignInPage = () => (
  <div>
    <h1>SignIn</h1>
    <SignInForm />
# leanpub-start-insert
    <SignInGoogle />
# leanpub-end-insert
    <PasswordForgetLink />
    <SignUpLink />
  </div>
);

...
~~~~~~~

Now implement the complete new form component in this same file for the Google sign in:

{title="src/components/SignIn/index.js",lang="javascript"}
~~~~~~~
...

class SignInGoogleBase extends Component {
  constructor(props) {
    super(props);

    this.state = { error: null };
  }

  onSubmit = event => {
    this.props.firebase
      .doSignInWithGoogle()
      .then(socialAuthUser => {
        this.setState({ error: null });
        this.props.history.push(ROUTES.HOME);
      })
      .catch(error => {
        this.setState({ error });
      });

    event.preventDefault();
  };

  render() {
    const { error } = this.state;

    return (
      <form onSubmit={this.onSubmit}>
        <button type="submit">Sign In with Google</button>

        {error && <p>{error.message}</p>}
      </form>
    );
  }
}

...
~~~~~~~

On submit the form component uses the new Google sign in method given by our Firebase's class instance. In order to pass Firebase and all other required configuration (e.g. history for a redirect after login) to this component, enhance it with all the needed higher-order components:

{title="src/components/SignIn/index.js",lang="javascript"}
~~~~~~~
...

const SignInForm = compose(
  withRouter,
  withFirebase,
)(SignInFormBase);

# leanpub-start-insert
const SignInGoogle = compose(
  withRouter,
  withFirebase,
)(SignInGoogleBase);
# leanpub-end-insert

export default SignInPage;

# leanpub-start-insert
export { SignInForm, SignInGoogle };
# leanpub-end-insert
~~~~~~~

So far, that should do the trick for the sign in with Google sign in method. You will have an authenticated user afterward, but what's missing is the database user that you have to create yourself. It's similar to the sign up (registration) in the SignUpForm component:

{title="src/components/SignIn/index.js",lang="javascript"}
~~~~~~~
...

class SignInGoogleBase extends Component {
  ...

  onSubmit = event => {
    this.props.firebase
      .doSignInWithGoogle()
# leanpub-start-insert
      .then(socialAuthUser => {
        // Create a user in your Firebase Realtime Database too
        return this.props.firebase
          .user(socialAuthUser.user.uid)
          .set({
            username: socialAuthUser.user.displayName,
            email: socialAuthUser.user.email,
            roles: {},
          });
      })
# leanpub-end-insert
      .then(() => {
        this.setState({ error: null });
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

In this scenario, every time a user signs in with Google, a new user with this stable id coming from the social login is created in your database. Basically if a user signs in twice with the same social login, the old user gets overridden. This can be a desired behavior, because maybe a user has changed their username on Google and want to see it reflected in your applications's database too. If you don't want to have this behavior and only create the user once with a social login, make use of the `socialuser.additionalUserInfo.isNewUser` property to only create a new user when signing in with Google for the first time.

### Exercises:

* Read more about the [Google Social Login](https://firebase.google.com/docs/auth/web/google-signin)
* Check your Firebase's Dashboard Authentication/Database tabs to manage your users (e.g. manually remove users).
* Confirm your [source code for the last section](http://bit.ly/2VuH8eh)

## Facebook Social Login

Identical to the previous social login, enable the sign in method on your Firebase dashboard for Facebook. The Facebook social login expects an App ID and and App Secret. You can get these by [creating a new Facebook App with your Facebook Account for this Firebase in React application](https://www.robinwieruch.de/firebase-facebook-login). Afterward, you can find the App ID and App Secret for your new Facebook App.

Afterward, we are able to implement the social login in our code. In the Firebase class, add the Facebook Authentication Provider and the class method to sign in with Facebook by using the provider:

{title="src/components/Firebase/firebase.js",lang="javascript"}
~~~~~~~
...

class Firebase {
  constructor() {
    app.initializeApp(config);

    this.auth = app.auth();
    this.db = app.database();

    this.googleProvider = new app.auth.GoogleAuthProvider();
# leanpub-start-insert
    this.facebookProvider = new app.auth.FacebookAuthProvider();
# leanpub-end-insert
  }

  // *** Auth API ***

  doCreateUserWithEmailAndPassword = (email, password) =>
    this.auth.createUserWithEmailAndPassword(email, password);

  doSignInWithEmailAndPassword = (email, password) =>
    this.auth.signInWithEmailAndPassword(email, password);

  doSignInWithGoogle = () =>
    this.auth.signInWithPopup(this.googleProvider);

# leanpub-start-insert
  doSignInWithFacebook = () =>
    this.auth.signInWithPopup(this.facebookProvider);
# leanpub-end-insert

  doSignOut = () => this.auth.signOut();

  ...
}

export default Firebase;
~~~~~~~

On your sign in page, add a new component for a sign in with Facebook next to your email/password and Google sign ins:

{title="src/components/SignIn/index.js",lang="javascript"}
~~~~~~~
...

const SignInPage = () => (
  <div>
    <h1>SignIn</h1>
    <SignInForm />
    <SignInGoogle />
# leanpub-start-insert
    <SignInFacebook />
# leanpub-end-insert
    <PasswordForgetLink />
    <SignUpLink />
  </div>
);

...
~~~~~~~

Now implement the complete new form component in this same file for the Facebook sign in:

{title="src/components/SignIn/index.js",lang="javascript"}
~~~~~~~
...

class SignInFacebookBase extends Component {
  constructor(props) {
    super(props);

    this.state = { error: null };
  }

  onSubmit = event => {
    this.props.firebase
      .doSignInWithFacebook()
      .then(socialAuthUser => {
        this.setState({ error: null });
        this.props.history.push(ROUTES.HOME);
      })
      .catch(error => {
        this.setState({ error });
      });

    event.preventDefault();
  };

  render() {
    const { error } = this.state;

    return (
      <form onSubmit={this.onSubmit}>
        <button type="submit">Sign In with Facebook</button>

        {error && <p>{error.message}</p>}
      </form>
    );
  }
}

...
~~~~~~~

On submit the form component uses the new Facebook sign in method given by our Firebase's class instance. In order to pass Firebase and all other required configuration to this component, enhance it with all the needed higher-order components:

{title="src/components/SignIn/index.js",lang="javascript"}
~~~~~~~
...

const SignInGoogle = compose(
  withRouter,
  withFirebase,
)(SignInGoogleBase);

# leanpub-start-insert
const SignInFacebook = compose(
  withRouter,
  withFirebase,
)(SignInFacebookBase);
# leanpub-end-insert

export default SignInPage;

# leanpub-start-insert
export { SignInForm, SignInGoogle, SignInFacebook };
# leanpub-end-insert
~~~~~~~

You will have an authenticated user afterward, but what's missing again is the database user that you have to create yourself:

{title="src/components/SignIn/index.js",lang="javascript"}
~~~~~~~
...

class SignInFacebookBase extends Component {
  ...

  onSubmit = event => {
    this.props.firebase
      .doSignInWithFacebook()
# leanpub-start-insert
      .then(socialAuthUser => {
        // Create a user in your Firebase Realtime Database too
        return this.props.firebase
          .user(socialAuthUser.user.uid)
          .set({
            username: socialAuthUser.additionalUserInfo.profile.name,
            email: socialAuthUser.additionalUserInfo.profile.email,
            roles: {},
          });
      })
# leanpub-end-insert
      .then(() => {
        this.setState({ error: null });
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

Again, every time a user signs in with Facebook, a new user with this stable id coming from the social login is created in your database. Basically if a user signs in twice with the same social login, the old user gets overridden. You can optionally make use of the `socialuser.additionalUserInfo.isNewUser` property to only create a new user when signing in with Facebook for the first time.

### Exercises:

* Read more about the [Facebook Social Login](https://firebase.google.com/docs/auth/web/facebook-login)
* Figure out whether there is a way to interact with Facebook's API afterward, because the `socialUser` has an `accessToken` in its `credentials` object.
* Like my [Facebook](https://www.facebook.com/rwieruch/) page to receive latest tutorials for web developers.
* Confirm your [source code for the last section](http://bit.ly/2VuH8eh)

## Twitter Social Login

Identical to the previous social logins, enable the sign in method on your Firebase dashboard for Twitter. The Twitter social login expects an API key and API secret. You can get these by [creating a new Twitter App with your Twitter Account for this Firebase in React application](https://www.robinwieruch.de/firebase-twitter-login). Afterward, you can find the API key and API secret for your new Twitter App.

Now, we are able to implement the social login in our code. In the Firebase class, add the Twitter Authentication Provider and the class method to sign in with Twitter by using the provider:

{title="src/components/Firebase/firebase.js",lang="javascript"}
~~~~~~~
...

class Firebase {
  constructor() {
    app.initializeApp(config);

    this.auth = app.auth();
    this.db = app.database();

    this.googleProvider = new app.auth.GoogleAuthProvider();
    this.facebookProvider = new app.auth.FacebookAuthProvider();
# leanpub-start-insert
    this.twitterProvider = new app.auth.TwitterAuthProvider();
# leanpub-end-insert
  }

  // *** Auth API ***

  ...

  doSignInWithGoogle = () =>
    this.auth.signInWithPopup(this.googleProvider);

  doSignInWithFacebook = () =>
    this.auth.signInWithPopup(this.facebookProvider);

# leanpub-start-insert
  doSignInWithTwitter = () =>
    this.auth.signInWithPopup(this.twitterProvider);
# leanpub-end-insert

  doSignOut = () => this.auth.signOut();

  ...
}

export default Firebase;
~~~~~~~

On your sign in page, add a new component for a sign in with Twitter next to your email/password, Google and Facebook sign ins:

{title="src/components/SignIn/index.js",lang="javascript"}
~~~~~~~
...

const SignInPage = () => (
  <div>
    <h1>SignIn</h1>
    <SignInForm />
    <SignInGoogle />
    <SignInFacebook />
# leanpub-start-insert
    <SignInTwitter />
# leanpub-end-insert
    <PasswordForgetLink />
    <SignUpLink />
  </div>
);

...
~~~~~~~

Now implement the complete new form component in this same file for the Twitter sign in:

{title="src/components/SignIn/index.js",lang="javascript"}
~~~~~~~
...

class SignInTwitterBase extends Component {
  constructor(props) {
    super(props);

    this.state = { error: null };
  }

  onSubmit = event => {
    this.props.firebase
      .doSignInWithTwitter()
      .then(socialAuthUser => {
        this.setState({ error: null });
        this.props.history.push(ROUTES.HOME);
      })
      .catch(error => {
        this.setState({ error });
      });

    event.preventDefault();
  };

  render() {
    const { error } = this.state;

    return (
      <form onSubmit={this.onSubmit}>
        <button type="submit">Sign In with Twitter</button>

        {error && <p>{error.message}</p>}
      </form>
    );
  }
}

...
~~~~~~~

On submit the form component uses the new Twitter sign in method given by our Firebase's class instance. In order to pass Firebase and all other required configuration to this component, enhance it with all the needed higher-order components:

{title="src/components/SignIn/index.js",lang="javascript"}
~~~~~~~
...

const SignInFacebook = compose(
  withRouter,
  withFirebase,
)(SignInFacebookBase);

# leanpub-start-insert
const SignInTwitter = compose(
  withRouter,
  withFirebase,
)(SignInTwitterBase);
# leanpub-end-insert

export default SignInPage;

# leanpub-start-insert
export { SignInForm, SignInGoogle, SignInFacebook, SignInTwitter };
# leanpub-end-insert
~~~~~~~

You will have an authenticated user afterward, but what's missing again is the database user that you have to create yourself:

{title="src/components/SignIn/index.js",lang="javascript"}
~~~~~~~
...

class SignInTwitterBase extends Component {
  ...

  onSubmit = event => {
    this.props.firebase
      .doSignInWithTwitter()
# leanpub-start-insert
      .then(socialAuthUser => {
        // Create a user in your Firebase Realtime Database too
        return this.props.firebase
          .user(socialAuthUser.user.uid)
          .set({
            username: socialAuthUser.additionalUserInfo.profile.name,
            email: socialAuthUser.additionalUserInfo.profile.email,
            roles: {},
          });
      })
# leanpub-end-insert
      .then(() => {
        this.setState({ error: null });
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

Again, every time a user signs in with Twitter, a new user with this stable id coming from the social login is created in your database. Basically if a user signs in twice with the same social login, the old user gets overridden. You can optionally make use of the `socialuser.additionalUserInfo.isNewUser` property to only create a new user when signing in with Twitter for the first time .

### Exercises:

* Read more about the [Twitter Social Login](https://firebase.google.com/docs/auth/web/twitter-login)
* Figure out whether there is a way to interact with Twitter's API afterward, because the `socialUser` has an `accessToken` and `secret` in its `credentials` object.
* Follow my [Twitter](https://twitter.com/rwieruch) page to receive latest tutorials for web developers.
* Confirm your [source code for the last section](http://bit.ly/2VuH8eh)

## Linking Social Logins to one Account

The last section walked you through implementing social logins for Google, Facebook, and Twitter to being able to sign up/in with a email/password combination. However, since you have enabled multiple accounts for one email address, there is more than one account associated to your email, which can lead to bugs for your service/product. Imagine a user signs in with Google, buys a ebook on your website, is able to download the book as signed in user, and then signs out again. The next sign-in with the email/password combination won't show the e-book anymore. This is because the user has two accounts on your website. While one account is associated with Google, the other one is associated with the email/password combination.

To walk through this scenario, take one of your social accounts (Google, Facebook, Twitter) and log into the Firebase in React application. Check the account page and copy the email address that is associated to your social account. Log out and log in again with your email/password combination, using the same email as for your social login. It's possible because we enabled multiple accounts for the same email address in the Firebase dashboard. When you check the account page again, you should see the same email as when you logged in with the social account. Now head to your Firebase dashboard and check the "Authentication" tab. You should find two accounts associated to the same email you used before. The same applies for the "Database" tab.

In this section, we want to prevent this behavior by using only one email address per user, while still being able to sign-in via email/password, Google, Facebook or Twitter. It shouldn't matter which sign-in you take, as the account should be the same. That's where the linking of all the social accounts comes in.

Before we get started, head to the Authentication and Database tabs on your Firebase dashboard and delete the user you used with your personal email address. We will use this email address later, except this time it will end up once in both tabs for one account. First, disable the setting on your Firebase dashboard that encourages email addresses associated to more than one account.

![](images/firebase-only-one-email_1024.jpg)

We will prevent the user from signing in with another account when there is already an account associated to this email address. A message should point the user to the account page to link all the social accounts and the email/password account with each other instead. Let's show the user a custom error message for the sign up page. First, extract the error code and the custom message as variables:

{title="src/components/SignIn/index.js",lang="javascript"}
~~~~~~~
# leanpub-start-insert
const ERROR_CODE_ACCOUNT_EXISTS =
  'auth/account-exists-with-different-credential';
# leanpub-end-insert

# leanpub-start-insert
const ERROR_MSG_ACCOUNT_EXISTS = `
  An account with an E-Mail address to
  this social account already exists. Try to login from
  this account instead and associate your social accounts on
  your personal account page.
`;
# leanpub-end-insert
~~~~~~~

Next, show the custom error message when the error code shows up. That's because we prevent more than one email address for one account:

{title="src/components/SignIn/index.js",lang="javascript"}
~~~~~~~
...

class SignInGoogleBase extends Component {
  ...

  onSubmit = event => {
    this.props.firebase
      .doSignInWithGoogle()
      .then(socialAuthUser => {
        ...
      })
      .then(() => {
        ...
      })
      .catch(error => {
# leanpub-start-insert
        if (error.code === ERROR_CODE_ACCOUNT_EXISTS) {
          error.message = ERROR_MSG_ACCOUNT_EXISTS;
        }
# leanpub-end-insert

        this.setState({ error });
      });

    event.preventDefault();
  };

  ...
}

...
~~~~~~~

Repeat this for the other social logins (Facebook, Twitter) as well. If a user signs in with one of the social logins, but there is already an account in the system with this email address, the custom error message shows up. The user has to log in with the correct sign-in method and link all other desired social accounts to this account on the account page. We will add this feature later in the account page, but before this, we need to show a similar custom error message for the sign up page as well. The user might use a social login first and later attempt to sign up with an email address (email/password sign up) that has been used by the social login already.

{title="src/components/SignUp/index.js",lang="javascript"}
~~~~~~~
# leanpub-start-insert
const ERROR_CODE_ACCOUNT_EXISTS = 'auth/email-already-in-use';
# leanpub-end-insert

# leanpub-start-insert
const ERROR_MSG_ACCOUNT_EXISTS = `
  An account with this E-Mail address already exists.
  Try to login with this account instead. If you think the
  account is already used from one of the social logins, try
  to sign-in with one of them. Afterward, associate your accounts
  on your personal account page.
`;
# leanpub-end-insert
~~~~~~~

Use the custom error message when the error code happens on sign-up:

{title="src/components/SignUp/index.js",lang="javascript"}
~~~~~~~
...

class SignUpFormBase extends Component {
  ...

  onSubmit = event => {
    const { username, email, passwordOne, isAdmin } = this.state;
    const roles = {};

    if (isAdmin) {
      roles[ROLES.ADMIN] = ROLES.ADMIN;
    }

    this.props.firebase
      .doCreateUserWithEmailAndPassword(email, passwordOne)
      .then(authUser => {
        ...
      })
      .then(() => {
        ...
      })
      .catch(error => {
# leanpub-start-insert
        if (error.code === ERROR_CODE_ACCOUNT_EXISTS) {
          error.message = ERROR_MSG_ACCOUNT_EXISTS;
        }
# leanpub-end-insert

        this.setState({ error });
      });

    event.preventDefault();
  };

  ...
}

...
~~~~~~~

Now users can use the same email address for different sign-in methods. Next, let's head to the account page, where we'll create an area to manage and activate/deactivate all the sign-in methods (social sign-ins, email/password sign-in). Introduce all available sign-in methods and their optional providers (see Firebase class) as list of objects:

{title="src/components/Account/index.js",lang="javascript"}
~~~~~~~
...

# leanpub-start-insert
const SIGN_IN_METHODS = [
  {
    id: 'password',
    provider: null,
  },
  {
    id: 'google.com',
    provider: 'googleProvider',
  },
  {
    id: 'facebook.com',
    provider: 'facebookProvider',
  },
  {
    id: 'twitter.com',
    provider: 'twitterProvider',
  },
];
# leanpub-end-insert

const AccountPage = () => (
  <AuthUserContext.Consumer>
    {authUser => (
      <div>
        <h1>Account: {authUser.email}</h1>
        <PasswordForgetForm />
        <PasswordChangeForm />
# leanpub-start-insert
        <LoginManagement authUser={authUser} />
# leanpub-end-insert
      </div>
    )}
  </AuthUserContext.Consumer>
);

...
~~~~~~~

Now implement the new component and render all available sign-in methods as buttons which are doing nothing:

{title="src/components/Account/index.js",lang="javascript"}
~~~~~~~
# leanpub-start-insert
import React, { Component } from 'react';
# leanpub-end-insert

...

# leanpub-start-insert
class LoginManagement extends Component {
  render() {
    return (
      <div>
        Sign In Methods:
        <ul>
          {SIGN_IN_METHODS.map(signInMethod => {
            return (
              <li key={signInMethod.id}>
                <button type="button" onClick={() => {}}>
                  {signInMethod.id}
                </button>
              </li>
            );
          })}
        </ul>
      </div>
    );
  }
}
# leanpub-end-insert

...
~~~~~~~

Remember to make the Firebase instance available to the component, because we need to use it in the next step:

{title="src/components/Account/index.js",lang="javascript"}
~~~~~~~
import React, { Component } from 'react';

import { AuthUserContext, withAuthorization } from '../Session';
# leanpub-start-insert
import { withFirebase } from '../Firebase';
# leanpub-end-insert
import { PasswordForgetForm } from '../PasswordForget';
import PasswordChangeForm from '../PasswordChange';

...

# leanpub-start-insert
class LoginManagementBase extends Component {
# leanpub-end-insert
  constructor(props) {
    ...
  }

  componentDidMount() {
    ...
  }

  render() {
    ...
  }
}

# leanpub-start-insert
const LoginManagement = withFirebase(LoginManagementBase);
# leanpub-end-insert

...
~~~~~~~

Then, fetch all active sign-in methods for the user's email address. Firebase has an API for it:

{title="src/components/Account/index.js",lang="javascript"}
~~~~~~~
...

class LoginManagementBase extends Component {
# leanpub-start-insert
  constructor(props) {
    super(props);

    this.state = {
      activeSignInMethods: [],
      error: null,
    };
  }
# leanpub-end-insert

# leanpub-start-insert
  componentDidMount() {
    this.props.firebase.auth
      .fetchSignInMethodsForEmail(this.props.authUser.email)
      .then(activeSignInMethods =>
        this.setState({ activeSignInMethods, error: null }),
      )
      .catch(error => this.setState({ error }));
  }
# leanpub-end-insert

  ...
}

...
~~~~~~~

Next, differentiate between active sign-in methods and the remaining sign-in methods not in the list of fetched sign-in methods. You can show an error message with a conditional rendering as well:

{title="src/components/Account/index.js",lang="javascript"}
~~~~~~~
class LoginManagementBase extends Component {
  ...

  render() {
# leanpub-start-insert
    const { activeSignInMethods, error } = this.state;
# leanpub-end-insert

    return (
      <div>
        Sign In Methods:
        <ul>
          {SIGN_IN_METHODS.map(signInMethod => {
# leanpub-start-insert
            const isEnabled = activeSignInMethods.includes(
              signInMethod.id,
            );
# leanpub-end-insert

            return (
              <li key={signInMethod.id}>
# leanpub-start-insert
                {isEnabled ? (
                  <button type="button" onClick={() => {}}>
                    Deactivate {signInMethod.id}
                  </button>
                ) : (
                  <button type="button" onClick={() => {}}>
                    Link {signInMethod.id}
                  </button>
                )}
# leanpub-end-insert
              </li>
            );
          })}
        </ul>
# leanpub-start-insert
        {error && error.message}
# leanpub-end-insert
      </div>
    );
  }
}
~~~~~~~

While all available sign-in methods are displayed, they differentiate between active and non-active. The active methods can be deactivated. On the other hand, sign-in methods that are available but not used by the user can be linked instead to make them active. We will implement both details in the next step:

{title="src/components/Account/index.js",lang="javascript"}
~~~~~~~
class LoginManagementBase extends Component {
  ...

# leanpub-start-insert
  componentDidMount() {
    this.fetchSignInMethods();
  }
# leanpub-end-insert

# leanpub-start-insert
  fetchSignInMethods = () => {
# leanpub-end-insert
    this.props.firebase.auth
      .fetchSignInMethodsForEmail(this.props.authUser.email)
      .then(activeSignInMethods =>
        this.setState({ activeSignInMethods, error: null }),
      )
      .catch(error => this.setState({ error }));
# leanpub-start-insert
  };
# leanpub-end-insert

# leanpub-start-insert
  onSocialLoginLink = provider => {
    ...
  };
# leanpub-end-insert

# leanpub-start-insert
  onUnlink = providerId => {
    ...
  };
# leanpub-end-insert

  ...
}
~~~~~~~

Extract the fetch method, because we will use it after we linked (activated) or unlinked (deactivated) sign-in methods. Then the new class methods can be used by the buttons:

{title="src/components/Account/index.js",lang="javascript"}
~~~~~~~
class LoginManagementBase extends Component {
  ...

  render() {
    const { activeSignInMethods, error } = this.state;

    return (
      <div>
        Sign In Methods:
        <ul>
          {SIGN_IN_METHODS.map(signInMethod => {
# leanpub-start-insert
            const onlyOneLeft = activeSignInMethods.length === 1;
# leanpub-end-insert
            const isEnabled = activeSignInMethods.includes(
              signInMethod.id,
            );

            return (
              <li key={signInMethod.id}>
                {isEnabled ? (
                  <button
                    type="button"
# leanpub-start-insert
                    onClick={() => this.onUnlink(signInMethod.id)}
                    disabled={onlyOneLeft}
# leanpub-end-insert
                  >
                    Deactivate {signInMethod.id}
                  </button>
                ) : (
                  <button
                    type="button"
# leanpub-start-insert
                    onClick={() =>
                      this.onSocialLoginLink(signInMethod.provider)
                    }
# leanpub-end-insert
                  >
                    Link {signInMethod.id}
                  </button>
                )}
              </li>
            );
          })}
        </ul>
        {error && error.message}
      </div>
    );
  }
}
~~~~~~~

Also, we added an improvement to avoid getting locked out of the application. If only one sign-in method is left as active, disable all deactivation buttons because there needs to be at least one sign-in method. Now let's implement the class methods for linking and unlinking accounts:

{title="src/components/Account/index.js",lang="javascript"}
~~~~~~~
class LoginManagementBase extends Component {
  ...

  onSocialLoginLink = provider => {
# leanpub-start-insert
    this.props.firebase.auth.currentUser
      .linkWithPopup(this.props.firebase[provider])
      .then(this.fetchSignInMethods)
      .catch(error => this.setState({ error }));
# leanpub-end-insert
  };

  onUnlink = providerId => {
# leanpub-start-insert
    this.props.firebase.auth.currentUser
      .unlink(providerId)
      .then(this.fetchSignInMethods)
      .catch(error => this.setState({ error }));
# leanpub-end-insert
  };

  ...
}
~~~~~~~

Finally we are able to link and unlink accounts. Afterward, all active sign-in methods are fetched again. That's why we have extracted this class method from the `componentDidMount()` lifecycle method before, which is reusable now. The linking of the sign-in methods should work for Google, Facebook and Twitter now. However, it doesn't work for the email/password combination yet, because this one isn't done by a simple button click. If the user has only active social sign-in methods but no email/password sign-in method, an email/password combination must be provided; then it is possible to link this sign-in method to the other social sign-in methods.

First, extract the social sign-in methods to its own component and add a conditional rendering for the password sign-in method:

{title="src/components/Account/index.js",lang="javascript"}
~~~~~~~
class LoginManagementBase extends Component {
  ...

# leanpub-start-insert
  onDefaultLoginLink = () => {
    ...
  };
# leanpub-end-insert

  render() {
    const { activeSignInMethods, error } = this.state;

    return (
      <div>
        Sign In Methods:
        <ul>
          {SIGN_IN_METHODS.map(signInMethod => {
            ...

            return (
              <li key={signInMethod.id}>
# leanpub-start-insert
                {signInMethod.id === 'password' ? (
                  <DefaultLoginToggle
                    onlyOneLeft={onlyOneLeft}
                    isEnabled={isEnabled}
                    signInMethod={signInMethod}
                    onLink={this.onDefaultLoginLink}
                    onUnlink={this.onUnlink}
                  />
                ) : (
                  <SocialLoginToggle
                    onlyOneLeft={onlyOneLeft}
                    isEnabled={isEnabled}
                    signInMethod={signInMethod}
                    onLink={this.onSocialLoginLink}
                    onUnlink={this.onUnlink}
                  />
                )}
# leanpub-end-insert
              </li>
            );
          })}
        </ul>
        {error && error.message}
      </div>
    );
  }
}
~~~~~~~

The DefaultLoginToggle component will use a different `onLink` handler than the SocialLoginToggle component, but the `onUnlink` stays the same. We will implement DefaultLoginToggle component and its missing handler in a moment, but first let's extract the SocialLoginToggle component:

{title="src/components/Account/index.js",lang="javascript"}
~~~~~~~
# leanpub-start-insert
const SocialLoginToggle = ({
  onlyOneLeft,
  isEnabled,
  signInMethod,
  onLink,
  onUnlink,
}) =>
# leanpub-end-insert
  isEnabled ? (
    <button
      type="button"
# leanpub-start-insert
      onClick={() => onUnlink(signInMethod.id)}
# leanpub-end-insert
      disabled={onlyOneLeft}
    >
      Deactivate {signInMethod.id}
    </button>
  ) : (
    <button
      type="button"
# leanpub-start-insert
      onClick={() => onLink(signInMethod.provider)}
# leanpub-end-insert
    >
      Link {signInMethod.id}
    </button>
  );
~~~~~~~

The implementation details didn't change, but the component is standalone now. Next, let's implement the other component for the email/password sign-in. When this sign-in method is activated, it's sufficient to render only a button similar to the social sign-in methods to unlink (deactivate) this sign-in method. If this sign-in method isn't activated, you need to retrieve the user's desired email and password combination to link it as account to the other social accounts. It's very similar to our sign up form then:

{title="src/components/Account/index.js",lang="javascript"}
~~~~~~~
class DefaultLoginToggle extends Component {
  constructor(props) {
    super(props);

    this.state = { passwordOne: '', passwordTwo: '' };
  }

  onSubmit = event => {
    event.preventDefault();

    this.props.onLink(this.state.passwordOne);
    this.setState({ passwordOne: '', passwordTwo: '' });
  };

  onChange = event => {
    this.setState({ [event.target.name]: event.target.value });
  };

  render() {
    const {
      onlyOneLeft,
      isEnabled,
      signInMethod,
      onUnlink,
    } = this.props;

    const { passwordOne, passwordTwo } = this.state;

    const isInvalid =
      passwordOne !== passwordTwo || passwordOne === '';

    return isEnabled ? (
      <button
        type="button"
        onClick={() => onUnlink(signInMethod.id)}
        disabled={onlyOneLeft}
      >
        Deactivate {signInMethod.id}
      </button>
    ) : (
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
          Link {signInMethod.id}
        </button>
      </form>
    );
  }
}
~~~~~~~

Next, let's implement the handler in the parent component for the default sign-in via email/password. It receives a password from the child component, which is added to the authenticated user's email address:

{title="src/components/Account/index.js",lang="javascript"}
~~~~~~~
class LoginManagementBase extends Component {
  ...

# leanpub-start-insert
  onDefaultLoginLink = password => {
    const credential = this.props.firebase.emailAuthProvider.credential(
      this.props.authUser.email,
      password,
    );

    this.props.firebase.auth.currentUser
      .linkAndRetrieveDataWithCredential(credential)
      .then(this.fetchSignInMethods)
      .catch(error => this.setState({ error }));
  };
# leanpub-end-insert

  ...
}
~~~~~~~

The Firebase API is not too elegant here, but it's good to know that it creates a credential from the user's email and desired password. Afterward, it links it to the other accounts. Then all active sign-in methods are fetched again to keep everything updated.

Previously, when we set up our Firebase class, we overrode its `auth` property with `app.auth()`. However, to create the credential from the email and password in the component, we need access to the Firebase internal `auth`, which has the `EmailAuthProvider` property, so we reference it before we override it with `app.auth()` in the next lines.

{title="src/components/Firebase/firebase.js",lang="javascript"}
~~~~~~~
...

class Firebase {
  constructor() {
    app.initializeApp(config);

# leanpub-start-insert
    this.emailAuthProvider = app.auth.EmailAuthProvider;
# leanpub-end-insert
    this.auth = app.auth();
    this.db = app.database();

    this.googleProvider = new app.auth.GoogleAuthProvider();
    this.facebookProvider = new app.auth.FacebookAuthProvider();
    this.twitterProvider = new app.auth.TwitterAuthProvider();
  }

  ...
}

...
~~~~~~~

Now you can link and unlink different sign-in methods using only one account and email address.

### Exercises:

* Try to link and unlink different sign-in methods and check if you are able to sign-in with this method afterwards.
* Implement loading indicators for each button that activate and deactivate the sign-in methods for a better user experience.
* Read more about [social account linking in Firebase](https://firebase.google.com/docs/auth/web/account-linking)
* Confirm your [source code for the last section](http://bit.ly/2VnF3Rf)
