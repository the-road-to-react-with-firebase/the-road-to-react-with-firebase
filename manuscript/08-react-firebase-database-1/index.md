# Firebase Realtime Database (1): Basic

So far, only Firebase knows about your users. There is no way to retrieve a single user or a list of users for your application from their authentication database. They are stored internally by Firebase to keep the authentication secure. That's good, because you are never involved in storing sensitive data like passwords. However, you can introduce the Firebase realtime database to keep track of user entities yourself. It makes sense, because then you can associate other domain entities (e.g. a message, a book, an invoice) created by your users to your users. You should keep control over your users, even though Firebase takes care about all the sensitive data. This section will explain how to store users in your realtime database in Firebase. First, initialize the realtime database API for your Firebase class as you did earlier for the authentication API:

{title="src/components/Firebase/firebase.js",lang="javascript"}
~~~~~~~
import app from 'firebase/app';
import 'firebase/auth';
# leanpub-start-insert
import 'firebase/database';
# leanpub-end-insert

const config = { ... };

class Firebase {
  constructor() {
    app.initializeApp(config);

    this.auth = app.auth();
# leanpub-start-insert
    this.db = app.database();
# leanpub-end-insert
  }

  // *** Auth API ***

  ...
}

export default Firebase;
~~~~~~~

Second, extend the interface for your Firebase class for the user entity. It defines two new functions: one to get a reference to a user by identifier (uid) and one to get a reference to all users:

{title="src/components/Firebase/firebase.js",lang="javascript"}
~~~~~~~
import app from 'firebase/app';
import 'firebase/auth';
import 'firebase/database';

const config = { ... };

class Firebase {
  constructor() {
    app.initializeApp(config);

    this.auth = app.auth();
    this.db = app.database();
  }

  // *** Auth API ***

  doCreateUserWithEmailAndPassword = (email, password) =>
    this.auth.createUserWithEmailAndPassword(email, password);

  doSignInWithEmailAndPassword = (email, password) =>
    this.auth.signInWithEmailAndPassword(email, password);

  doSignOut = () => this.auth.signOut();

  doPasswordReset = email => this.auth.sendPasswordResetEmail(email);

  doPasswordUpdate = password =>
    this.auth.currentUser.updatePassword(password);

# leanpub-start-insert
  // *** User API ***

  user = uid => this.db.ref(`users/${uid}`);

  users = () => this.db.ref('users');
# leanpub-end-insert
}

export default Firebase;
~~~~~~~

The paths in the `ref()` method match the location where your entities (users) will be stored in Firebase's realtime database API. If you delete a user at "users/5", the user with the identifier 5 will be removed from the database. If you create a new user at "users", Firebase creates the identifier for you and assigns all the information you pass for the user. The paths follow the [REST philosophy](https://en.wikipedia.org/wiki/Representational_state_transfer) where every entity (e.g. user, message, book, author) is associated with a URI, and HTTP methods are used to create, update, delete and get entities. In Firebase, the RESTful URI becomes a simple path, and the HTTP methods become Firebase's API.

### Exercises:

