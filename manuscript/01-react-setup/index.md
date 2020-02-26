# Application Setup

Let's get started with the React + Firebase application we are going to build together. The application should be the perfect starter project to realize your ideas. It should be possible to display information with React, to navigate from URL to URL with React Router and to store and retrieve data with Firebase. Also the application will have everything that's needed to register, login and logout users. In the end, you should be able to implement any feature on top of this application to create well-rounded React applications.

If you lack information on how to setup your React development environment, checkout these setup guides for [MacOS](https://www.robinwieruch.de/react-js-macos-setup/) and [Windows](https://www.robinwieruch.de/react-js-windows-setup/). Now, there are two ways to begin with this application: either follow my guidance in this section; or find a starter project in this [GitHub repository](https://github.com/the-road-to-react-with-firebase/react-firebase-authentication-starter-kit) and follow its installation instructions. This section will show how to set up the same project from scratch, whereas the starter project grants instant access without setting up the folder/file structure yourself.

The application we are going to build with React and Firebase will be set up with Facebook's official React boilerplate project, called [create-react-app](https://github.com/facebookincubator/create-react-app). You can set up your project with it on the command line whereas the name for the project is up to you. Afterward, navigate on the command line into the project:

{title="Command Line",lang="json"}
~~~~~~~
npx create-react-app react-firebase-authentication
cd react-firebase-authentication
~~~~~~~

Now you have the following command on your command line to start your application. You can start your application and visit it in the browser:

{title="Command Line",lang="json"}
~~~~~~~
npm start
~~~~~~~

Now we'll set up the project for our needs. First, get rid of the files from the boilerplate React project, since we won't be using them. From the command line, head to your *src/* folder and execute it:

{title="Command Line",lang="json"}
~~~~~~~
cd src
rm App.js App.test.js App.css logo.svg
~~~~~~~

Second, create a *components/* folder in your application's *src/* folder on the command line. This is where all your components will be implemented. Also, the App component that you have removed in the previous step will be recreated here:

{title="Command Line",lang="json"}
~~~~~~~
mkdir components
~~~~~~~

Create a dedicated folder for each component we will implement for this application. For the sake of readability, I split up the commands into multiple lines:

{title="Command Line",lang="json"}
~~~~~~~
cd components
mkdir Account Admin App Home Landing SignIn SignOut SignUp
mkdir Navigation PasswordChange PasswordForget
mkdir Session Firebase
~~~~~~~

In each folder, create an *index.js* file for the component. Navigate into a folder, create the file, and navigate out again. Repeat these steps for every component. You can choose to name your folders/files differently, but that's how I liked to do it for my applications.

{title="Command Line",lang="json"}
~~~~~~~
cd App
touch index.js
cd ..
~~~~~~~

Next, implement a basic React component for each file you created. For the App component in *src/components/App/index.js*, it could look like the following:

{title="src/components/App/index.js",lang="javascript"}
~~~~~~~
import React from 'react';

const App = () => (
  <div>
    <h1>App</h1>
  </div>
);

export default App;
~~~~~~~

Fix the relative path to the App component in the *src/index.js* file. Since you have moved the App component to the *src/components* folder, you need to add the */components* subpath to it.

{title="src/index.js",lang="javascript"}
~~~~~~~
import React from 'react';
import ReactDOM from 'react-dom';

import './index.css';
import * as serviceWorker from './serviceWorker';

# leanpub-start-insert
import App from './components/App';
# leanpub-end-insert

ReactDOM.render(<App />, document.getElementById('root'));

serviceWorker.unregister();
~~~~~~~

Then, create one more folder in your *src/* folder:

{title="Command Line",lang="json"}
~~~~~~~
mkdir constants
~~~~~~~

The folder should be located next to *src/components/*. Move into *src/constants/*  and create two files for the application's routing and roles management later:

{title="Command Line",lang="json"}
~~~~~~~
cd constants
touch routes.js roles.js
cd ..
~~~~~~~

The application with its folders and files is set up, and you can verify this by running it on the command line and accessing it through a browser. Check the starter project on GitHub I linked in the beginning of this section to verify whether you have set up everything properly.

### Exercises:

* Familiarize yourself with the folder structure of a project.
* Optionally, introduce a test for your App component and test the application.
* Optionally, introduce [CSS Modules](https://www.robinwieruch.de/create-react-app-css-modules/), [SASS](https://www.robinwieruch.de/create-react-app-with-sass-support/) or [Styled Components](https://www.styled-components.com) and style the application.
* Optionally, introduce [Git and keep track of your changes by having your project on GitHub](https://www.robinwieruch.de/git-essential-commands/).