# Firebase Realtime Database (2): Advanced

Now we've worked with a list of data and single entities with the Firebase's realtime database to create an admin dashboard in the previous sections. In this section, I want to introduce a new entity to demonstrate a business-related feature for a Firebase in React application, a message entity that lets you create a chat application. We'll cover how to interact with Firebase's realtime database; specifically, how to structure data, work with lists of data, and how to create, update, and remove data. Also, you will see how ordering and pagination works with Firebase. In the end, it's up to you to decide whether your application should become a chat application with a message entity or a book application with a book entity in the database. The message entity is only there as example.

## Defining the API

Our Firebase class is the glue between our React application and the Firebase API. We instantiate it once, and then pass it to our React application via React's Context API. Then, we can define all APIs to connect both worlds in the Firebase class. We completed it earlier for the authentication API and the user management. Next, let's introduce the API for the new message entity.

{title="src/components/Firebase/firebase.js",lang="javascript"}
~~~~~~~
class Firebase {
  ...

  // *** User API ***

  user = uid => this.db.ref(`users/${uid}`);

  users = () => this.db.ref('users');

# leanpub-start-insert
  // *** Message API ***

  message = uid => this.db.ref(`messages/${uid}`);

  messages = () => this.db.ref('messages');
# leanpub-end-insert
}
~~~~~~~

Messages are readable and writeable on two API endpoints: messages and messages/:messageId. You will retrieve a list of messages and create a message with the `messages` reference, but you will edit and remove messages with the `messages/:messageId` reference.

If you want to be more specific, put more informative class methods for the message API in your Firebase class. For instance, there could be one class method for creating, updating, and removing a message. We will keep it general, however, and perform the specifics in the React components.

## How to fetch a List

The HomePage component might be the best place to add the chat feature with messages, which is only accessible by authenticated users due to authorization. Let's add a Message component that has access to the Firebase instance:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
# leanpub-start-insert
import React, { Component } from 'react';
# leanpub-end-insert
import { compose } from 'recompose';

import { withAuthorization, withEmailVerification } from '../Session';
# leanpub-start-insert
import { withFirebase } from '../Firebase';
# leanpub-end-insert

const HomePage = () => (
  <div>
    <h1>Home Page</h1>
    <p>The Home Page is accessible by every signed in user.</p>

# leanpub-start-insert
    <Messages />
# leanpub-end-insert
  </div>
);

# leanpub-start-insert
class MessagesBase extends Component {
  ...
}
# leanpub-end-insert

# leanpub-start-insert
const Messages = withFirebase(MessagesBase);
# leanpub-end-insert

export default compose(
  withEmailVerification,
  withAuthorization(condition),
)(HomePage);
~~~~~~~

The Messages component has a local state for a loading indicator and the list of messages. In the lifecycle methods of the component, you can initialize (and remove) listeners to get messages from the Firebase database in realtime. When messages change (create, update, remove), the callback function in the listener is triggered and Firebase provides a snapshot of the data.

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessagesBase extends Component {
# leanpub-start-insert
  constructor(props) {
    super(props);

    this.state = {
      loading: false,
      messages: [],
    };
  }

  componentDidMount() {
    this.setState({ loading: true });

    this.props.firebase.messages().on('value', snapshot => {
      // convert messages list from snapshot

      this.setState({ loading: false });
    });
  }

  componentWillUnmount() {
    this.props.firebase.messages().off();
  }

  render() {
    const { messages, loading } = this.state;

    return (
      <div>
        {loading && <div>Loading ...</div>}

        <MessageList messages={messages} />
      </div>
    );
  }
# leanpub-end-insert
}
~~~~~~~

The new MessageList and MessageItem components only render the message content:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
# leanpub-start-insert
const MessageList = ({ messages }) => (
  <ul>
    {messages.map(message => (
      <MessageItem key={message.uid} message={message} />
    ))}
  </ul>
);
# leanpub-end-insert

# leanpub-start-insert
const MessageItem = ({ message }) => (
  <li>
    <strong>{message.userId}</strong> {message.text}
  </li>
);
# leanpub-end-insert
~~~~~~~

If you run the application, the loading indicator disappears after a few seconds when the Firebase realtime database listener is called for the first time. Every other time the loading indicator isn't shown, because it is only `true` when the component mounts and the first message fetching starts.

