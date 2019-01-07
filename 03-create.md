# Adding objects to our database

In the previous lesson we connected our app to Firebase. Now it's time to start writing content into the database, for that we need a couple of things:

- A form that lets us write the content.
- A function that takes that content and sends it to Firestore.
- A way of knowing the content was saved successfully.

With that in mind let's begin with number 1, we want to create a simple form that allows us to get 2 properties, the show's name, and the show's description.

## Creating a new component

One of the coolest things I love about Angular is the ability to make everything a component, that means that if we create our form as a component then we can plug it into any part of the app in the future without needing to copy/paste or re-do the code.

With that in mind lets take advantage of Stackblitz integration with Angular and generate a new component for our app, go ahead and right-click into the `app/` folder, choose the **Angular Generator** option and select **Component**.

![Add a component to the project](img/add-component.png)

It will show you an input box for you to add the name, call it **add-show**. Stackblitz will generate all the files for you and import your new component into the `app.module.ts` file automagically, how cool is that!

Go into the `add-show` folder and you'll notice that Stackblitz generated 3 files for you.

![Add show component](img/add-show-component.png)

- The `add-show.component.css` that handles our styles.
- The `add-show.component.html` that will have our view.
- And the `add-show.component.ts` that will have the component's logic.

While we're generating the files, open the **Angular Generator** again (right-click the app/ folder) and select **Service** and name it **firebase**.

It's a service where we'll store all the interactions with the database, we'll cover more about it when we start using it.

We'll start setting up the component's logic, that way when it's time to implement wiew or styles it will just work &trade;

So, go into `add-show.component.ts` and the first thing we'll do is to add all the imports we need at the top of the file:

```js
import { Component, OnInit, Output, EventEmitter } from '@angular/core';
import { FormGroup, Validators, FormBuilder } from '@angular/forms';
import { FirebaseService } from '../firebase.service';
```

We're importing:

- `{ Component, OnInit, Output, EventEmitter }` to handle Angular component interactions, we'll go into more details later but the `add-show` component needs to have a way to talk with the app component.
- `{ FormGroup, Validators, FormBuilder }` are for working with Reactive forms in Angular.
- `{ FirebaseService }`is the Firebase service we just created, it's for handling the interactions with the database.

Before the app starts throwing errors, we need to add a module to the `app.module.ts` file, open it, and where you find the `FormsModule` replace it by the `ReactiveFormsModule`.

It should look like this:

```js
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { ReactiveFormsModule } from '@angular/forms';

import { AppComponent } from './app.component';

import { AngularFireModule } from '@angular/fire';
import { AngularFirestoreModule } from '@angular/fire/firestore';
import { AddShowComponent } from './add-show/add-show.component';
import { FirebaseService } from './firebase.service';

@NgModule({
  imports: [
    BrowserModule,
    ReactiveFormsModule,
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
  declarations: [AppComponent, AddShowComponent],
  bootstrap: [AppComponent],
  providers: [FirebaseService],
})
export class AppModule {}
```

Alright, now that everything is imported we'll start by adding the Class variables we need for this component, right before our `constructor()` we'll add the variable that will hold our form.

```js
public newShowForm: FormGroup;
```

It's a public variable that we can bind to the HTML file, and it's of type `FormGroup`, this will hold the entire form and its properties for us.

Now, right below that one we need to add our Output:

```js
@Output() showCreationForm: EventEmitter<boolean> =   new EventEmitter();
```

The `@Output()` decorator lets the component know we'll be broadcasting the value of the `showCreationForm` variable to our parent component.

The `showCreationForm` variable is a boolean that will trigger when the form is submited (_more on that later_), and we're adding it so we can close the form once someone clicks on the submit button.

And the `EventEmitter` will be in charge of broadcasting (_or emiting_) the value of `showCreationForm` to the parent component.

Now we need to inject a couple of things to the class constructor:

```js
constructor(private formBuilder: FormBuilder, private firebaseService: FirebaseService) {
  this.newShowForm = this.formBuilder.group({
    showName: ['', Validators.required],
    showDescription: ['', Validators.required],
  });
}
```

Here's what we're doing:

- We're injecting formBuilder, it will create the form (_with logic, validators, etc_) for us.
- We're injecting the Firebase service so that it's available for use inside our class.
- We're initializing our form:
  - We're telling the component that our form has 2 properties and they're both required.

After setting up everything we need to create a function that will take the form values and send it to the Firebase service to be stored in the database, go ahead and add it after the `ngOnInit()` function:

```js
addShow(filledShowForm: FormGroup): void {
  const showName: string = filledShowForm.value.showName;
  const showDescription: string = filledShowForm.value.showDescription;
  this.firebaseService.createShow(showName, showDescription)
    .then(() => {
      this.newShowForm.reset();
      this.showCreationForm.emit(false);
    });
}
```

