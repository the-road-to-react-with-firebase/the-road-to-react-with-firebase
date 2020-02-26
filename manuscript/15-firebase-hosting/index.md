# Firebase Hosting

After we built a full-fledged application in React, the final step is deployment, the tipping point of getting your ideas out into the world, from consuming tutorials to producing applications. We will use Firebase Hosting for the deployment.

In this section, I want to guide you through deploying your React application to Firebase. It works for create-react-app too. Also it should work for any other library and framework such as Angular or Vue. First, install the Firebase CLI globally to your node modules:

{title="Command Line",lang="json"}
~~~~~~~
npm install -g firebase-tools
~~~~~~~

Using a global installation of the Firebase CLI, you can deploy any application without worrying about the dependency in your project. For any global installed node package, remember to update it occasionally to a newer version with the identical command:

{title="Command Line",lang="json"}
~~~~~~~
npm install -g firebase-tools
~~~~~~~

Next associate the Firebase CLI with a Firebase account (Google account):

{title="Command Line",lang="json"}
~~~~~~~
firebase login
~~~~~~~

There should be a URL in your command line that opens in a browser. If this doesn't happen, Firebase CLI may open up the URL automatically. Choose your Google account to create a Firebase project, and give Google the necessary permissions. You should see a confirmation for a successful setup. Return to the command line to verify a successful login.

Next, move to the project's folder and execute the following command, which initializes a Firebase project that can be used for the Firebase hosting features:

{title="Command Line",lang="json"}
~~~~~~~
firebase init
~~~~~~~

Then, choose the Hosting option. If you are interested in using another tool to host your Firebase application, choose another option:

{title="Command Line",lang="json"}
~~~~~~~
? Which Firebase CLI features do you want to setup for this folder? Press Space to select features, then Enter to confirm your choices.
 ◯ Database: Deploy Firebase Realtime Database Rules
 ◯ Firestore: Deploy rules and create indexes for Firestore
 ◯ Functions: Configure and deploy Cloud Functions
❯◯ Hosting: Configure and deploy Firebase Hosting sites
 ◯ Storage: Deploy Cloud Storage security rules
~~~~~~~

Since Google knows about Firebase projects associated with your account after logged in, you are able to select your Firebase project from a list of projects from the Firebase platform:

{title="Command Line",lang="json"}
~~~~~~~
First, let's associate this project directory with a Firebase project.
You can create multiple project aliases by running firebase use --add,
but for now we'll just set up a default project.

? Select a default Firebase project for this directory:
-> my-react-project-abc123 (my-react-project)
i  Using project my-react-project-abc123 (my-react-project)
~~~~~~~

There are a few other configuration steps to define. Instead of using the default *public/* folder, we want to use the *build/* folder for create-react-app. If you set up the bundling with a tool like Webpack, you can choose the appropriate name for the build folder:

{title="Command Line",lang="json"}
~~~~~~~
? What do you want to use as your public directory? build
? Configure as a single-page app (rewrite all urls to /index.html)? Yes
? File public/index.html already exists. Overwrite? No
~~~~~~~

The create-react-app application creates a *build/* folder after you perform the `npm run build` for the first time. There you will find all the merged content from the *public/* folder and the *src/* folder. Since it is a single page application, we want to redirect the user always to the *index.html* file. From there React Router takes over for the client-side routing.

Now your Firebase initialization is complete. This step created a few configuration files for Firebase Hosting in your project's folder. You can read more about them in [Firebase's documentation](https://firebase.google.com/docs/hosting/full-config) for configuring redirects, a 404 page, or headers. Finally, deploy your React application with Firebase on the command line:

{title="Command Line",lang="json"}
~~~~~~~
firebase deploy
~~~~~~~

After a successful deployment, you should see a similar output with your project's identifier:

{title="Command Line",lang="json"}
~~~~~~~
Project Console: https://console.firebase.google.com/project/my-react-project-abc123/overview
Hosting URL: https://my-react-project-abc123.firebaseapp.com
~~~~~~~

Visit both pages to observe the results. The former link navigates to your Firebase project's dashboard. There, you should have a new panel for the Firebase Hosting. The latter link navigates to your deployed React application.

If you only see a blank page for your deployed React application, see if the `public` key/value pair in the *firebase.json* is set to `build`. That's the case if your build folder has the name *build*. If it has another name, set the value to this. Second, check if you have ran the build script of your React app with `npm run build`. And third, if there is still a problem, check out the [official troubleshoot area for deploying create-react-app applications to Firebase](https://create-react-app.dev/docs/deployment). Afterward, try another deployment with `firebase deploy`. That should get your recent React build up and running for Firebase Hosting.

### Exercises

* Add the [security rules from the installation instructions](https://github.com/the-road-to-react-with-firebase/react-firebase-authentication) to your Firebase Project's Dashboard for your Database.
* Read more about [Firebase Hosting](https://firebase.google.com/docs/hosting/).
* [Connect your domain to your Firebase deployed application](https://firebase.google.com/docs/hosting/custom-domain).