It could be that there are no messages yet, which is the case for this application since we didn't use the message API to create a message yet. We're only showing the messages for now. To show conditional feedback to users, we need to know if the list of messages is empty (see constructor), if the message API didn't return any messages and the local state is changed from an empty array to null:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessagesBase extends Component {
  constructor(props) {
    super(props);

    this.state = {
      loading: false,
      messages: [],
    };
  }

  componentDidMount() {
    this.setState({ loading: true });

    this.props.firebase.messages().on('value', snapshot => {
# leanpub-start-insert
      const messageObject = snapshot.val();
# leanpub-end-insert

# leanpub-start-insert
      if (messageObject) {
        // convert messages list from snapshot

        this.setState({ loading: false });
      } else {
        this.setState({ messages: null, loading: false });
      }
# leanpub-end-insert
    });
  }

  ...

  render() {
    const { messages, loading } = this.state;

    return (
      <div>
        {loading && <div>Loading ...</div>}

# leanpub-start-insert
        {messages ? (
# leanpub-end-insert
          <MessageList messages={messages} />
# leanpub-start-insert
        ) : (
          <div>There are no messages ...</div>
        )}
# leanpub-end-insert
      </div>
    );
  }
}
~~~~~~~

Lastly, you need to convert the messages from the snapshot object to a list of items. Since Firebase comes with its own internal representation of data, you need to transform the data as before for the list of users on the admin page:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessagesBase extends Component {
  ...

  componentDidMount() {
    this.setState({ loading: true });

    this.props.firebase.messages().on('value', snapshot => {
      const messageObject = snapshot.val();

      if (messageObject) {
# leanpub-start-insert
        const messageList = Object.keys(messageObject).map(key => ({
          ...messageObject[key],
          uid: key,
        }));
# leanpub-end-insert

        this.setState({
# leanpub-start-insert
          messages: messageList,
# leanpub-end-insert
          loading: false,
        });
      } else {
        this.setState({ messages: null, loading: false });
      }
    });
  }

  ...
}
~~~~~~~

Since you have no messages, nothing shows up. Creating chat messages is our next task.

## Creating an Item in a List

We were able to get all messages from the Firebase realtime database. It's even updated for us using the Firebase listener on a reference with the `on` and not `once` method. Next, let's implement a React form that lets us create a message entity in the Firebase realtime database:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessagesBase extends Component {
  ...

  render() {
# leanpub-start-insert
    const { text, messages, loading } = this.state;
# leanpub-end-insert

    return (
      <div>
        {loading && <div>Loading ...</div>}

        {messages ? (
          <MessageList messages={messages} />
        ) : (
          <div>There are no messages ...</div>
        )}

# leanpub-start-insert
        <form onSubmit={this.onCreateMessage}>
          <input
            type="text"
            value={text}
            onChange={this.onChangeText}
          />
          <button type="submit">Send</button>
        </form>
# leanpub-end-insert
      </div>
    );
  }
}
~~~~~~~

Next, add the new initial state for the component to keep track of the text property for a new message and its two new class methods to update the text in an input field element and create the actual message with Firebase:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessagesBase extends Component {
  constructor(props) {
    super(props);

    this.state = {
# leanpub-start-insert
      text: '',
# leanpub-end-insert
      loading: false,
      messages: [],
    };
  }

  ...

# leanpub-start-insert
  onChangeText = event => {
    this.setState({ text: event.target.value });
  };
# leanpub-end-insert

# leanpub-start-insert
  onCreateMessage = event => {
    this.props.firebase.messages().push({
      text: this.state.text,
    });

    this.setState({ text: '' });

    event.preventDefault();
  };
# leanpub-end-insert

  ...
}
~~~~~~~

We can use the push method on a Firebase reference to create a new entity in this list of entities, though we don't want to create a message just yet. One piece is missing for associating messages to users, which needs to be implemented before we create messages.

## Relationships

If you look closer at the MessageItem component, you can see that a message not only has a `text`, but also a `userId` that can be used to associate the message to a user. Let's use the authenticated user from our React Context to store the user identifier in a new message. First, add the Consumer component and add the identifier for the authenticated user in the class method call that creates the message:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
...

import {
# leanpub-start-insert
  AuthUserContext,
# leanpub-end-insert
  withAuthorization,
  withEmailVerification,
} from '../Session';

...

class MessagesBase extends Component {
  ...

