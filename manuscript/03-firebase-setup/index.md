# Firebase

The main focus here is using Firebase in React for the application we'll build together. Firebase, bought by Google in 2014, enables realtime databases, extensive authentication and authorization, and even for deployment. You can build real-world applications with React and Firebase without worrying about implementing a backend application. All the things a backend application would handle, like authentication and a database, is handled by Firebase. Many businesses use React and Firebase to power their applications, as it is the ultimate combination to launch an [MVP](https://en.wikipedia.org/wiki/Minimum_viable_product).

To start, sign up on the [official Firebase website](https://firebase.google.com/). After you have created a Firebase account, you should be able to create projects and be granted access to the project dashboard. We'll begin by creating a project for this application on their platform whereas the project can have any name. In the case of this application, run it on the free pricing plan. If you want to scale your application later, you can change the plan. Follow this [visual Firebase setup and introduction guide](https://www.robinwieruch.de/firebase-tutorial) to learn more about Firebase's dashboard and features. It would also give you first guidelines on how to acivate Firebase's Realtime Database instead of Firebase's Cloud Firestore.

Next, find the project's configuration in the settings on your project's dashboard. There, you'll have access to all the necessary information: secrets, keys, ids and other details to set up your application. Copy these in the next step to your React application.

![](images/firebase-config_1024.jpg)

Sometimes the Firebase website doesn't make it easy to find this page. Since it's moved around with every iteration of the website, I cannot give you any clear advice where to find it on your dashboard. This is an opportunity to familiarize yourself with Firebase project's dashboard while you search for the configuration.

![](images/firebase-web-settings_1024.jpg)

Now that we've completed the Firebase setup, you can return to your application in your editor/IDE to add the Firebase configuration. First, install Firebase for your application on the command line:

{title="Command Line",lang="json"}
~~~~~~~
npm install firebase
~~~~~~~

Next, we'll create a new file for the Firebase setup. We will use a JavaScript class to encapsulate all Firebase functionalities, realtime database, and authentication, as a well-defined API for the rest of the application. You need only instantiate the class once, after which it can use it then to interact with the Firebase API, your custom Firebase interface.

Let's start by copying the configuration from your Firebase project's dashboard on their website to your application as a configuration object in a new *src/components/Firebase/firebase.js* file. Make sure to replace the capitalized keys with the corresponding keys from your copied configuration:

{title="src/components/Firebase/firebase.js",lang="javascript"}
~~~~~~~
const config = {
  apiKey: YOUR_API_KEY,
  authDomain: YOUR_AUTH_DOMAIN,
  databaseURL: YOUR_DATABASE_URL,
  projectId: YOUR_PROJECT_ID,
  storageBucket: '',
  messagingSenderId: YOUR_MESSAGING_SENDER_ID,
};
~~~~~~~

As alternative, you can also use environment variables in React applications, but you have to use the `REACT_APP` prefix when you use create-react-app to set up the application:

{title="src/components/Firebase/firebase.js",lang="javascript"}
~~~~~~~
const config = {
# leanpub-start-insert
  apiKey: process.env.REACT_APP_API_KEY,
  authDomain: process.env.REACT_APP_AUTH_DOMAIN,
  databaseURL: process.env.REACT_APP_DATABASE_URL,
  projectId: process.env.REACT_APP_PROJECT_ID,
  storageBucket: process.env.REACT_APP_STORAGE_BUCKET,
  messagingSenderId: process.env.REACT_APP_MESSAGING_SENDER_ID,
# leanpub-end-insert
};
~~~~~~~

Now you can define the environmental variables in a new *.env* file in your project's root folder. The *.env* file can also be added to your *.gitginore* file (in case you are using git), so your Firebase credentials are not exposed publicly on a platform like GitHub.

{title=".env",lang="text"}
~~~~~~~
REACT_APP_API_KEY=XXXXxxxx
REACT_APP_AUTH_DOMAIN=xxxxXXXX.firebaseapp.com
REACT_APP_DATABASE_URL=https://xxxXXXX.firebaseio.com
REACT_APP_PROJECT_ID=xxxxXXXX
REACT_APP_STORAGE_BUCKET=xxxxXXXX.appspot.com
REACT_APP_MESSAGING_SENDER_ID=xxxxXXXX
~~~~~~~

Both ways work. You can define the configuration inline in source code or as environment variables. Environmental variables are more secure, and should be used when uploading your project to a version control system like git, though we will be continuing with the Firebase setup. Import firebase from the library you installed earlier, and then use it within a new Firebase class to initialize firebase with the configuration:

{title="src/components/Firebase/firebase.js",lang="javascript"}
~~~~~~~
# leanpub-start-insert
import app from 'firebase/app';
# leanpub-end-insert

const config = {
  apiKey: process.env.REACT_APP_API_KEY,
  authDomain: process.env.REACT_APP_AUTH_DOMAIN,
  databaseURL: process.env.REACT_APP_DATABASE_URL,
  projectId: process.env.REACT_APP_PROJECT_ID,
  storageBucket: process.env.REACT_APP_STORAGE_BUCKET,
  messagingSenderId: process.env.REACT_APP_MESSAGING_SENDER_ID,
};

# leanpub-start-insert
class Firebase {
  constructor() {
    app.initializeApp(config);
  }
}
# leanpub-end-insert

# leanpub-start-insert
export default Firebase;
# leanpub-end-insert
~~~~~~~

That's all that is needed for a firebase configuration in your application. Optionally, you can create a second Firebase project on the Firebase website to have one project for your development environment and one project for your production environment. That way, you never mix data in the Firebase database in development mode with data from your deployed application (production mode). If you decide to create projects for both environments, use the two configuration objects in your Firebase setup and decide which one you take depending on the development/production environment:

{title="src/components/Firebase/firebase.js",lang="javascript"}
~~~~~~~
import app from 'firebase/app';

# leanpub-start-insert
const prodConfig = {
  apiKey: process.env.REACT_APP_PROD_API_KEY,
  authDomain: process.env.REACT_APP_PROD_AUTH_DOMAIN,
  databaseURL: process.env.REACT_APP_PROD_DATABASE_URL,
  projectId: process.env.REACT_APP_PROD_PROJECT_ID,
  storageBucket: process.env.REACT_APP_PROD_STORAGE_BUCKET,
  messagingSenderId: process.env.REACT_APP_PROD_MESSAGING_SENDER_ID,
};
# leanpub-end-insert

# leanpub-start-insert
const devConfig = {
  apiKey: process.env.REACT_APP_DEV_API_KEY,
  authDomain: process.env.REACT_APP_DEV_AUTH_DOMAIN,
  databaseURL: process.env.REACT_APP_DEV_DATABASE_URL,
  projectId: process.env.REACT_APP_DEV_PROJECT_ID,
  storageBucket: process.env.REACT_APP_DEV_STORAGE_BUCKET,
  messagingSenderId: process.env.REACT_APP_DEV_MESSAGING_SENDER_ID,
};
# leanpub-end-insert

# leanpub-start-insert
const config =
  process.env.NODE_ENV === 'production' ? prodConfig : devConfig;
# leanpub-end-insert

class Firebase {
  constructor() {
    app.initializeApp(config);
  }
}

export default Firebase;
~~~~~~~

 An alternate way to implement this is to specify a dedicated *.env.development* and *.env.production* file for both kinds of environment variables in your project. Each file is used to define environmental variables for the matching environment. Defining a configuration becomes straightforward again, because you don't have to select the correct configuration yourself.

{title="src/components/Firebase/firebase.js",lang="javascript"}
~~~~~~~
import app from 'firebase/app';

# leanpub-start-insert
const config = {
  apiKey: process.env.REACT_APP_API_KEY,
  authDomain: process.env.REACT_APP_AUTH_DOMAIN,
  databaseURL: process.env.REACT_APP_DATABASE_URL,
  projectId: process.env.REACT_APP_PROJECT_ID,
  storageBucket: process.env.REACT_APP_STORAGE_BUCKET,
  messagingSenderId: process.env.REACT_APP_MESSAGING_SENDER_ID,
};
# leanpub-end-insert

class Firebase {
  constructor() {
    app.initializeApp(config);
  }
}

export default Firebase;
~~~~~~~

Whether you used environment variables, defined the configuration inline, used only one Firebase project, or multiple projects for each environment, you configured Firebase for your React application. The next section will show you how a Firebase instance created from the Firebase class is used in React.

### Exercises:

* Read more about the [Firebase Setup for Web Applications](https://firebase.google.com/docs/web/setup)
* Read more about [Firebase's Pricing Plans](https://firebase.google.com/pricing/) to know better about the limitations of the free plan.
* Confirm your [source code for the last section](http://bit.ly/2VqqWKP)