Let's go over the function:

- It takes the entire form as a parameter.
- It's extracting the `showName` and `showDescription` from the form.
- It's sending the data to the Firebase service. (_We'll create this function next_)
- After the show is successfully created it will clear the form and let the parent component know the form was submitted.

Right now you should be seeing a red underline on `.createShow()` saying something like this: **Property `'createShow'` does not exist on type `'FirebaseService'`.**

That's because we're calling the `.createShow()` function from the Firebase service but we haven't created the function yet, so let's do it right now.

## Creating a Service

Open the `firebase.service.ts` file, we'll start by importing what we need at the top of the file:

```js
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import {
  AngularFirestore,
  AngularFirestoreCollection,
  AngularFirestoreDocument,
} from '@angular/fire/firestore';
```

We're importing `Observable`, `AngularFirestoreCollection`, and `AngularFirestoreDocument` for type-setting only.

> NOTE: We use TypeScript and it's aways a good idea to set the type of the variable, propery, function return, etc. This way we get better autocompletion from the editor, and make fewer syntax mistakes.

And we're importing `AngularFirestore` to call our Firestore database.

The next thing we'l add is a variable to hold our TV Show Collection, we'll create it right before the constructor and then use the constructor to initialize it:

```js
tvShowCollection: AngularFirestoreCollection<any>;

constructor(private db: AngularFirestore) {
  this.tvShowCollection = db.collection('tvShowList');
}
```

Now let's take a little break to do 2 things:

First, let's go over a little theory of Firestore. The Firestore database is composed of Collections and Documents, the best way to understand that is with an example:

Our TV Show is a document, it has an ID, a name, a description, and any other property you want to add. And a Collection is nothing more than a list that holds those documents.

So in the above code snippet, `tvShowCollection` is a list that will hold all the TV Shows we create and store in the database.

And second, it really bugs me to use `<any>` when working with TypeScript, using `any` makes us waste all the TypeScript powers and we should correct that.

We're using `any` but we know the type of the collection, it's TV Show, because that collection will have a list of TV Shows, knowing that, we can create a TVShow interface with the properties.

## Creating an Interface

Go ahead and open the **Angular Generator** again (_right-click the app/ folder_) and choose **Interface**, name it **show**.

It will create the file `show-interface.ts`, open it and make it look like this:

```js
export interface TVShow {
  showId: string;
  showName: string;
  showDescription: string;
}
```

We're telling our app that the interface `TVShow` has 3 properties, `showId`, `showName`, and `showDescription`. They are all of type `string`.

Now we can go back to our `firebase.service.ts` and change that code snippet to look like this:

```js
tvShowCollection: AngularFirestoreCollection<TVShow>;

constructor(private db: AngularFirestore) {
  this.tvShowCollection = db.collection('tvShowList');
}
```

It will tell you that it doesn't know what `TVShow` is, so we need to add it to the page imports at the begining of the file:

```js
import { TVShow } from './show-interface';
```

Now that we have that taken care of, it's time to add our `createShow()` function to the service, go ahead and add it after the constructor:

```js
public createShow(showName: string, showDescription: string): Promise<void> {
  const showId: string = this.db.createId();
  return this.tvShowCollection
    .doc(showId)
    .set({ showName, showDescription, showId });
}
```

Here's what's going on:

- The function `createShow()` takes 2 arguments.
  - `showName` and `showDescription`.
  - They're both type string.
- It returns a promise of type void (_This is an empty promise just to let us know it worked_).
- We're asking Firestore to create an ID for us.
- Then we're navigating to the document with that ID (_It doesn't exists, but no worries, the function creates it_).
- We're adding the value of the properties to the new TV Show document.

Now that we've created all of the component's logic, let's start creating the view, move into the `add-show.component.html` file and add a form:

```html
<form [formGroup]="newShowForm"></form>
```

We're binding our form to the `newShowForm` we created in the TypeScript file, now let's start with the inputs, we need to add an input to hold the forms name:

```html
<form [formGroup]="newShowForm">
  <label class="formItem">
    <span class="form-label">Show Name:</span>
    <input type="text" formControlName="showName" />
  </label>
</form>
```

It's a regular input type text and we're binding it to the `showName` property of our `newShowForm`.

Next, we need to add a description field:

```html
<form [formGroup]="newShowForm">
  <label class="formItem">
    <span class="form-label">Show Name:</span>
    <input type="text" formControlName="showName" />
  </label>

  <label class="formItem">
    <span class="form-label">Description:</span>
    <textarea formControlName="showDescription"></textarea>
  </label>
</form>
```