  render() {
    const { text, messages, loading } = this.state;

    return (
# leanpub-start-insert
      <AuthUserContext.Consumer>
        {authUser => (
# leanpub-end-insert
          <div>
            {loading && <div>Loading ...</div>}

            {messages ? (
              <MessageList messages={messages} />
            ) : (
              <div>There are no messages ...</div>
            )}

# leanpub-start-insert
            <form onSubmit={event => this.onCreateMessage(event, authUser)}>
# leanpub-end-insert
              <input
                type="text"
                value={text}
                onChange={this.onChangeText}
              />
              <button type="submit">Send</button>
            </form>
          </div>
# leanpub-start-insert
        )}
      </AuthUserContext.Consumer>
# leanpub-end-insert
    );
  }
}
~~~~~~~

Next, use the authenticated user to associate the user identifier to the message. It makes sense to use the authenticated user, because this is the person authorized to write messages:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessagesBase extends Component {
  ...

# leanpub-start-insert
  onCreateMessage = (event, authUser) => {
# leanpub-end-insert
    this.props.firebase.messages().push({
      text: this.state.text,
# leanpub-start-insert
      userId: authUser.uid,
# leanpub-end-insert
    });

    this.setState({ text: '' });

    event.preventDefault();
  };

  ...
}
~~~~~~~

Now go ahead and create a message. Since we only can access this page as an authenticated user due to authorization, we know that each message that is written here will be associated to a user identifier. After you have created a message, the realtime feature of the Firebase database makes sure that the message will show up in our rendered list.

So far, we have chosen to keep the footprint of a user entity within a message as little as possible. There is only an user identifier which associates the message to a user. Generally speaking it's good to structure data in your database this way, because it avoids plenty of pitfalls. For instance, let's imagine you would associate the whole user entity to a message and not only the identifier. Then every time a user entity changes in the database, you would have to change the message entity with the user as well. That's a common problem when not following the principal of the single source of truth when designing your database models. In our case, we are associating entities with each other only by their identifiers instead, whereas each entity in the database is the single source of truth without any duplications.

Another thing we decided earlier is giving the messages their dedicated API reference with `messages`. In another scenario, it could have been `users/:userId/messages` to associate users directly with the message via the reference. But doing it this way, we would have to fetch messages from multiple API endpoints in the end to show a nice chatroom as we do it right now.

## Removing an Item in a List

We are reading a list of messages and created our first message. What about the other two missing functionalities to remove and edit a message. Let's continue with removing a message. Pass through a new class method that will remove a message eventually:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessagesBase extends Component {
  ...

# leanpub-start-insert
  onRemoveMessage = () => {
    ...
  };
# leanpub-end-insert

  render() {
    const { text, messages, loading } = this.state;

    return (
      <AuthUserContext.Consumer>
        {authUser => (
          <div>
            ...

            {messages ? (
              <MessageList
                messages={messages}
# leanpub-start-insert
                onRemoveMessage={this.onRemoveMessage}
# leanpub-end-insert
              />
            ) : (
              <div>There are no messages ...</div>
            )}

            ...
          </div>
        )}
      </AuthUserContext.Consumer>
    );
  }
}
~~~~~~~

The MessageList component in between just pass the function through to the MessageItem component:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
# leanpub-start-insert
const MessageList = ({ messages, onRemoveMessage }) => (
# leanpub-end-insert
  <ul>
    {messages.map(message => (
      <MessageItem
        key={message.uid}
        message={message}
# leanpub-start-insert
        onRemoveMessage={onRemoveMessage}
# leanpub-end-insert
      />
    ))}
  </ul>
);
~~~~~~~

Finally it can be used in the MessageItem component. When clicking the button, we will pass the message identifier to the function. Then in our parent component that has access to Firebase we can remove the message associated with the identifier.

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
# leanpub-start-insert
const MessageItem = ({ message, onRemoveMessage }) => (
# leanpub-end-insert
  <li>
    <strong>{message.userId}</strong> {message.text}
# leanpub-start-insert
    <button
      type="button"
      onClick={() => onRemoveMessage(message.uid)}
    >
      Delete
    </button>
# leanpub-end-insert
  </li>
);
~~~~~~~

Last, implement the class method that deletes the item from the list. Since we have access to the identifier of the message, we can use the reference of a single message to remove it.

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessagesBase extends Component {
  ...

# leanpub-start-insert
  onRemoveMessage = uid => {
    this.props.firebase.message(uid).remove();
  };
# leanpub-end-insert

  ...
}
~~~~~~~

