# Firebase in React

You created a Firebase class, but you are not using it in your React application yet. In this section, we'll connect the Firebase with the React world. The simple approach is to create a Firebase instance with the Firebase class, and then import the instance (or class) in every React component where it's needed. That's not the best approach though, for two reasons:

* It is more difficult to test your React components.
* It is more error prone, because Firebase should only be initialized once in your application ([singleton](https://en.wikipedia.org/wiki/Singleton_pattern)) and by exposing the Firebase class to every React component, you could end up by mistake with multiple Firebase instances.

An alternative way is to use [React's Context API](https://www.robinwieruch.de/react-context-api/) to provide a Firebase instance once at the top-level of your component hierarchy. Create a new *src/components/Firebase/context.js* file in your Firebase module and provide the following implementation details:

{title="src/components/Firebase/context.js",lang="javascript"}
~~~~~~~
import React from 'react';

const FirebaseContext = React.createContext(null);

export default FirebaseContext;
~~~~~~~

The `createContext()` function essentially creates two components. The `FirebaseContext.Provider` component is used to provide a Firebase instance once at the top-level of your React component tree, which we will do in this section; and the `FirebaseContext.Consumer` component is used to retrieve the Firebase instance if it is needed in the React component. For a well-encapsulated Firebase module, we'll define a *index.js* file in our Firebase folder that exports all necessary functionalities (Firebase class, Firebase context for Consumer and Provider components):

{title="src/components/Firebase/index.js",lang="javascript"}
~~~~~~~
import FirebaseContext from './context';
import Firebase from './firebase';

export default Firebase;

export { FirebaseContext };
~~~~~~~

The Firebase Context from the Firebase module (folder) is used to provide a Firebase instance to your entire application in the *src/index.js* file. You only need to create the Firebase instance with the Firebase class and pass it as value prop to the React's Context:

{title="src/index.js",lang="javascript"}
~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';

import './index.css';
import * as serviceWorker from './serviceWorker';

import App from './components/App';
# leanpub-start-insert
import Firebase, { FirebaseContext } from './components/Firebase';
# leanpub-end-insert

ReactDOM.render(
# leanpub-start-insert
  <FirebaseContext.Provider value={new Firebase()}>
# leanpub-end-insert
    <App />
# leanpub-start-insert
  </FirebaseContext.Provider>,
# leanpub-end-insert
  document.getElementById('root'),
);

serviceWorker.unregister();
~~~~~~~

Doing it this way, we can be assured that Firebase is only instantiated once and that it is injected via React's Context API to React's component tree. Now, every component that is interested in using Firebase has access to the Firebase instance with a `FirebaseContext.Consumer` component. Even though you will see it first-hand later for this application, the following code snippet shows how it would work:

{title="Code Playground",lang="javascript"}
~~~~~~~
import React from 'react';

import  { FirebaseContext } from '../Firebase';

const SomeComponent = () => (
  <FirebaseContext.Consumer>
    {firebase => {
      return <div>I've access to Firebase and render something.</div>;
    }}
  </FirebaseContext.Consumer>
);

export default SomeComponent;
~~~~~~~

Firebase and React are now connected, the fundamental step to make the layers communicate with each other. Next, we will implement the interface for the Firebase class on our side to communicate with the Firebase API.

### Exercises:

* Read more about [React's Context API](https://www.robinwieruch.de/react-context-api/)
* Confirm your [source code for the last section](http://bit.ly/2VrUms0)
