# Firebase's Authentication API

In the previous section, you created a Firebase project on the official Firebase website. This section will implement the interface of your Firebase class that enables communication between the class and the Firebase authentication API. In the sections afterward, you will use the interface of the Firebase class in your React components.

First, we need to activate one of the available authentication providers on Firebase's website. On your project's Firebase dashboard, you can find a menu item which says "Authentication". Select it and click "Sign-In Method" menu item afterward. There you can enable the authentication with Email/Password:

![](images/firebase-authentication-methods_1024.jpg)

Second, we will implement the authentication API for our Firebase class. Import and instantiate the package from Firebase responsible for all the authentication in your *src/components/Firebase/firebase.js* file:

{title="src/components/Firebase/firebase.js",lang="javascript"}
~~~~~~~
import app from 'firebase/app';
# leanpub-start-insert
import 'firebase/auth';
# leanpub-end-insert

const config = {
  apiKey: process.env.REACT_APP_API_KEY,
  authDomain: process.env.REACT_APP_AUTH_DOMAIN,
  databaseURL: process.env.REACT_APP_DATABASE_URL,
  projectId: process.env.REACT_APP_PROJECT_ID,
  storageBucket: process.env.REACT_APP_STORAGE_BUCKET,
  messagingSenderId: process.env.REACT_APP_MESSAGING_SENDER_ID,
};

class Firebase {
  constructor() {
    app.initializeApp(config);

# leanpub-start-insert
    this.auth = app.auth();
# leanpub-end-insert
  }
}

export default Firebase;
~~~~~~~

Let's define all the authentication functions as class methods step by step. They will serve our communication channel from the Firebase class to the Firebase API. First, the sign up function (registration) takes email and password parameters for its function signature and uses an official Firebase API endpoint to create a user:

{title="src/components/Firebase/firebase.js",lang="javascript"}
~~~~~~~
import app from 'firebase/app';
import 'firebase/auth';

const config = { ... };

class Firebase {
  constructor() {
    app.initializeApp(config);

    this.auth = app.auth();
  }

# leanpub-start-insert
  // *** Auth API ***
# leanpub-end-insert

# leanpub-start-insert
  doCreateUserWithEmailAndPassword = (email, password) =>
    this.auth.createUserWithEmailAndPassword(email, password);
# leanpub-end-insert
}

export default Firebase;
~~~~~~~

We'll also set up the login/sign-in function, which takes email and password parameters, as well:

{title="src/components/Firebase/firebase.js",lang="javascript"}
~~~~~~~
import app from 'firebase/app';
import 'firebase/auth';

const config = { ... };

class Firebase {
  constructor() {
    app.initializeApp(config);

    this.auth = app.auth();
  }

  // *** Auth API ***

  doCreateUserWithEmailAndPassword = (email, password) =>
    this.auth.createUserWithEmailAndPassword(email, password);

# leanpub-start-insert
  doSignInWithEmailAndPassword = (email, password) =>
    this.auth.signInWithEmailAndPassword(email, password);
# leanpub-end-insert
}

export default Firebase;
~~~~~~~

These endpoints are called asynchronously, and they will need to be resolved later, as well as error handling. For instance, it is not possible to sign in a user who is not signed up yet since the Firebase API would return an error. In case of the sign out function, you don't need to pass any argument to it, because Firebase knows about the currently authenticated user. If no user is authenticated, nothing will happen when this function is called.

{title="src/components/Firebase/firebase.js",lang="javascript"}
~~~~~~~
import app from 'firebase/app';
import 'firebase/auth';

const config = { ... };

class Firebase {
  constructor() {
    app.initializeApp(config);

    this.auth = app.auth();
  }

  // *** Auth API ***

  doCreateUserWithEmailAndPassword = (email, password) =>
    this.auth.createUserWithEmailAndPassword(email, password);

  doSignInWithEmailAndPassword = (email, password) =>
    this.auth.signInWithEmailAndPassword(email, password);

# leanpub-start-insert
  doSignOut = () => this.auth.signOut();
# leanpub-end-insert
}

export default Firebase;
~~~~~~~

There are two more authentication methods to reset and change a password for an authenticated user:

{title="src/components/Firebase/firebase.js",lang="javascript"}
~~~~~~~
import app from 'firebase/app';
import 'firebase/auth';

const config = { ... };

class Firebase {
  constructor() {
    app.initializeApp(config);

    this.auth = app.auth();
  }

  // *** Auth API ***

  doCreateUserWithEmailAndPassword = (email, password) =>
    this.auth.createUserWithEmailAndPassword(email, password);