Deleting a message works, and you can also make Firebase instance available to the MessageItem component and delete the message there right away. The real-time connection to the Firebase database in the Messages component would still be called to remove the message, which keeps the displayed messages in sync. However, aggregating all the business logic in one place, in this case the Messages component, makes sense for a better maintainability and predictability of the application. Only a few components have the more complex logic whereas the other components are just there to render the content.

## Editing an Item in a List

It's abnormal to update a message in a chat application, but we'll implement this feature anyway. Eventually, we'll give other users feedback that a message was edited. That way, all statements made in the chat keep their integrity. Again, implement the class method first, which we will fill with details later, and pass it down to the MessageList component:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessagesBase extends Component {
  ...

# leanpub-start-insert
  onEditMessage = () => {
    ...
  };
# leanpub-end-insert

  render() {
    const { text, messages, loading } = this.state;

    return (
      <AuthUserContext.Consumer>
        {authUser => (
          <div>
            ...

            {messages ? (
              <MessageList
                messages={messages}
# leanpub-start-insert
                onEditMessage={this.onEditMessage}
# leanpub-end-insert
                onRemoveMessage={this.onRemoveMessage}
              />
            ) : (
              <div>There are no messages ...</div>
            )}

            ...
          </div>
        )}
      </AuthUserContext.Consumer>
    );
  }
}
~~~~~~~

Again, the MessageList component just passes it through to the MessageItem component:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
const MessageList = ({
  messages,
# leanpub-start-insert
  onEditMessage,
# leanpub-end-insert
  onRemoveMessage,
}) => (
  <ul>
    {messages.map(message => (
      <MessageItem
        key={message.uid}
        message={message}
# leanpub-start-insert
        onEditMessage={onEditMessage}
# leanpub-end-insert
        onRemoveMessage={onRemoveMessage}
      />
    ))}
  </ul>
);
~~~~~~~

Editing a message involves a few more rendered elements, business logic, and state in the MessageItem component. That's why we refactor it to a class component:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
# leanpub-start-insert
class MessageItem extends Component {
# leanpub-end-insert
  ...
# leanpub-start-insert
}
# leanpub-end-insert
~~~~~~~

Next, we'll keep track of the mode of the component, which tells us if we're showing the text of a message or editing it. Also, if we are editing a message, we need to track the value of the input field element. As initial state, it receives the text of the message entity which makes sense if we only want to edit a typo in the message:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessageItem extends Component {
# leanpub-start-insert
   constructor(props) {
    super(props);

    this.state = {
      editMode: false,
      editText: this.props.message.text,
    };
  }
# leanpub-end-insert

  ...
}
~~~~~~~

Now, let's implement three class methods, the first of which is a class method for toggling the mode from edit to preview and back. If this mode is toggled, we always fill in the text of the message as a value for the input field element to improve the user experience when the mode is toggled:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessageItem extends Component {
  ...

# leanpub-start-insert
  onToggleEditMode = () => {
    this.setState(state => ({
      editMode: !state.editMode,
      editText: this.props.message.text,
    }));
  };
# leanpub-end-insert

  ...
}
~~~~~~~

Second, a class method for updating the value in the input field:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessageItem extends Component {
  ...

# leanpub-start-insert
  onChangeEditText = event => {
    this.setState({ editText: event.target.value });
  };
# leanpub-end-insert

  ...
}
~~~~~~~

And third, a class method to submit the final value to the parent component to edit the message:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessageItem extends Component {
  ...

# leanpub-start-insert
  onSaveEditText = () => {
    this.props.onEditMessage(this.props.message, this.state.editText);

    this.setState({ editMode: false });
  };
# leanpub-end-insert

  ...
}
~~~~~~~

Later, we will see why we send the message with the edited text. Next, let's implement the render method of the MessageItem component. Make sure that the button to delete a message is not displayed in edit mode:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessageItem extends Component {
  ...

# leanpub-start-insert
  render() {
    const { message, onRemoveMessage } = this.props;
    const { editMode, editText } = this.state;
# leanpub-end-insert

# leanpub-start-insert
    return (
# leanpub-end-insert
      <li>
        <span>
          <strong>{message.userId}</strong> {message.text}
        </span>

# leanpub-start-insert
        {!editMode && (
# leanpub-end-insert
          <button
            type="button"
            onClick={() => onRemoveMessage(message.uid)}
          >
            Delete
          </button>
# leanpub-start-insert
        )}
# leanpub-end-insert
      </li>
# leanpub-start-insert
    );
  }
