# React Router in React

Since we are building a larger application in the following sections, it would be great to have a couple of pages (e.g. landing page, account page, admin page, sign up page, sign in page) to split the application into multiple URLs (e.g. /landing, /account, /admin). These URLs or subpaths of a domain are called routes in a client-side web application. Let's implement the routing with [React Router](https://github.com/ReactTraining/react-router) before we dive into Firebase for the realtime database and authentication/authorization. If you haven't used React Router before, it should be straightforward to pick up the basics throughout building this application.

The application should have multiple routes. For instance, a user should be able to visit a public landing page, and also use sign up and sign in pages to enter the application as an authenticated user. If a user is authenticated, it is possible to visit protected pages like account or admin pages whereas the latter is only accessible by authenticated users with an admin role. You can consolidate all the routes of your application in a well-defined *src/constants/routes.js* constants file:

{title="src/constants/routes.js",lang="javascript"}
~~~~~~~
export const LANDING = '/';
export const SIGN_UP = '/signup';
export const SIGN_IN = '/signin';
export const HOME = '/home';
export const ACCOUNT = '/account';
export const ADMIN = '/admin';
export const PASSWORD_FORGET = '/pw-forget';
~~~~~~~

Each route represents a page in your application. For instance, the sign up page should be reachable in development mode via *http://localhost:3000/signup* and in production mode via *http://yourdomain/signup*.

First, you will have a **sign up page** (register page) and a **sign in page** (login page). You can take any web application as the blueprint to structure these routes for a well-rounded authentication experience. Take the following scenario: A user visits your web application, is convinced by your service, and finds the button in the top-level navigation to sign in to your application. But the user has no account yet, so a sign up button is presented as an alternative on the sign in page.

![](images/sign_1024.jpg)

Second, there will be a **landing page** and a **home page**. The landing page is your default route (e.g. *http://yourdomain/*). That's the place where a user ends up when visiting your web application. The user doesn't need to be authenticated to go this route. On the other hand, the home page is a **protected route**, which users can only access if they have been authenticated. You will implement the protection of the route using authorization mechanisms for this application.

Third, next to the **home page**, there will be protected **account page** and **admin page** as well. On the account page, a user can reset or change a password. It is secured by authorization as well, so it is only reachable for authenticated users. On the admin page, a user authorized as admin will be able to manage this application's users. The admin page is protected on a more fine-grained level, because it is only accessible for authenticated admin users.

![](images/account_1024.jpg)

Lastly, the **password forget** component will be exposed on another non-protected page, a **password forget page**, as well. It is used for users who are not authenticated and forgot about their password.

![](images/password-reset_1024.jpg)

We've completed the routes for this React with Firebase application. I find it exciting to build a well-rounded application with you, because it can be used as a boilerplate project that gives you authentication, authorization, and a database.  These are foundational pillars for any web-based application.

Now, all these routes need to be accessible to the user. First, you need a router for your web application, which is responsible to map routes to React components. React Router is a popular package to enable routing, so install it on the command line:

{title="Command Line",lang="json"}
~~~~~~~
npm install react-router-dom
~~~~~~~

The best way to start is implementing a Navigation component that will be used in the App component. The App component is the perfect place to render the Navigation component, because it always renders the Navigation component but replaces the other components (pages) based on the routes. Basically, the App component is the container where all your fixed components are going (e.g. navigation bar, side bar, footer), but also your components that are displayed depending on the route in the URL (e.g. account page, login page, password forget page).

First, the App component will use the Navigation component that is not implemented yet. Also, it uses the Router component provided by React Router. The Router makes it possible to navigate from URL-to-URL on the client-side application without another request to a web server for every route change. The application is only fetched once from a web server, after which all routing is done on the client-side with React Router.

{title="src/components/App/index.js",lang="javascript"}
~~~~~~~
import React from 'react';
# leanpub-start-insert
import { BrowserRouter as Router } from 'react-router-dom';
# leanpub-end-insert

# leanpub-start-insert
import Navigation from '../Navigation';
# leanpub-end-insert

const App = () => (
# leanpub-start-insert
  <Router>
    <Navigation />
  </Router>
# leanpub-end-insert
);

export default App;
~~~~~~~

Second, implement the Navigation component. It uses the Link component of React Router to enable navigation to different routes. These routes were defined previously in your constants file. Let's import all of them and give every Link component a specific route.

{title="src/components/Navigation/index.js",lang="javascript"}
~~~~~~~
import React from 'react';
# leanpub-start-insert
import { Link } from 'react-router-dom';
# leanpub-end-insert

# leanpub-start-insert
import * as ROUTES from '../../constants/routes';
# leanpub-end-insert

const Navigation = () => (
  <div>
# leanpub-start-insert
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
    </ul>
# leanpub-end-insert
  </div>
);

export default Navigation;
~~~~~~~

Now, run your application again and verify that the links show up in your browser, and that once you click a link, the URL changes. Notice that even though the URL changes, the displayed content doesn't change. The navigation is only there to enable navigation through your application. But no one knows what to render on each route. That's where the *route to component* mapping comes in. In your App component, you can specify which components should show up according to corresponding routes with the help of the Route component from React Router.

{title="src/components/App/index.js",lang="javascript"}
~~~~~~~
import React from 'react';
import {
  BrowserRouter as Router,
# leanpub-start-insert
  Route,
# leanpub-end-insert
} from 'react-router-dom';

import Navigation from '../Navigation';
# leanpub-start-insert
import LandingPage from '../Landing';
import SignUpPage from '../SignUp';
import SignInPage from '../SignIn';
import PasswordForgetPage from '../PasswordForget';
import HomePage from '../Home';
import AccountPage from '../Account';
import AdminPage from '../Admin';
# leanpub-end-insert

# leanpub-start-insert
import * as ROUTES from '../../constants/routes';
# leanpub-end-insert

const App = () => (
  <Router>
# leanpub-start-insert
    <div>
# leanpub-end-insert
      <Navigation />
# leanpub-start-insert

      <hr />

      <Route exact path={ROUTES.LANDING} component={LandingPage} />
      <Route path={ROUTES.SIGN_UP} component={SignUpPage} />
      <Route path={ROUTES.SIGN_IN} component={SignInPage} />
      <Route path={ROUTES.PASSWORD_FORGET} component={PasswordForgetPage} />
      <Route path={ROUTES.HOME} component={HomePage} />
      <Route path={ROUTES.ACCOUNT} component={AccountPage} />
      <Route path={ROUTES.ADMIN} component={AdminPage} />
    </div>
# leanpub-end-insert
  </Router>
);

export default App;
~~~~~~~

If a route matches a path prop, the respective component will be displayed; thus, all the page components in the App component are exchangeable by changing the route, but the Navigation component stays fixed independently of any route changes. This is how you enable a static frame with various components (e.g. Navigation) around your dynamic pages driven by routes. It's all made possible by [React's powerful composition](https://www.robinwieruch.de/react-component-composition/).

Previously, you created basic components for each page component used by our routes. Now you should be able to start the application again. When you click through the links in the Navigation component, the displayed page component should change according to the URL. The routes for the PasswordForget and SignUp components are not used in the Navigation component, but will be defined elsewhere later. For now, you have successfully implemented fundamental routing for this application.

### Exercises:

* Learn more about [React Router](https://reacttraining.com/react-router/web/guides/quick-start)
* Confirm your [source code for the last section](http://bit.ly/2VmQnNi)