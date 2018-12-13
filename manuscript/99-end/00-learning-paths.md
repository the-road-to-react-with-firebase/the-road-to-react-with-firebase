# Learning Paths

The last chapters of this book are to inspire you to apply what you've learned. So far, the book has taught you how to use Firebase in React applications to build sophisticated web applications. You have implemented all the fundamental features that power modern applications nowadays. Firebase offers you everything you need to implement authentication, authorization and database interactions. However, there is plenty of room to explore the ecosystem. If you haven't completed all the exercises from the book or read all the articles, you should start with these. Otherwise, let's see what else you can do to advance your skills.

## Firebase's Cloud Firestore

Firebase's Cloud Firestore is the new version of Firebase's Realtime Database. You have used the latter in this book, because it comes with plenty of documentation, is used for a longer time in the community, and offers lots of other tutorials online. However, at some point Firebase's Cloud Firestore may take over, because it comes with a couple of benefits such as a more intuitive data model, more features, faster queries, and a better scaling experience for larger applications. Since it only affects the Firebase database, everything else like the Firebase authentication mechanisms we have built stays the same. You only need to migrate over to Cloud Firestore if you want to give it a shot. If you got the complete course for this book, you will find one source code project with a tutorial on how to migrate the application from this book over to the new Cloud Firestore.

## Firebase's Admin API

In this book, you have used Firebase's authentication and realtime database. What about Firebase's admin API? It gives you advanced control over your users by changing their email addresses, sending them verification emails, or deleting them. You as developer and admin are in charge to implement a full-fledged admin dashboard to manage all your user entities without any boundaries.

## Firebase's Storage API

What about enabling Firebase's Storage? Users who have signed in to your application already come with an avatar, because they often use an image of themselves on Google, Facebook or Twitter. But what about people who have signed up with a email and password combination? You could offer them the possibility to upload their own profile picture by enabling Firebase's storage API.

## Firebase's Cloud Functions

Firebase's Cloud Functions are another advanced tool to outsource business logic from your application. For instance, cloud functions are often used to send recurring emails to users. When a user doesn't sign in for a while to your application, you could send this user a reminder email about the latest features they are missing out. Also you could use the cloud function to send emails about new content in your application. Exploring these email features with Firebase's Cloud Functions and [Sendgrid](https://sendgrid.com/) is a great learning experience in my opinion, because it let's you step away from only building frontend applications.

## Stripe and PayPal

What about enabling payments for your Firebase in React application to make it a full-fledged business application? [Stripe](https://www.robinwieruch.de/react-express-stripe-payment/) and [PayPal](https://www.robinwieruch.de/react-paypal-payment/) are two integrations you could make use of to charge your application's users for a one-time fee or recurring subscription. It is only the next step it needs to take to build a profitable online service yourself.

## Keep Tinkering

My ultimate recommendation would be to continue with the React with Firebase application that you built in this book, as it's an ideal starter kit to realize your own ideas. As mentioned, you are free to substitute the technologies used under the hood, but it would be great to focus on the features for your application. Since the user management is implemented for you, you can start to add your own features. If you want to substitute Firebase with your own database and authentication eventually, checkout my other book The Road to GraphQL where you will essentially implement the same features but with your own backend application.