# leanpub-end-insert
}
~~~~~~~

Next add "Edit" and "Reset" buttons to toggle between preview and edit mode. Depending on the edit mode, the correct button is displayed, and a "Save" button is shown in edit mode to save the edited text:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessageItem extends Component {
  ...

  render() {
    const { message, onRemoveMessage } = this.props;
    const { editMode, editText } = this.state;

    return (
      <li>
        <span>
          <strong>{message.userId}</strong> {message.text}
        </span>

# leanpub-start-insert
        {editMode ? (
          <span>
            <button onClick={this.onSaveEditText}>Save</button>
            <button onClick={this.onToggleEditMode}>Reset</button>
          </span>
        ) : (
          <button onClick={this.onToggleEditMode}>Edit</button>
        )}
# leanpub-end-insert

        ...
      </li>
    );
  }
}
~~~~~~~

Last, we need the input field element to edit the text. It is only displayed in edit mode. If we are not in edit mode, the actual text of the message is shown:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessageItem extends Component {
  ...

  render() {
    const { message, onRemoveMessage } = this.props;
    const { editMode, editText } = this.state;

    return (
      <li>
# leanpub-start-insert
        {editMode ? (
          <input
            type="text"
            value={editText}
            onChange={this.onChangeEditText}
          />
        ) : (
# leanpub-end-insert
          <span>
            <strong>{message.userId}</strong> {message.text}
          </span>
# leanpub-start-insert
        )}
# leanpub-end-insert

        ...
      </li>
    );
  }
}
~~~~~~~

Now we can edit the text in edit mode, and we can also reset the whole thing using a button. If we save the edited text, the text and the message will be sent through the MessageList component to the Messages component, where the message can be identified by id to be edited with the text property. Using the spread operator, all other properties of the message entity are kept as before:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessagesBase extends Component {
  ...

# leanpub-start-insert
  onEditMessage = (message, text) => {
    const { uid, ...messageSnapshot } = message;

    this.props.firebase.message(message.uid).set({
      ...messageSnapshot,
      text,
    });
  };
# leanpub-end-insert

  ...
}
~~~~~~~

If we set only the new text for the message, all other properties (e.g. userId) would be lost. Also we can add `createdAt` and `editedAt` dates. The second date gives users feedback that someone changed a chat message:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessagesBase extends Component {
  ...

  onCreateMessage = (event, authUser) => {
    this.props.firebase.messages().push({
      text: this.state.text,
      userId: authUser.uid,
# leanpub-start-insert
      createdAt: this.props.firebase.serverValue.TIMESTAMP,
# leanpub-end-insert
    });

    this.setState({ text: '' });

    event.preventDefault();
  };

  onEditMessage = (message, text) => {
    const { uid, ...messageSnapshot } = message;

    this.props.firebase.message(message.uid).set({
      ...messageSnapshot,
      text,
# leanpub-start-insert
      editedAt: this.props.firebase.serverValue.TIMESTAMP,
# leanpub-end-insert
    });
  };

  ...
}
~~~~~~~

When using Firebase, it's best not to choose the date yourself, but let Firebase choose it depending on their internal mechanics. The server value constants from Firebase can be made available in the Firebase class:

{title="src/components/Firebase/firebase.js",lang="javascript"}
~~~~~~~
class Firebase {
  constructor() {
    app.initializeApp(config);

    /* Helper */

# leanpub-start-insert
    this.serverValue = app.database.ServerValue;
# leanpub-end-insert
    this.emailAuthProvider = app.auth.EmailAuthProvider;

    ...
  }

  ...
}
~~~~~~~

In the MessageItem component, give users feedback that shows when a message was edited:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessageItem extends Component {
  ...

  render() {
    const { message, onRemoveMessage } = this.props;
    const { editMode, editText } = this.state;

    return (
      <li>
        {editMode ? ( ... ) : (
          <span>
            <strong>{message.userId}</strong> {message.text}
# leanpub-start-insert
            {message.editedAt && <span>(Edited)</span>}
# leanpub-end-insert
          </span>
        )}

        ...
      </li>
    );
  }
}
~~~~~~~