* Activate [Firebase's Realtime Database](https://www.robinwieruch.de/firebase-tutorial/) on your Firebase Dashboard
  * Set your Database Rules on your Firebase Project's Dashboard to `{ "rules": { ".read": true, ".write": true } }` to give everyone read and write access for now.
* Read more about [Firebase's realtime database setup for Web](https://firebase.google.com/docs/database/web/start)
* Confirm your [source code for the last section](http://bit.ly/2VpDkdW)

## User Management with Firebase

Now, use these references in your React components to create and get users from Firebase's realtime database. The best place to add user creation is the SignUpForm component, as it is the most natural place to save users after signing up via the Firebase authentication API. Add another API request to create a user when the sign up is successful.

{title="src/components/SignUp/index.js",lang="javascript"}
~~~~~~~
...

class SignUpFormBase extends Component {
  constructor(props) {
    super(props);

    this.state = { ...INITIAL_STATE };
  }

  onSubmit = event => {
    const { username, email, passwordOne } = this.state;

    this.props.firebase
      .doCreateUserWithEmailAndPassword(email, passwordOne)
# leanpub-start-insert
      .then(authUser => {
        // Create a user in your Firebase realtime database
        return this.props.firebase
          .user(authUser.user.uid)
          .set({
            username,
            email,
          });
      })
# leanpub-end-insert
      .then(() => {
        this.setState({ ...INITIAL_STATE });
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

There are two important things happening for a new sign up via the submit handler:

* (1) It creates a user in Firebase's internal authentication database that is only limited accessible.
* (2) If (1) was successful, it creates a user in Firebase's realtime database that is accessible.

To create a user in Firebase's realtime database, it uses the previously created reference from the Firebase class by providing the identifier (uid) of the user from Firebase's authentication database. Then the `set()` method can be used to provide data for this entity which is allocated for "users/uid". Finally, you can use the `username` as well to provide additional information about your user.

Note: It is fine to store user information in your own database. However, you should make sure not to store the password or any other sensitive data of the user on your own. Firebase already deals with the authentication, so there is no need to store the password in your database. Many steps are necessary to secure sensitive data (e.g. encryption), and it could be a security risk to perform it on your own.

After the second Firebase request that creates the user resolves successfully, the previous business logic takes place again: reset the local state and redirect to the home page. To verify the user creation is working, retrieve all the users from the realtime database in one of your other components. The admin page may be a good choice for it, because it can be used by admin users to manage the application-wide users later. First, make the admin page available via your Navigation component:

{title="src/components/Navigation/index.js",lang="javascript"}
~~~~~~~
...

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
# leanpub-start-insert
    <li>
      <Link to={ROUTES.ADMIN}>Admin</Link>
    </li>
# leanpub-end-insert
    <li>
      <SignOutButton />
    </li>
  </ul>
);

...
~~~~~~~

Next, the AdminPage component's `componentDidMount()` lifecycle method in *src/components/Admin/index.js* is the perfect place to fetch users from your Firebase realtime database API:

{title="src/components/Admin/index.js",lang="javascript"}
~~~~~~~
import React, { Component } from 'react';

# leanpub-start-insert
import { withFirebase } from '../Firebase';
# leanpub-end-insert

class AdminPage extends Component {
  constructor(props) {
    super(props);

# leanpub-start-insert
    this.state = {
      loading: false,
      users: {},
    };
# leanpub-end-insert
  }

# leanpub-start-insert
  componentDidMount() {
    this.setState({ loading: true });

    this.props.firebase.users().on('value', snapshot => {
      this.setState({
        users: snapshot.val(),
        loading: false,
      });
    });
  }
# leanpub-end-insert

  render() {
    return (
      <div>
        <h1>Admin</h1>
      </div>
    );
  }
}

# leanpub-start-insert
export default withFirebase(AdminPage);
# leanpub-end-insert
~~~~~~~

We are using the users reference from our Firebase class to attach a listener. The listener is called `on()`, which receives a type and a callback function. The `on()` method registers a continuous listener that triggers every time something has changed, the `once()` method registers a listener that would be called only once. In this scenario, we are interested to keep the latest list of users though.

Since the users are objects rather than lists when they are retrieved from the Firebase database, you have to restructure them as lists (arrays), which makes it easier to display them later:

{title="src/components/Admin/index.js",lang="javascript"}
~~~~~~~
...

class AdminPage extends Component {
  constructor(props) {
    super(props);

    this.state = {
      loading: false,
# leanpub-start-insert
      users: [],
# leanpub-end-insert
    };
  }

  componentDidMount() {
    this.setState({ loading: true });

    this.props.firebase.users().on('value', snapshot => {
# leanpub-start-insert
      const usersObject = snapshot.val();
# leanpub-end-insert

# leanpub-start-insert
      const usersList = Object.keys(usersObject).map(key => ({
        ...usersObject[key],
        uid: key,
      }));
# leanpub-end-insert

      this.setState({
# leanpub-start-insert
        users: usersList,
# leanpub-end-insert
        loading: false,
      });
    });
  }

  ...
}

export default withFirebase(AdminPage);
~~~~~~~

Remember to remove the listener to avoid memory leaks from using the same reference with the `off()` method:

{title="src/components/Admin/index.js",lang="javascript"}
~~~~~~~
...

class AdminPage extends Component {
  ...

# leanpub-start-insert
  componentWillUnmount() {
    this.props.firebase.users().off();
  }
# leanpub-end-insert

  ...
}

export default withFirebase(AdminPage);
~~~~~~~

Render your list of users in the AdminPage component or in a child component. In this case, we are using a child component:

{title="src/components/Admin/index.js",lang="javascript"}
~~~~~~~
...

class AdminPage extends Component {
  ...

  render() {
# leanpub-start-insert
    const { users, loading } = this.state;
# leanpub-end-insert

    return (
      <div>
        <h1>Admin</h1>

# leanpub-start-insert
        {loading && <div>Loading ...</div>}
# leanpub-end-insert

# leanpub-start-insert
        <UserList users={users} />
# leanpub-end-insert
      </div>
    );
  }
}

# leanpub-start-insert
const UserList = ({ users }) => (
  <ul>
    {users.map(user => (
      <li key={user.uid}>
        <span>
          <strong>ID:</strong> {user.uid}
        </span>
        <span>
          <strong>E-Mail:</strong> {user.email}
        </span>
        <span>
          <strong>Username:</strong> {user.username}
        </span>
      </li>
    ))}
  </ul>
);
# leanpub-end-insert

export default withFirebase(AdminPage);
~~~~~~~

You have gained full control of your users now. It is possible to create and retrieve users from your realtime database. You can decide whether this is a one-time call to the Firebase realtime database, or if you want to continuously listen for updates as well.

### Exercises:

* Read more about [how to read and write data to Firebase's realtime database](https://firebase.google.com/docs/database/web/read-and-write)
* Confirm your [source code for the last section](http://bit.ly/2VmRegY)