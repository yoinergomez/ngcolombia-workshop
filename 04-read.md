# 5. Reading and Updating 📱

In the previous lesson, we learned how to store data in Firestore. Now, it's time to learn how to read that data and display it inside our application.

Remember in the last lesson when we did this:

{% code-tabs %}
{% code-tabs-item title="app.component.ts" %}
```javascript
constructor(private db: AngularFirestore) {
  this.showCollection = db.collection('shows'); // remember to use a different name
  this.showList = this.showCollection.valueChanges();
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* That bit of code right there is enough to create a reference to our TV Shows, and then transform that into an `Observable` we can display on our page.
* All you need to do now is adding the markup \(_The HTML_\) and the app will show the cards you start creating 👌

So move to our `app.component.html` file and add the following code:

{% code-tabs %}
{% code-tabs-item title="app.component.html" %}
```markup
<div class="card" *ngFor="let show of showList | async">
  <img [src]="show.picture" width="50px" height="50px" />
  <input [value]="show.name" />
</div>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Here's what's going on:

* `*ngFor="let show of showList | async"` is telling our angular app that the `<div>`is going to look at the `showList` property and repeat itself with every show it finds inside.
* The `async` pipe is telling the app that `showList` has an asynchronous value.
* `<img [src]="show.picture" width="50px" height="50px" />` is accessing at the `picture` property of our show and setting it as the source for the image.
* `<input [value]="show.name" />` is creating an input pre-populated with our TV Show's name.

### You made it! 🎉🎉🎉

Right now you should be able to see the TV Shows you add to your database, so go ahead and click the '**ADD**' button a few times and notice how the cards start popping out on your page.

![](https://media.giphy.com/media/TdfyKrN7HGTIY/giphy.gif)

{% hint style="info" %}
If it is not showing the results, call one of the mentors! 🖐
{% endhint %}

## Updating objects from our database 🦄

Now that we can see our cards, the next step would is to update some of the show's information, such as the show's name, and the picture \(_remember, we're showing a random picture there_\).

For that let's modify the HTML we just wrote, we want to add event handlers on the image and the input so that when users click the image or write the show's name we can call functions that save that information to Firebase. 💻

Right now it looks like this:

{% code-tabs %}
{% code-tabs-item title="app.component.html" %}
```markup
<div class="card" *ngFor="let show of showList | async">
  <img [src]="show.picture" width="50px" height="50px" />
  <input [value]="show.name" />
</div>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Let's add a click handler for the image that triggers a function called `updatePicture()` and takes the show's ID as a parameter:

{% code-tabs %}
{% code-tabs-item title="app.component.ts" %}
```markup
<div class="card" *ngFor="let show of showList | async">
  <img (click)="updatePicture(show.id)" [src]="show.picture" width="50px" height="50px" />
  <input [value]="show.name" />
</div>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Now let's create a change handler on the input that calls the `updateName()` function when we change the show's name:

{% code-tabs %}
{% code-tabs-item title="app.component.ts" %}
```markup
<div class="card" *ngFor="let show of showList | async">
  <img (click)="updatePicture(show.id)" [src]="show.picture" width="50px" height="50px" />
  <input (change)="updateName(show.id, $event.target.value)" [value]="show.name" />
</div>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

* Notice how we're passing the show's ID and whatever text is on the input

Now we need to go to our `app.component.ts` page and create both of those functions, let's start with the update image one:

{% code-tabs %}
{% code-tabs-item title="app.component.ts" %}
```javascript
updatePicture(id: string) {
  const picture = prompt('Please enter an image URL:');
  if (picture) {
    this.showCollection.doc(id).update({ picture });
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

{% hint style="info" %}
**What is updatePicture doing?**

* **👉** Calling the browser's native prompt alert asking you for the new image URL.
* **👉** Getting whatever you write and assigning it to the variable `picture`.
* **👉** If you do write something it goes inside the Firestore document and updates the property `picture`
{% endhint %}

Let's do the same for our update name function:

{% code-tabs %}
{% code-tabs-item title="app.component.ts" %}
```javascript
updateName(id: string, name: string) {
  this.showCollection.doc(id).update({ name });
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

We're creating a function that goes into our show and updates the name property.

{% hint style="info" %}
## `.set()` vs `.update()`

You might have noticed that we've used 2 different ways to add data to Firestore, one was using the method `.set()` and the other one was using the `.update()` method.

What's the difference between these two?

Well, `.set()` is destructive and `.update()` isn't.

That meas that when you use `.set({ name: 'Jorge' })` it will delete every single property of that object and leave only the name property.

And when you use `.update({ name: 'Jorge' })` it's going to look for the `name` property and change its value, all the other properties will remain the same.
{% endhint %}

## We are done here! 🤙

Let's just into the step \#6 to learn how to remove the information from Firestores!