As before, we could have used Firebase directly in the MessageItem component. It's also good to keep the MessageItem component encapsulated with its own business logic. Only the message itself and the other functions to alter the message are passed from above to the component, and only the Messages component speaks to the outside world (e.g. Firebase).

You have implemented the popular CRUD operations: create, read, update, delete, which is everything you need to manage the new message entity in your Firebase database. Also, you have learned how to assign dates to your Firebase entities, and how to listen for real-time updates when a message has been added, edited or removed.

## Securing User Interactions

So far, every user can edit and remove messages. Let's change this by giving only owner of messages the power to perform these operations within the UI. Therefore, we need the authenticated user in the MessageItem component. Since we already have the authenticated user in the Messages component, let's pass it down to the MessageList component:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessagesBase extends Component {

  ...

  render() {
    const { text, messages, loading } = this.state;
     return (
      <AuthUserContext.Consumer>
        {authUser => (
          <div>
            ...

            {messages ? (
              <MessageList
# leanpub-start-insert
                authUser={authUser}
# leanpub-end-insert
                messages={messages}
                onEditMessage={this.onEditMessage}
                onRemoveMessage={this.onRemoveMessage}
              />
            ) : (
              <div>There are no messages ...</div>
            )}

            ...
          </div>
        )}
      </AuthUserContext.Consumer>
    );
  }
}
~~~~~~~

And from there down to the MessageItem component:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
const MessageList = ({
# leanpub-start-insert
  authUser,
# leanpub-end-insert
  messages,
  onEditMessage,
  onRemoveMessage,
}) => (
  <ul>
    {messages.map(message => (
      <MessageItem
# leanpub-start-insert
        authUser={authUser}
# leanpub-end-insert
        key={message.uid}
        message={message}
        onEditMessage={onEditMessage}
        onRemoveMessage={onRemoveMessage}
      />
    ))}
  </ul>
);
~~~~~~~

Now in your MessageItem component, you can secure the buttons to edit and remove messages by comparing the message's `userId` with the authenticated user's id:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessageItem extends Component {
  ...

  render() {
# leanpub-start-insert
    const { authUser, message, onRemoveMessage } = this.props;
# leanpub-end-insert
    const { editMode, editText } = this.state;

     return (
      <li>
        ...

# leanpub-start-insert
        {authUser.uid === message.userId && (
          <span>
# leanpub-end-insert
            {editMode ? (
              <span>
                <button onClick={this.onSaveEditText}>Save</button>
                <button onClick={this.onToggleEditMode}>Reset</button>
              </span>
            ) : (
              <button onClick={this.onToggleEditMode}>Edit</button>
            )}
             {!editMode && (
              <button
                type="button"
                onClick={() => onRemoveMessage(message.uid)}
              >
                Delete
              </button>
            )}
# leanpub-start-insert
          </span>
        )}
# leanpub-end-insert
      </li>
    );
  }
}
~~~~~~~

That's it for only enabling users who are owners of a message to edit and delete the message in the UI. You will see later how you can secure the Firebase API endpoint as well to not allow users to edit/delete entities; otherwise it would still be possible to alter the source code in the browser to show the buttons for deleting and editing messages even though the user has no permission to perform it.

## Ordering

Currently, messages are retrieved in no specific order from the Firebase realtime database, which means they would be in the order of their creation. This is appropriate for a chat application, but let's make this behavior more explicit by ordering them by the `createdAt` date property since we have introduced this earlier:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessagesBase extends Component {
  ...

  componentDidMount() {
    this.setState({ loading: true });

    this.props.firebase
      .messages()
# leanpub-start-insert
      .orderByChild('createdAt')
# leanpub-end-insert
      .on('value', snapshot => {
        const messageObject = snapshot.val();

        ...
      });
  }

  ...
}
~~~~~~~

Pass the property that should be used to retrieved the list as ordered list from the Firebase realtime database. By default Firebase is ordering the items in ascending direction. To reverse the order, add a `reverse()` after transforming the list of messages from an object to an array.

You might see a warning about indexing data in Firebase's realtime database, because we're fetching data in a specific order, and Firebase uses the property `createdAt` to fetch it more efficiently. You can index messages using the `createdAt` property to give Firebase a performance boost when fetching the messages with this ordering. Head over to your project's Firebase dashboard, open the "Database" tab, and click the "Rules" tab. You can add the indexing of the data there:

