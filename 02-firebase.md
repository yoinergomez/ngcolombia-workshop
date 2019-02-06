# Fire-what!?

For this app, we'll use Firebase as our backend. If you don't know what Firebase is, it's an entire platform provided by Google that can handle almost every aspect of your app's infrastructure.

For example, they provide a database, file storage, push notifications, hosting, among other things.

For this workshop, we'll use Firestore, their new NoSQL database that has built-in real-time and offline features.

This gives us two things out of the box, the content you change in the database gets updated real-time in every app that's connected, and it has offline sync without you doing extra work.

You can learn more about Firebase at [https://firebase.google.com/](https://firebase.google.com/) and use it for future projects.

Since we want to focus on building the actual app on this workshop we have already created a Firebase app for you to connect to, and we'll share the credentials with you later on this step.

This we'll help you save some time that can be used to finish the app :)

## Installing Firebase

The first thing we need to do to start using Firebase is to install it in our project since Stackblitz handles that part for us, we only need to tell it the packages names.

Go to the "**DEPENDENCIES**" section on your left panel and in the input box that says "_enter package name_" type **firebase**, then hit enter.

> NOTE: The package name needs to be written in lowercase, if you use uppercase it will show you an error saying it can't find it.

![Install Firebase](img/install-firebase.png)

That gives you access to the entire Firebase SDK for web development. Now, we can take things one step further and install `@angular/fire` and `@firebase/util`.

`@angular/fire` is a library created by people from both the Firebase and the Angular teams, and it gives you better integration with Firebase when you're working on Angular projects.

`@firebase/util` is a utility library from Firebase that is needed starting from version 5.8.0.

You can install it by typing `@angular/fire` in the input box that says "_enter package name_" the same way you installed Firebase, then do the same for `@firebase/util`.

Once both are installed you can go into `package.json` and see the package names there, if you don't, feel free to ask one of the mentors for help :)

## Connect your app with Firebase

Now that Firebase is installed we need to let our Angular app know how it will communicate with it. For that, we'll go to the `app.module.ts` file and initialize Firebase.

It's a 2 step process, first, we need to import the packages we're going to use, so go ahead and add this to your imports:

```js
import { AngularFireModule } from '@angular/fire';
import { AngularFirestoreModule } from '@angular/fire/firestore';
```

`@angular/fire` is modular, which means we only import what we'll use, in this case, we need the base functionality and Firestore.

And second, we'll add both `AngularFireModule` and `AngularFirestoreModule` to our imports array, right now, it looks like this:

```js
@NgModule({
  imports: [BrowserModule, ReactiveFormsModule],
  declarations: [AppComponent, CardComponent, AddShowComponent],
  bootstrap: [AppComponent],
})
```

We want it to look like this:

```js
@NgModule({
  imports: [
    BrowserModule,
    AngularFireModule.initializeApp({
      apiKey: 'AIzaSyBIM6XVuaqeBDtyyB9Ef1U5oUv9Cue9BK8',
      authDomain: 'first-firebase-angular.firebaseapp.com',
      databaseURL: 'https://first-firebase-angular.firebaseio.com',
      projectId: 'first-firebase-angular',
      storageBucket: 'first-firebase-angular.appspot.com',
      messagingSenderId: '306103315077',
    }),
    AngularFirestoreModule,
  ],
  declarations: [AppComponent, CardComponent, AddShowComponent],
  bootstrap: [AppComponent]
})
```

We're adding `AngularFireModule` and calling the `.initializeApp()` method to connect our app to Firebase. The `.initializeApp()` method takes an object as a parameter, that object is our credential object, which lets Angular know **which** Firebase app it needs to connect to.

You'll copy those same credentials, the idea is we all share the same app and work faster that way.

We're also adding the `AngularFirestoreModule` to let Angular know we'll be using the Firestore database in our project.

In the end, the entire file should look like this:

```js
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppComponent } from './app.component';
import { AngularFireModule } from '@angular/fire';
import { AngularFirestoreModule } from '@angular/fire/firestore';

@NgModule({
  imports: [
    BrowserModule,
    AngularFireModule.initializeApp({
      apiKey: 'AIzaSyBIM6XVuaqeBDtyyB9Ef1U5oUv9Cue9BK8',
      authDomain: 'first-firebase-angular.firebaseapp.com',
      databaseURL: 'https://first-firebase-angular.firebaseio.com',
      projectId: 'first-firebase-angular',
      storageBucket: 'first-firebase-angular.appspot.com',
      messagingSenderId: '306103315077',
    }),
    AngularFirestoreModule,
  ],
  declarations: [AppComponent],
  bootstrap: [AppComponent],
  providers: [],
})
export class AppModule {}
```

## Next Steps

Once you're ready, move to the next step where we'll start building the **C** part of CRUD, we'll add the functionality to create a TV show and save it to the database
