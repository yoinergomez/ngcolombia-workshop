# Updating objects from our database

In the previous lesson we learned how to read data from Firestore. Now, it's time to learn how to update the data from the TV Shows we already created.

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

Now we need a way to let our form know that if we have those inputs it shoud assign them as a value of the form properties, go ahead and add this to your `ngOnInit()` function:

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

- We changed the function's name to match its new responsability.
- We added a new parameter `oldShowId` of type string, and we're setting it to be `null`by default.
- We're asking if the component is sending us the `oldShowId` or not.
  - If it's sending the ID we move to that document and use the `.update()` to save the new name or description of the show.
  - If there's no ID we execute the create functionality we created at the begining.

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

All we're doing is adding the inputs to the
