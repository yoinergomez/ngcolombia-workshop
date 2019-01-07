# Updating objects from our database

In the previous lesson, we learned how to read data from Firestore. Now, it's time to learn how to update the data from the TV Shows we already created.

The good news, most of this is already done, we will need to make a few edits to our app and add a new function, but no new components or anything like that.

## Update Component?

We're not going to be creating a component to update the show's information, why? Because we already have it, we can use the `add-show` not only to create a component but also to edit one.

The first thing we need to do is to edit our `add-show.component.ts` file so that the component accepts the current TV Show's information as `@Input()` properties.

Add the inputs before the constructor, it should look like this now:

```js
public newShowForm: FormGroup;
@Input() oldShowName: string;
@Input() oldShowDescription: string;
@Input() showIdToUpdate: string = null;
@Output() showCreationForm: EventEmitter<boolean> =   new EventEmitter();

constructor(private formBuilder: FormBuilder, private firebaseService: FirebaseService) {
  this.newShowForm = this.formBuilder.group({
    showName: ['', Validators.required],
    showDescription: ['', Validators.required],
  });
}
```

Now we need a way to let our form know that if we have those inputs it should assign them as a value of the form properties, go ahead and add this to your `ngOnInit()` function:

```js
ngOnInit() {
  this.newShowForm.patchValue({
    showName: this.oldShowName,
    showDescription: this.oldShowDescription
  });
}
```

We can also use the same create functions to update, we just need to pass the right data, go ahead and look for the `addShow()` function, it should look like this:

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

Refactor it to look like this:

```js
addOrUpdateShow(filledShowForm: FormGroup): void {
  const showName: string = filledShowForm.value.showName;
  const showDescription: string = filledShowForm.value.showDescription;
  this.firebaseService.createOrUpdateShow(showName, showDescription, this.showIdToUpdate)
    .then(() => {
      this.newShowForm.reset();
      this.showCreationForm.emit(false);
    });
}
```

We're changing the functions' names to match their new functionality, and we're passing the `showIdToUpdate` property to the Firebase service so it knows which show we're going to update.

You've probably guessed what's next, we have to refactor the Firebase Service so the `createShow()` becomes the `createOrUpdateShow()` and reflects this new functionality.

Go ahead and open the `firebase.service.ts` file and find the `createShow()` function, it should look like this:

```js
public createShow(showName: string, showDescription: string): Promise<void> {
  const showId: string = this.db.createId();
  return this.tvShowCollection
    .doc(showId)
    .set({ showName, showDescription, showId });
}
```

We're going to refactor it to look like this:

```js
public createOrUpdateShow(
  showName: string,
  showDescription: string,
  oldShowId: string = null): Promise<void> {
  if (oldShowId) {
    return this.tvShowCollection.doc(oldShowId).update({ showName, showDescription });
  } else {
    const showId: string = this.db.createId();
    return this.tvShowCollection
      .doc(showId)
      .set({ showName, showDescription, showId });
  }
}
```

Here's what's going on:

- We changed the function's name to match its new responsibility.
- We added a new parameter `oldShowId` of type string, and we're setting it to be `null`by default.
- We're asking if the component is sending us the `oldShowId` or not.
  - If it's sending the ID we move to that document and use the `.update()` to save the new name or description of the show.
  - If there's no ID we execute the create functionality we created at the beginning.

Now go ahead and open the `app.component.html`, we'll find our `<app-add-show>` tag that should look like this:

```html
<app-add-show (showCreationForm)="hideForm()" *ngIf="showForm"></app-add-show>
```

And replace it with this:

```html
<app-add-show
  [oldShowName]="oldShowName"
  [showIdToUpdate]="showIdToUpdate"
  [oldShowDescription]="oldShowDescription"
  (showCreationForm)="hideForm()"
  *ngIf="showForm"
></app-add-show>
```

All we're doing is adding the inputs to the component call, now we need to actually get those variables from our `app.component.ts` file.

First, we'l declare the properties right before our constructor:

```js
public tvShowList: Observable<TVShow[]>;
public showForm = false;
public oldShowName = '';
public oldShowDescription = '';
public showIdToUpdate: string;
```

Next we'l create a function that takes the TV Show as a parameter and returns the information to update to us, we'll call it `activateEditShow()`:

```js
activateEditShow(tvShow: TVShow): void {
  this.showForm = true;
  this.showIdToUpdate = tvShow.showId;
  this.oldShowName = tvShow.showName;
  this.oldShowDescription = tvShow.showDescription;
}
```

The function is setting `showForm` to `true`to display the form, and then setting all the information we need to perform the update.

Now we need to figure out where do we call this function from, it has to be from a place where we have access to the current TV Show object.

The only place it makes sense to do this is inside `CardComponent`, since it's where we have access to the TV show, we'll pass it through the **Edit** function.

Open `card.component.html` and find the action buttons, they look like this:

```html
<div class="show-actions">
  <button (click)="deleteShow()" class="button delete">Delete Show</button>
  <button (click)="editShow()" class="button edit">Edit Show</button>
</div>
```

We're going to use the `editShow()` function to send the `TVShow`object to the main component, for that we first need to send the proper parameters to the function:

```html
<div class="show-actions">
  <button (click)="deleteShow(showId, showName)" class="button delete">
    Delete Show
  </button>
  <button (click)="editShow(showId, showName, showDescription)" class="button edit">
    Edit Show
  </button>
</div>
```

We're sending the `showId` and `showName` to the `deleteShow()` function, and we're also adding the `showDescription` to the `editShow()` function.

Now let's move to `card.component.ts` and add a new property, it's going to be an `@Output()` called `showIdChanged`:

```js
@Input() showName: string;
@Input() showDescription: string;
@Input() showId: string;
@Output() showIdChanged: EventEmitter<TVShow> =   new EventEmitter();
```

After we create the property we're going to create the `editShow()` function, it needs to emit the show's information to the parent component:

```js
editShow(showId: string, showName: string, showDescription: string): void {
  const newShowInfo: TVShow = { showId, showName, showDescription }
  this.showIdChanged.emit(newShowInfo);
}
```

And that's it when you click on the **Edit Show** button from any of the cards you should see the form display already pre-filled with the show's information.