{title="Firebase Dashboard -> Databasse Tab -> Rules Tab",lang="json"}
~~~~~~~
{
  "rules": {
    "messages": {
      ".indexOn": ["createdAt"]
    }
  }
}
~~~~~~~

The warning should no longer appear, and Firebase became faster at retrieving messages by creation date. Every time you see the warning popping up, head over to your rules and index your Firebase entities. It makes your Firebase database operations faster.

## Pagination

Next is the ordering feature, and we will paginate the list from the Firebase realtime database as well. You can pass the Firebase API a limit method with an integer to specify how many items you are interested in:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessagesBase extends Component {
  ...

  componentDidMount() {
    this.setState({ loading: true });

    this.props.firebase
      .messages()
      .orderByChild('createdAt')
# leanpub-start-insert
      .limitToLast(5)
# leanpub-end-insert
      .on('value', snapshot => {
        ...
      });
  }

  ...
}
~~~~~~~

Limiting the items is half the task for enabling pagination for our chat application. We also need to move the limit to the local state of the component to adjust it later with user interactions to fetch more than five items:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessagesBase extends Component {
  constructor(props) {
    super(props);

    this.state = {
      text: '',
      loading: false,
      messages: [],
# leanpub-start-insert
      limit: 5,
# leanpub-end-insert
    };
  }

  componentDidMount() {
    this.setState({ loading: true });

    this.props.firebase
      .messages()
      .orderByChild('createdAt')
# leanpub-start-insert
      .limitToLast(this.state.limit)
# leanpub-end-insert
      .on('value', snapshot => {
        ...
      });
  }

  ...

}
~~~~~~~

Move this functionality outside of the lifecycle method to make it reusable for other user interaction, and to use it outside of when the component mounts:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessagesBase extends Component {
  ...

  componentDidMount() {
# leanpub-start-insert
    this.onListenForMessages();
# leanpub-end-insert
  }

# leanpub-start-insert
  onListenForMessages() {
# leanpub-end-insert
    this.setState({ loading: true });

    this.props.firebase
      .messages()
      .orderByChild('createdAt')
      .limitToLast(this.state.limit)
      .on('value', snapshot => {
        ...
      });
# leanpub-start-insert
  }
# leanpub-end-insert

  ...
}
~~~~~~~

Next, let's add a button to indicate that we are interested in more than five items:

{title="src/components/Home/index.js",lang="javascript"}
~~~~~~~
class MessagesBase extends Component {
  ...

# leanpub-start-insert
  onNextPage = () => {
    this.setState(
      state => ({ limit: state.limit + 5 }),
      this.onListenForMessages,
    );
  };
# leanpub-end-insert

  render() {
    const { text, messages, loading } = this.state;

    return (
      <AuthUserContext.Consumer>
        {authUser => (
          <div>
# leanpub-start-insert
            {!loading && messages && (
              <button type="button" onClick={this.onNextPage}>
                More
              </button>
            )}
# leanpub-end-insert

            ...
          </div>
        )}
      </AuthUserContext.Consumer>
    );
  }
}
~~~~~~~

The button uses a new class method that increases the limit by five again. Afterward, using the second argument of React's setState method, we can renew the Firebase listener with the new limit from the local state. We know that the second function in this React-specific method runs when the asynchronous state update happens, at which point the listener can use the correct limit from the local state.

Everything you have learned in this chapter should make you proficient with structured and list data in Firebase's realtime database. You have learned how to get, create, update and remove entities in a Firebase realtime database, and how to keep a synchronized connection to Firebase and always show the latest entities. Finally, we went through the pagination and ordering features offered by Firebase.

### Exercises:

* Read more about [structuring data in Firebase](https://firebase.google.com/docs/database/web/structure-data)
* Read more about [working with lists of data in Firebase](https://firebase.google.com/docs/database/web/lists-of-data)
* Read more about [indexing your Firebase data](https://firebase.google.com/docs/database/security/indexing-data)
* Confirm your [source code for the last section](http://bit.ly/2Vng1Sc)
* Refactoring:
  * Move all user related components on the AdminPage to their own folder/file module.
  * Move all message related components on the HomePage to their own folder/file module.
  * Confirm your [source code for this refactoring](http://bit.ly/2VplDLI)
* Prevent fetching more items with the "More" button when there are no more items available.
