# How to continue your Journey ...

The [Learning Pyramid](https://www.google.com/search?q=learning+pyramid) shows the relation between retention rates and mental activities, proving usable data for both teaching and learning. It is one of the most effective ways to measure lessons I have encountered since I started teaching programming. This is how typical mental activities break down according to retention rates:

* 5% Lecture
* 10% Reading
* 20% Audiovisual
* 30% Demonstration
* 50% Discussion
* 75% Practice by Doing
* 90% Teach Others

At the start of this book I mentioned that very few people can master programming by reading a book, and throughout I have emphasized applying the lessons as the best way to retain them. As you can see, teaching others has the biggest return on investment. I had the same experience when I started writing about web development, answered questions on Quora, Reddit, Stack Overflow, and wrote books. Teaching others forces you to dive deeper into topics, so you learn about the nuances of programming. Think of friends, coworkers, or online peers who want to learn about your lessons learned from this book. Both mentor and student can grow from the experience.

What's next after you have finished this book? So far, the book has taught you how to use Firebase in React to build modern web applications. You've used all the fundamental features to power your application with authentication, authorization, and database interaction. However, there is plenty of room to explore the ecosystem, starting with this book's exercises and articles.

### Firebase's Cloud Firestore

Firebase's Cloud Firestore is the latest version of Firebase's Realtime Database. We used Realtime Database because it comes with plenty of documentation, has a more established community, and offers a lot of online tutorials. However, Firebase's Cloud Firestore might take over at some point, because it has a more intuitive data model, more features, faster queries, and a better scaling experience for larger applications. Since it only affects the Firebase database, everything else like the Firebase authentication mechanisms stays the same. You only need to [migrate to Cloud Firestore](https://www.robinwieruch.de/react-firestore-tutorial) to give it a shot.

### Firebase's Admin API

We used Firebase's authentication and realtime database in this book. Conversely, Firebase's admin API gives advanced control over users by changing email addresses, sending verification emails, or deleting them. As developer and admin, you are responsible for crafting a full-fledged admin dashboard to manage user entities without boundaries.

### Firebase's Storage API

Firebase enables you to store files as well. For instance, users who have signed in to your application with a social login already come with an avatar, because they often use an image of themselves on Google, Facebook or Twitter. With Firebase's storage API, you can allow them to upload personalized profile pictures. It's may be also helpful for users who sign up with a email/password combination, because they will not have a profile picture in the first place. Give your users an option to upload images and explore the Firebase Storage API.

### Firebase's Cloud Functions

Firebase's Cloud Functions are another advanced tool to outsource business logic from your application. For instance, cloud functions are often used to send recurring emails to users. When a user doesn't sign in for a while to your application, you could send this user a reminder email about the latest features they are missing out. Also you could use the cloud function to send emails about new content in your application. Exploring these email features with Firebase's Cloud Functions and [Sendgrid](https://sendgrid.com/) is a great learning experience in my opinion, because it let's you step away from only building frontend applications.

### Stripe and PayPal

You can also enable payments for your Firebase in React application to make it a full-fledged business application. [Stripe](https://www.robinwieruch.de/react-express-stripe-payment/) and [PayPal](https://www.robinwieruch.de/react-paypal-payment/) are two platforms that add monetization, for actions like charging users a one-time fee or adding recurring subscriptions. This is the next step in building a profitable online service.

### Keep Tinkering

The foremost recommendation I have is to continue tinkering with the React with Firebase application that you built in this book; as it's an ideal starter kit to realize your own ideas. As mentioned, you are free to substitute the technologies used under the hood, but maybe just focus on the features of your application. Since the user management is implemented for you, you can start to add your own features from there. If you want to substitute Firebase with your own database and authentication, my other book The Road to GraphQL shows how too add these same features with your own backend application.