  doSignInWithEmailAndPassword = (email, password) =>
    this.auth.signInWithEmailAndPassword(email, password);

  doSignOut = () => this.auth.signOut();

# leanpub-start-insert
  doPasswordReset = email => this.auth.sendPasswordResetEmail(email);
# leanpub-end-insert

# leanpub-start-insert
  doPasswordUpdate = password =>
    this.auth.currentUser.updatePassword(password);
# leanpub-end-insert
}

export default Firebase;
~~~~~~~

That's the authentication interface for your React components that will connect to the Firebase API. In the next section, we will consume all the methods of your Firebase class in your React components.

### Exercises:

* Read more about [Firebase Authentication for Web](https://firebase.google.com/docs/auth/web/start)
* Confirm your [source code for the last section](http://bit.ly/2VpQ1pk)

## Sign Up with React and Firebase

We set up all the routes for your application, configured Firebase and implemented the authentication API for your Firebase class. It's also possible to use Firebase within your React components. Now it's time to use the authentication functionalities in your React components, which we'll build from scratch. I try to put most of the code in one block, because the components are not too small, and splitting them up step by step might be too verbose. Nevertheless, I will guide you through each code block afterward. The code blocks for forms can become repetitive, so they will be explained once well.

Let's start with the sign up page (registration page). It consists of the page, a form, and a link. The form is used to sign up a new user to your application with username, email, and password. The link will be used on the sign in page (login page) later if a user has no account yet. It is a redirect to the sign up page, but not used on the sign up page itself. Implement the *src/components/SignUp/index.js* file the following way:

{title="src/components/SignUp/index.js",lang="javascript"}
~~~~~~~
import React, { Component } from 'react';
import { Link } from 'react-router-dom';

import * as ROUTES from '../../constants/routes';

const SignUpPage = () => (
  <div>
    <h1>SignUp</h1>
    <SignUpForm />
  </div>
);

class SignUpForm extends Component {
  constructor(props) {
    super(props);
  }

  onSubmit = event => {

  }

  onChange = event => {

  };

  render() {
    return (
      <form onSubmit={this.onSubmit}>

      </form>
    );
  }
}

const SignUpLink = () => (
  <p>
    Don't have an account? <Link to={ROUTES.SIGN_UP}>Sign Up</Link>
  </p>
);

export default SignUpPage;

export { SignUpForm, SignUpLink };
~~~~~~~

The SignUpForm component is the only React class component in this file, because it has to manage the form state in React's local state. There are two pieces missing in the current SignUpForm component: the form content in the render method in terms of input fields to capture the information (email address, password, etc.) of a user and the implementation of the `onSubmit` class method when a user signs up eventually.

First, let's initialize the state of the component. It will capture the user information such as username, email, and password. There will be a second password field/state for a password confirmation. In addition, there is an error state to capture an error object in case of the sign up request to the Firebase API fails. The state is initialized by an object destructuring. This way, we can use the initial state object to reset the state after a successful sign up.

{title="src/components/SignUp/index.js",lang="javascript"}
~~~~~~~
...

# leanpub-start-insert
const INITIAL_STATE = {
  username: '',
  email: '',
  passwordOne: '',
  passwordTwo: '',
  error: null,
};
# leanpub-end-insert

class SignUpForm extends Component {
  constructor(props) {
    super(props);

# leanpub-start-insert
    this.state = { ...INITIAL_STATE };
# leanpub-end-insert
  }

  ...

}

...
~~~~~~~

Let's implement all the input fields to capture the information in the render method of the component. The input fields need to update the local state of the component by using a `onChange` handler.

{title="src/components/SignUp/index.js",lang="javascript"}
~~~~~~~
...

class SignUpForm extends Component {

  ...

# leanpub-start-insert
  onChange = event => {
    this.setState({ [event.target.name]: event.target.value });
  };
# leanpub-end-insert

  render() {
# leanpub-start-insert
    const {
      username,
      email,
      passwordOne,
      passwordTwo,
      error,
    } = this.state;
# leanpub-end-insert

    return (
      <form onSubmit={this.onSubmit}>
# leanpub-start-insert
        <input
          name="username"
          value={username}
          onChange={this.onChange}
          type="text"
          placeholder="Full Name"
        />
        <input
          name="email"
          value={email}
          onChange={this.onChange}
          type="text"
          placeholder="Email Address"
        />
        <input
          name="passwordOne"
          value={passwordOne}
          onChange={this.onChange}
          type="password"
          placeholder="Password"
        />
        <input
          name="passwordTwo"
          value={passwordTwo}
          onChange={this.onChange}
          type="password"
          placeholder="Confirm Password"
        />
        <button type="submit">Sign Up</button>

        {error && <p>{error.message}</p>}
# leanpub-end-insert
      </form>
    );
  }
}

...
~~~~~~~

Let's take the last implemented code block apart. All the input fields implement the unidirectional data flow of React; thus, each input field gets a value from the local state and updates the value in the local state with a `onChange` handler. The input fields are controlled by the local state of the component and don't control their own states. [They are controlled components](https://www.robinwieruch.de/react-controlled-components/).

In the last part of the form, there is an optional error message from an error object. The error objects from Firebase have this message property by default, so you can rely on it to display the proper text for your application's user. However, the message is only shown when there is an actual error using a [conditional rendering](https://www.robinwieruch.de/conditional-rendering-react/).

One piece in the form is missing: validation. Let's use an `isInvalid` boolean to enable or disable the submit button.

{title="src/components/SignUp/index.js",lang="javascript"}
~~~~~~~
...

class SignUpForm extends Component {

  ...

  render() {
    const {
      username,
      email,
      passwordOne,
      passwordTwo,
      error,
    } = this.state;

# leanpub-start-insert
    const isInvalid =
      passwordOne !== passwordTwo ||
      passwordOne === '' ||
      email === '' ||
      username === '';
# leanpub-end-insert

    return (
      <form onSubmit={this.onSubmit}>
        <input
        ...
# leanpub-start-insert
        <button disabled={isInvalid} type="submit">
# leanpub-end-insert
          Sign Up
        </button>

        {error && <p>{error.message}</p>}
      </form>
    );
  }
}

...
~~~~~~~

The user is only allowed to sign up if both passwords are the same, and if the username, email and at least one password are filled with a string. This is password confirmation in a common sign up process.

You should be able to visit the */signup* route in your browser after starting your application to confirm that the form with all its input fields shows up. You should also  be able to type into it (confirmation that the local state updates are working) and able to enable the submit button by providing all input fields a string (confirmation that the validation works).

What's missing in the component is the `onSubmit()` class method, which will pass all the form data to the Firebase authentication API via your authentication interface in the Firebase class:

{title="src/components/SignUp/index.js",lang="javascript"}
~~~~~~~
...

class SignUpForm extends Component {

  ...

  onSubmit = event => {
# leanpub-start-insert
    const { username, email, passwordOne } = this.state;

    this.props.firebase
      .doCreateUserWithEmailAndPassword(email, passwordOne)
      .then(authUser => {
        this.setState({ ...INITIAL_STATE });
      })
      .catch(error => {
        this.setState({ error });
      });

    event.preventDefault();
# leanpub-end-insert
  };

  ...
}

...
~~~~~~~

The code is not working yet, but let's break down what we have so far. All the necessary information passed to the authentication API can be destructured from the local state. You will only need one password property, because both password strings should be the same after the validation.

Next, call the sign up function defined in the previous section in the Firebase class, which takes the email and the password property. The username is not used yet for the sign up process, but will be used later.

If the request resolves successfully, you can set the local state of the component to its initial state to empty the input fields. If the request is rejected, you will run into the catch block and set the error object in the local state. An error message should show up in the form due to the conditional rendering in your component's render method.

Also, the `preventDefault()` method on the event prevents a reload of the browser which otherwise would be a natural behavior when using a submit in a form. Note that the signed up user object from the Firebase API is available in the callback function of the then block in our request. You will use it later with the username.

You may have also noticed that one essential piece is missing: We didn't make the Firebase instance available in the SignUpForm [component's props](https://www.robinwieruch.de/react-pass-props-to-component/) yet. Let's change this by utilizing our Firebase Context in the SignUpPage component, and by passing the Firebase instance to the SignUpForm.

{title="src/components/SignUp/index.js",lang="javascript"}
~~~~~~~
import React, { Component } from 'react';
import { Link } from 'react-router-dom';

# leanpub-start-insert
import { FirebaseContext } from '../Firebase';
# leanpub-end-insert
import * as ROUTES from '../../constants/routes';

const SignUpPage = () => (
  <div>
    <h1>SignUp</h1>
# leanpub-start-insert
    <FirebaseContext.Consumer>
      {firebase => <SignUpForm firebase={firebase} />}
    </FirebaseContext.Consumer>
# leanpub-end-insert
  </div>
);

const INITIAL_STATE = { ... };

class SignUpForm extends Component {
  ...
}

...
~~~~~~~

Now the registration of a new user should work. However, I'd like to make one improvement on how we access the Firebase instance here. Rather than using a [render prop component](https://www.robinwieruch.de/react-render-props-pattern/), which is automatically given with React's Context Consumer component, it may be simpler to use a [higher-order component](https://www.robinwieruch.de/gentle-introduction-higher-order-components/). Let's implement this higher-order component:

{title="src/components/Firebase/context.js",lang="javascript"}
~~~~~~~
import React from 'react';

const FirebaseContext = React.createContext(null);

# leanpub-start-insert
export const withFirebase = Component => props => (
  <FirebaseContext.Consumer>
    {firebase => <Component {...props} firebase={firebase} />}
  </FirebaseContext.Consumer>
);
# leanpub-end-insert

export default FirebaseContext;
~~~~~~~

Next, make it available via our Firebase module in the *src/components/Firebase/index.js* file:

{title="src/components/Firebase/index.js",lang="javascript"}
~~~~~~~
# leanpub-start-insert
import FirebaseContext, { withFirebase } from './context';
# leanpub-end-insert
import Firebase from './firebase';

export default Firebase;

# leanpub-start-insert
export { FirebaseContext, withFirebase };
# leanpub-end-insert
~~~~~~~

Now, instead of using the Firebase Context directly in the SignUpPage, which doesn't need to know about the Firebase instance, use the higher-order component to wrap your SignUpForm. Afterward, the SignUpForm has access to the Firebase instance via the higher-order component. It's also possible to use the SignUpForm as standalone without the SignUpPage, because it is responsible to get the Firebase instance via the higher-order component.

{title="src/components/SignUp/index.js",lang="javascript"}
~~~~~~~
import React, { Component } from 'react';
import { Link } from 'react-router-dom';

# leanpub-start-insert
import { withFirebase } from '../Firebase';
# leanpub-end-insert
import * as ROUTES from '../../constants/routes';

const SignUpPage = () => (
  <div>
    <h1>SignUp</h1>
# leanpub-start-insert
    <SignUpForm />
# leanpub-end-insert
  </div>
);

const INITIAL_STATE = { ... };

# leanpub-start-insert
class SignUpFormBase extends Component {
# leanpub-end-insert
  ...
}

const SignUpLink = () => ...

# leanpub-start-insert
const SignUpForm = withFirebase(SignUpFormBase);
# leanpub-end-insert

export default SignUpPage;

export { SignUpForm, SignUpLink };
~~~~~~~

When a user signs up to your application, you want to redirect the user to another page. It could be the user's home page, a protected route for only authenticated users. You will need the help of React Router to redirect the user after a successful sign up.

{title="src/components/SignUp/index.js",lang="javascript"}
~~~~~~~
import React, { Component } from 'react';
# leanpub-start-insert
import { Link, withRouter } from 'react-router-dom';
# leanpub-end-insert

import { withFirebase } from '../Firebase';
import * as ROUTES from '../../constants/routes';

...

class SignUpFormBase extends Component {

  ...

  onSubmit = (event) => {
    const { username, email, passwordOne } = this.state;

    this.props.firebase
      .doCreateUserWithEmailAndPassword(email, passwordOne)
      .then(authUser => {
        this.setState({ ...INITIAL_STATE });
# leanpub-start-insert
        this.props.history.push(ROUTES.HOME);
# leanpub-end-insert
      })
      .catch(error => {
        this.setState({ error });
      });

    event.preventDefault();
  }

  ...
}

...

# leanpub-start-insert
const SignUpForm = withRouter(withFirebase(SignUpFormBase));
# leanpub-end-insert

export default SignUpPage;

export { SignUpForm, SignUpLink };
~~~~~~~

Let's take the previous code block apart again. To redirect a user to another page programmatically, we need access to React Router to redirect the user to another page. Fortunately, the React Router node package offers a higher-order component to make the router properties accessible in the props of a component. Any component that goes in the `withRouter()` higher-order component gains access to all the properties of the router, so when passing the enhanced SignUpFormBase component to the `withRouter()` higher-order component, it has access to the props of the router. The relevant property from the router props is the `history` object, because it allows us to redirect a user to another page by pushing a route to it.

The history object of the router can be used in the `onSubmit()` class method eventually. If a request resolves successfully, you can push any route to the history object. Since the pushed */home* route is defined in our App component with a matching component to be rendered, the displayed page component will change after the redirect.

There is one improvement that we can make for the higher-order components used for the SignUpForm. Nesting functions (higher-order components) into each other like we did before can become verbose. A better way is to compose the higher-order components instead. To do this, install [recompose](https://github.com/acdlite/recompose) for your application on the command line:

{title="Command Line",lang="json"}
~~~~~~~
npm install recompose
~~~~~~~

You can use recompose to organize your higher-order components. Since the higher-order components don't depend on each other, the order doesn't matter. Otherwise, it may be good to know that the compose function applies the higher-order components from right to left.

{title="src/components/SignUp/index.js",lang="javascript"}
~~~~~~~
import React, { Component } from 'react';
import { Link, withRouter } from 'react-router-dom';
# leanpub-start-insert
import { compose } from 'recompose';
# leanpub-end-insert

import { withFirebase } from '../Firebase';
import * as ROUTES from '../../constants/routes';

...

# leanpub-start-insert
const SignUpForm = compose(
  withRouter,
  withFirebase,
)(SignUpFormBase);
# leanpub-end-insert

export default SignUpPage;

export { SignUpForm, SignUpLink };
~~~~~~~

Run your application again. If you signed up a user successfully, it should redirect to the home page. If the sign up fails, you should see an error message. Try to sign up a user with the same email address twice and verify that a similar error message shows up: "The email address is already in use by another account.". Congratulations, you signed up your first user via Firebase authentication.

### Exercises:

* Read more about [data fetching in React](https://www.robinwieruch.de/react-fetching-data/)
* Read more about [higher-order components in React](https://www.robinwieruch.de/gentle-introduction-higher-order-components/)
* Read more about [render prop components in React](https://www.robinwieruch.de/react-render-props-pattern/)
* Confirm your [source code for the last section](http://bit.ly/2VkrTEA)

## Sign In with React and Firebase

A sign up automatically results in a sign in/login by the user. We cannot rely on this mechanic, however, since a user could be signed up but not signed in. Let's implement the login with Firebase now. It is similar to the sign up mechanism and components, so this time we won't split it into so many code blocks:

{title="src/components/SignIn/index.js",lang="javascript"}
~~~~~~~
import React, { Component } from 'react';
import { withRouter } from 'react-router-dom';
import { compose } from 'recompose';

import { SignUpLink } from '../SignUp';
import { withFirebase } from '../Firebase';
import * as ROUTES from '../../constants/routes';

const SignInPage = () => (
  <div>
    <h1>SignIn</h1>
    <SignInForm />
    <SignUpLink />
  </div>
);

const INITIAL_STATE = {
  email: '',
  password: '',
  error: null,
};

class SignInFormBase extends Component {
  constructor(props) {
    super(props);

    this.state = { ...INITIAL_STATE };
  }

  onSubmit = event => {
    const { email, password } = this.state;

    this.props.firebase
      .doSignInWithEmailAndPassword(email, password)
      .then(() => {
        this.setState({ ...INITIAL_STATE });
        this.props.history.push(ROUTES.HOME);
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
    const { email, password, error } = this.state;

    const isInvalid = password === '' || email === '';

    return (
      <form onSubmit={this.onSubmit}>
        <input
          name="email"
          value={email}
          onChange={this.onChange}
          type="text"
          placeholder="Email Address"
        />
        <input
          name="password"
          value={password}
          onChange={this.onChange}
          type="password"
          placeholder="Password"
        />
        <button disabled={isInvalid} type="submit">
          Sign In
        </button>

        {error && <p>{error.message}</p>}
      </form>
    );
  }
}

const SignInForm = compose(
  withRouter,
  withFirebase,
)(SignInFormBase);

export default SignInPage;

export { SignInForm };
~~~~~~~

It is almost the same as the sign up form. Its input fields capture all the necessary information like username and password. A validation step makes sure the email and password are set before performing the request by enabling or disabling the submit button. The authentication API is used again, this time with a function to sign in the user rather than sign them up. If sign in succeeds, the local state is updated with the initial state and the user is redirected again. If the sign in fails, an error object is stored in the local state and an error message appears. The SignUpLink, which was defined earlier in the SignUp module, is used on the sign in page. It lets users sign up if they don't have an account, and it is found on the sign in page.

### Exercises:

* Familiarize yourself with the SignIn and SignInForm components.
  * If they are mysterious to you, checkout the previous section with the implementation of the SignUpForm again
* Confirm your [source code for the last section](http://bit.ly/2VnEzup)

## Sign Out with React and Firebase

To complete the authentication loop, next we'll implement the sign out component. The component is just a button that appears within the Navigation component. Since we can use the previously-defined authentication API to sign out a user, passing functionality to a button in a React component is fairly straightforward.

{title="src/components/SignOut/index.js",lang="javascript"}
~~~~~~~
import React from 'react';

import { withFirebase } from '../Firebase';

const SignOutButton = ({ firebase }) => (
  <button type="button" onClick={firebase.doSignOut}>
    Sign Out
  </button>
);

export default withFirebase(SignOutButton);
~~~~~~~

The SignOutButton has access to the Firebase instance using the higher-order component again. Now, use the SignOutButton in the Navigation component:

{title="src/components/Navigation/index.js",lang="javascript"}
~~~~~~~
import React from 'react';
import { Link } from 'react-router-dom';

# leanpub-start-insert
import SignOutButton from '../SignOut';
# leanpub-end-insert
import * as ROUTES from '../../constants/routes';

const Navigation = () => (
  <div>
    <ul>
      <li>
        <Link to={ROUTES.SIGN_IN}>Sign In</Link>
      </li>
      <li>
        <Link to={ROUTES.LANDING}>Landing</Link>
      </li>
      <li>
        <Link to={ROUTES.HOME}>Home</Link>
      </li>
      <li>
        <Link to={ROUTES.ACCOUNT}>Account</Link>
      </li>
      <li>
        <Link to={ROUTES.ADMIN}>Admin</Link>
      </li>
# leanpub-start-insert
      <li>
        <SignOutButton />
      </li>
# leanpub-end-insert
    </ul>
  </div>
);

export default Navigation;
~~~~~~~

Regarding components, everything is set to fulfil a full authentication roundtrip. Users can sign up (register), sign in (login), and sign out (logout).

### Exercises:

* Read more about [Firebase Authentication with E-Mail/Password](https://firebase.google.com/docs/auth/web/password-auth)
* Confirm your [source code for the last section](http://bit.ly/2VpQ9oO)

## Session Handling

This section is the most important one for the authentication process. You have all the components needed to fulfil an authentication roundtrip in React, and all that's missing is an overseer for the session state.  Logic regarding the current authenticated user needs to be stored and made accessible to other components. This is often the point where developers start to use a state management library like [Redux or MobX](https://www.robinwieruch.de/redux-mobx-confusion/). Without these, we'll make due using [global state](https://www.robinwieruch.de/react-global-state-without-redux/) instead of state management libraries.

Since our application is made under the umbrella of App component, it's sufficient to manage the session state in the App component using React's local state. The App component only needs to keep track of an authenticated user (session). If a user is authenticated, store it in the local state and pass the authenticated user object down to all components that are interested in it. Otherwise, pass the authenticated user down as `null`. That way, all components interested in it can adjust their behavior (e.g. use conditional rendering) based on the session state. For instance, the Navigation component is interested because it has to show different options to authenticated and non authenticated users. The SignOut component shouldn't show up for a non authenticated user, for example.

We handle session handling in the App component in the *src/components/App/index.js* file. Because the component handles local state now, you have to refactor it to a class component. It manages the local state of a `authUser` object, and then passes it to the Navigation component.

{title="src/components/App/index.js",lang="javascript"}
~~~~~~~
# leanpub-start-insert
import React, { Component } from 'react';
# leanpub-end-insert
import { BrowserRouter as Router, Route } from 'react-router-dom';

...

# leanpub-start-insert
class App extends Component {
  constructor(props) {
    super(props);

    this.state = {
      authUser: null,
    };
  }
# leanpub-end-insert

# leanpub-start-insert
  render() {
    return (
# leanpub-end-insert
      <Router>
        <div>
# leanpub-start-insert
          <Navigation authUser={this.state.authUser} />
# leanpub-end-insert

          <hr/>

          ...
        </div>
      </Router>
# leanpub-start-insert
    );
  }
}
# leanpub-end-insert

export default App;
~~~~~~~

The Navigation component can be made aware of authenticated user to display different options. It should either show the available links for an authenticated user or a non authenticated user.

{title="src/components/Navigation/index.js",lang="javascript"}
~~~~~~~
import React from 'react';
import { Link } from 'react-router-dom';

import SignOutButton from '../SignOut';
import * as ROUTES from '../../constants/routes';

# leanpub-start-insert
const Navigation = ({ authUser }) => (
  <div>{authUser ? <NavigationAuth /> : <NavigationNonAuth />}</div>
);
# leanpub-end-insert

# leanpub-start-insert
const NavigationAuth = () => (
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
    <li>
      <SignOutButton />
    </li>
  </ul>
);
# leanpub-end-insert

# leanpub-start-insert
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
# leanpub-end-insert

export default Navigation;
~~~~~~~

Let's see where the `authUser` (authenticated user) comes from in the App component. Firebase offers a listener function to get the authenticated user from Firebase:

{title="src/components/App/index.js",lang="javascript"}
~~~~~~~
...

import * as ROUTES from '../constants/routes';
# leanpub-start-insert
import { withFirebase } from '../Firebase';
# leanpub-end-insert

class App extends Component {
  constructor(props) {
    super(props);

    this.state = {
      authUser: null,
    };
  }

# leanpub-start-insert
  componentDidMount() {
    this.props.firebase.auth.onAuthStateChanged(authUser => {
      authUser
        ? this.setState({ authUser })
        : this.setState({ authUser: null });
    });
  }
# leanpub-end-insert

  ...

}

# leanpub-start-insert
export default withFirebase(App);
# leanpub-end-insert
~~~~~~~

The helper function `onAuthStateChanged()` receives a function as parameter that has access to the authenticated user. Also, the passed function is called every time something changes for the authenticated user. It is called when a user signs up, signs in, and signs out. If a user signs out, the `authUser` object becomes null, so the `authUser` property in the local state is set to null and all components depending on it adjust their behavior (e.g. display different options like the Navigation component).

We also want to avoid memory leaks that lead to [performance issues](https://www.robinwieruch.de/react-warning-cant-call-setstate-on-an-unmounted-component/), so we'll remove the listener if the component unmounts.

{title="src/components/App/index.js",lang="javascript"}
~~~~~~~
...

class App extends Component {
  ...

  componentDidMount() {
# leanpub-start-insert
    this.listener = this.props.firebase.auth.onAuthStateChanged(
# leanpub-end-insert
      authUser => {
        authUser
          ? this.setState({ authUser })
          : this.setState({ authUser: null });
      },
    );
  }

# leanpub-start-insert
  componentWillUnmount() {
    this.listener();
  }
# leanpub-end-insert

  ...

}

export default withFirebase(App);
~~~~~~~

Start your application and verify that your sign up, sign in, and sign out functionality works, and that the Navigation component displays the options depending on the session state (authenticated user).

Congratulations, you have successfully implemented the authentication process with Firebase in React. Everything in the following sections reagrding authentication is considered extra, to improve the developer's experience and add a couple of useful features along the way.

### Exercises:

* Read more about [Firebase's Authenticated User](https://firebase.google.com/docs/auth/web/manage-users)
* Confirm your [source code for the last section](http://bit.ly/2VrULL2)

## Session Handling with Higher-Order Components

We added a basic version of session handling in the last section. However,  the authenticated user still needs to be passed down from the App component to interested parties. That can become tedious over time, because the authenticated user has to be passed through all components until it reaches all the leaf components. You used the React Context API to pass down the Firebase instance to any component before. Here, you will do the same for the authenticated user. In a new *src/components/Session/context.js* file, place the following new React Context for the session (authenticated user):

{title="src/components/Session/context.js",lang="javascript"}
~~~~~~~
import React from 'react';

const AuthUserContext = React.createContext(null);

export default AuthUserContext;
~~~~~~~

Next, import and export it from the *src/components/Session/index.js* file that is the entry point to this module:

{title="src/components/Session/index.js",lang="javascript"}
~~~~~~~
import AuthUserContext from './context';

export { AuthUserContext };
~~~~~~~

The App component can use the new context to provide the authenticated user to components that are interested in it:

{title="src/components/App/index.js",lang="javascript"}
~~~~~~~
...

# leanpub-start-insert
import { AuthUserContext } from '../Session';
# leanpub-end-insert

class App extends Component {
  ...

  render() {
    return (
# leanpub-start-insert
      <AuthUserContext.Provider value={this.state.authUser}>
# leanpub-end-insert
        <Router>
          <div>
# leanpub-start-insert
            <Navigation />
# leanpub-end-insert

            <hr />

            ...
          </div>
        </Router>
# leanpub-start-insert
      </AuthUserContext.Provider>
# leanpub-end-insert
    );
  }
}

export default withFirebase(App);
~~~~~~~

The `authUser` doesn't need to be passed to the Navigation component anymore. Instead, the Navigation component uses the new context to consume the authenticated user:

{title="src/components/Navigation/index.js",lang="javascript"}
~~~~~~~
...

# leanpub-start-insert
import { AuthUserContext } from '../Session';
# leanpub-end-insert

# leanpub-start-insert
const Navigation = () => (
# leanpub-end-insert
  <div>
# leanpub-start-insert
    <AuthUserContext.Consumer>
      {authUser =>
        authUser ? <NavigationAuth /> : <NavigationNonAuth />
      }
    </AuthUserContext.Consumer>
# leanpub-end-insert
  </div>
);
~~~~~~~

The application works the same as before, except any component can simply use React's Context to consume the authenticated user. To keep the App component clean and concise, I like to extract the session handling for the authenticated user to a separate higher-order component in a new *src/components/Session/withAuthentication.js* file:

{title="src/components/Session/withAuthentication.js",lang="javascript"}
~~~~~~~
import React from 'react';

const withAuthentication = Component => {
  class WithAuthentication extends React.Component {
    render() {
      return <Component {...this.props} />;
    }
  }

  return WithAuthentication;
};

export default withAuthentication;
~~~~~~~

Move all logic that deals with the authenticated user from the App component to it:

{title="src/components/Session/withAuthentication.js",lang="javascript"}
~~~~~~~
import React from 'react';

# leanpub-start-insert
import AuthUserContext from './context';
import { withFirebase } from '../Firebase';
# leanpub-end-insert

const withAuthentication = Component => {
  class WithAuthentication extends React.Component {
# leanpub-start-insert
    constructor(props) {
      super(props);

      this.state = {
        authUser: null,
      };
    }
# leanpub-end-insert

# leanpub-start-insert
    componentDidMount() {
      this.listener = this.props.firebase.auth.onAuthStateChanged(
        authUser => {
          authUser
            ? this.setState({ authUser })
            : this.setState({ authUser: null });
        },
      );
    }

    componentWillUnmount() {
      this.listener();
    }
# leanpub-end-insert

    render() {
# leanpub-start-insert
      return (
        <AuthUserContext.Provider value={this.state.authUser}>
          <Component {...this.props} />
        </AuthUserContext.Provider>
      );
# leanpub-end-insert
    }
  }

# leanpub-start-insert
  return withFirebase(WithAuthentication);
# leanpub-end-insert
};

export default withAuthentication;
~~~~~~~

As you can see, it also uses the new React Context to provide the authenticated user. The App component will not be in charge of it anymore. Next, export the higher-order component from the *src/components/Session/index.js* file, so that it can be used in the App component after:

{title="src/components/Session/index.js",lang="javascript"}
~~~~~~~
import AuthUserContext from './context';
# leanpub-start-insert
import withAuthentication from './withAuthentication';
# leanpub-end-insert

# leanpub-start-insert
export { AuthUserContext, withAuthentication };
# leanpub-end-insert
~~~~~~~

The App component becomes a function component again, without the additional business logic for the authenticated user. Now, it uses the higher-order component to make the authenticated user available for all other components below of the App component:

{title="src/components/App/index.js",lang="javascript"}
~~~~~~~
# leanpub-start-insert
import React from 'react';
# leanpub-end-insert
import { BrowserRouter as Router, Route } from 'react-router-dom';

import Navigation from '../Navigation';
import LandingPage from '../Landing';
import SignUpPage from '../SignUp';
import SignInPage from '../SignIn';
import PasswordForgetPage from '../PasswordForget';
import HomePage from '../Home';
import AccountPage from '../Account';
import AdminPage from '../Admin';

import * as ROUTES from '../../constants/routes';
# leanpub-start-insert
import { withAuthentication } from '../Session';
# leanpub-end-insert

# leanpub-start-insert
const App = () => (
# leanpub-end-insert
  <Router>
    <div>
      <Navigation />

      <hr />

      <Route exact path={ROUTES.LANDING} component={LandingPage} />
      <Route path={ROUTES.SIGN_UP} component={SignUpPage} />
      <Route path={ROUTES.SIGN_IN} component={SignInPage} />
      <Route
        path={ROUTES.PASSWORD_FORGET}
        component={PasswordForgetPage}
      />
      <Route path={ROUTES.HOME} component={HomePage} />
      <Route path={ROUTES.ACCOUNT} component={AccountPage} />
      <Route path={ROUTES.ADMIN} component={AdminPage} />
    </div>
  </Router>
# leanpub-start-insert
);
# leanpub-end-insert

# leanpub-start-insert
export default withAuthentication(App);
# leanpub-end-insert
~~~~~~~

Start the application and verify that it still works. You didn't change any behavior in this section, but shielded away the more complex logic into a higher-order component. Also, the application now passes the authenticated user implicitly via React's Context, rather than explicitly through the component tree using props.

### Exercises:

* Check again your Firebase Context and higher-order component implementation in the *src/components/Firebase* module, which is quite similar to what you have done in this section.
* Confirm your [source code for the last section](http://bit.ly/2VjYz0R)