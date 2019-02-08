# 6. Removing Information 💣

In this lesson we'll finish building our application. The only missing functionality is the ability to remove content from the database.

Fortunately, we've already done all the leg work for this, all we need now is a way to call a function that removes the TV Show.

First, open `app.component.html` and look for our TVShow card, remember it looks like this:

```markup
<div class="card" *ngFor="let show of showList | async">
  <img (click)="updatePicture(show.id)" [src]="show.picture" width="50px" height="50px" />
  <input (change)="updateName(show.id, $event.target.value)" [value]="show.name" />
</div>
```

Let's add a button inside that card that triggers a `remove()` function passing the show's ID:

{% code-tabs %}
{% code-tabs-item title="app.component.html" %}
```markup
<div class="card" *ngFor="let show of showList | async">
  <img (click)="updatePicture(show.id)" [src]="show.picture" width="50px" height="50px" />
  <input (change)="updateName(show.id, $event.target.value)" [value]="show.name" />
  <button (click)="remove(show.id)">❌</button>
</div>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Nothing weird there, we're creating a button, and calling the `remove()`function, now let's move to our `app.component.ts` file and create that remove function:

{% code-tabs %}
{% code-tabs-item title="app.component.ts" %}
```javascript
remove(id: string) {
  this.showCollection.doc(id).delete();
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

The function is going into our specific TV Show and calling the `.delete()` method. And that's it! Whenever you click the X it's going to remove the show from the Firestore database.

## Don't delete by accident 😱

Ok, this isn't required, but it's a good practice, let's add a confirmation alert before calling the delete function. The last thing we want to do is to click by accident and permanently lose a database record.

We'll use the browser native confirmation prompt, so make the `remove()` function look like this:

```javascript
remove(id: string) {
  if(confirm('Are you sure to delete the show from your list?')){
    this.showCollection.doc(id).delete();
  }
}
```

That way the app will show an alert asking you if you're sure about deleting the record.

## ⚡ CONGRATULATIONS ⚡

**Congratulations**, you're done with the workshop, by now you should have a fully functional app, if you're up for a new challenge, let's move to a bonus section where you'll learn a bit more about Angular best practices.

![](https://media.giphy.com/media/nDSlfqf0gn5g4/giphy.gif)

## HELP US! 🙏

Before moving onto to the next section, please help us by filling in the feedback form:

* \*\*\*\*[**FEEBACK FORM**](https://docs.google.com/forms/d/e/1FAIpQLSd-IHXNx8q7xQnCyk9gJHGaOTU02cTmRzPkmv3T3II3wEZNgw/viewform)\*\*\*\*