It's the same as above, we're just changing the input for a `textarea` to hold more information, and lastly we need to add a button to submit the form:

```html
<form [formGroup]="newShowForm">
  <label class="formItem">
    <span class="form-label">Show Name:</span>
    <input type="text" formControlName="showName" />
  </label>

  <label class="formItem">
    <span class="form-label">Description:</span>
    <textarea formControlName="showDescription"></textarea>
  </label>

  <button class="formItem" (click)="addShow(newShowForm)" [disabled]="!newShowForm.valid">
    Save Show Info
  </button>
</form>
```

Once clicked, the button calls the `addShow(newShowForm)` passing the form as a parameter. We added `[disabled]="!newShowForm.valid"`to make sure the button is only enabled when the form is valid (_both fields have value_).

And lastly we're going to give our form some styles, open `add-show.component.css` and paste this in:

```css
form {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  font-family: 'Arial';
  margin: 10px auto;
  width: 80%;
}

.formItem {
  margin: 20px 0;
  width: 100%;
}

.form-label {
  display: block;
  margin-bottom: 10px;
}

input {
  width: 100%;
  padding: 12px 20px;
  box-sizing: border-box;
  border: 2px solid #ccc;
  border-radius: 4px;
  background-color: #f8f8f8;
  font-size: 16px;
  resize: none;
}

textarea {
  width: 100%;
  height: 150px;
  padding: 12px 20px;
  box-sizing: border-box;
  border: 2px solid #ccc;
  border-radius: 4px;
  background-color: #f8f8f8;
  font-size: 16px;
  resize: none;
}

button {
  width: 100%;
  height: 40px;
  border-radius: 10px;
  background-color: #ffffff;
  color: #000000;
  text-align: center;
  text-decoration: none;
  font-size: 16px;
  margin: 10px 0;
  border: 1px solid #0037b1;
}
```

We won't go into details about the styles, but they are mainly giving some margins, paddings and colors to make sure everything looks nice.

## Let's test it

By now you've written a good amount of code but haven't seen the results of it yet, this can be a little frustraiting so let's take a moment to display our form and create new shows.

Let's start by opening the `app.component.html` file and pasting in the following code:

```html
<h1 class="page-title">My TV Show Database</h1>

<p>
  Welcome to <strong>My TV Show Database</strong>, an app where you can store the
  information about your favorite TV Shows!
</p>

<button (click)="showCreateForm()" class="show-form-button">Add a new show</button>

<app-add-show (showCreationForm)="hideForm()" *ngIf="showForm"></app-add-show>
```

We're adding some text to the app, then we're adding a button to show/hide our newly created form, and then we're adding our `add-show` component using the `<app-add-show></app-add-show>` tag.

If you're not sure where the `<app-add-show></app-add-show>` comes from, take a look at the `@Component()` decorator inside the `add-show` component.

![AddShow Component Decorator](img/add-show-component.png)

We're passing it the `Output()` we created in the component `showCreationForm`, once it triggers it will call the function `hideForm()` and we're setting a conditional that the form will only be visible if the `showForm` property is `true`.

Open `app.component.css` now and add the styles:

```css
h1,
p {
  text-align: center;
  font-family: Arial;
}

.show-form-button {
  background-color: #ffffff;
  border: 1px solid rgb(10, 124, 10);
  border-radius: 10px;
  color: #000000;
  display: block;
  font-size: 18px;
  height: 40px;
  margin: 25px auto;
  text-align: center;
  text-decoration: none;
  width: 50%;
}
```

Like before, we're just giving some love to our app, making it look nicer.

And lastly, let's open the `app.component.ts` file and make it look like this:

```js
import { Component } from '@angular/core';
import { Observable } from 'rxjs';
import { FirebaseService } from './firebase.service';
import { TVShow } from './show-interface';

@Component({
  selector: 'my-app',
  templateUrl: './app.component.html',
  styleUrls: [ './app.component.css' ]
})
export class AppComponent {
  public showForm = false;
  constructor() {}

  showCreateForm(): void {
    this.showForm = !this.showForm;
  }

  hideForm(): void {
    this.showForm = false;
  }
}
```

Here's what's going on:

- We're adding the `showForm` property and setting it to `false`, that way the form won't be visible on page load.
- We're adding the `showCreateForm()` function that sets the value of `showForm` to the opposite of what it currently is (_this will either show or hide the form_).
- We're adding the `hideForm()` function, this will make the value of `showForm` equal to `false` and hide our form.

Right now you should have something like this on your app:

![Show form button](img/show-form-button.png)

And when you click the button it either shows or hides the form depending on its current state.

When you're ready let's move to the next step, where we'll be adding the **R** part of CRUD, we'll learn how to read data from Firestore and display it inside our app.
