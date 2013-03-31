Early beginnings of a Firebase adapter.

## What? Why?

Firebase is powerful because it sort of works as a standard CRUD store, with the unique twist of being able to live update models and collections.

Honestly, it's a bit weird mixing standard Ember Data CRUD operations with real-time updates, but it seems to be working out so far, and only feels *slightly* hacky. 

This adapter can be used as a normal CRUD adapter, or to automatically add or remove models and properties as they are changed on your Firebase backend. This makes it shockingly powerful for combining persistence and live updates.

## Usage

Don't, yet. I mean the amount of stuff left to implement is *staggering*. But if you really want to...

Create an Ember Data store:

```javascript
App.store = DS.Store.create({
  revision: 12,
  adapter: DS.Firebase.Adapter.create({
    dbName: "your-db-name-here"
  })
});
```

This works out of the box as a standard CRUD store. 

### Real Time

For real time *collections* - that is, an array of your models at a particular resource that will automatically create and delete models based on changes on your Firebase reference - you'll want to use `YourModel.find()`. If you'd rather have static collections, just use `YourModel.find({})`.

Real time collections are only supported when they're "bound" to an entire resource - that is, doing a find with any kind of query (including a blank one) will create a static collection.

For real time *models*, subclass `DS.Firebase.LiveModel`. Note that will add any child references added to your model on your Firebase backend, but will only be able to *persist* references that have been defined on your model with `DS.attr()`.

### (theoretical) Associations

Associations aren't implemented yet, but here's how they will be!

In Firebase, all an "association" is is a child that has more children. Basically, Firebase is a big ol' tree. Either a value is a primative of some sort, and thus is a property on a model in Ember Data, or it has its own children, in which case it's a model of its own.

Thus:

```javascript
App.Post = DS.Model.create({
  title: DS.attr("string"),
  content: DS.attr("string"),

  hasMany: "App.Comment",
});

App.Comment = DS.Model.create({
  author: DS.attr("string"),
  content: DS.attr("string"),

  belongsTo: "App.Post"
});
```

Will be turned into an *embedded association*. Essentially:

```javascript
var post = firebase.child("post").child(postId);
var comment = post.child("comments").child(commentId);
```

But what if you want *relational associations*? I dunno yet.

## Implemented

### CRUD

* `find(id)`
* `findAll()`
* `createRecord()`
* `updateRecord()`

### Live Updating

#### Collections

* `child_added`

#### Models

* `child_added`
* `child_changed

## Todo

Literally just about everything. I mean assocations, most live updating, etc. Oh, and error handling. Dear lord, error handling.

And assocations are going to be a trainwreck, probably. So I'm excited for that!

## Tests

Create a Firebase DB for testing, then create a `tests/firebase_db.js` file like so:

```javascript
window.DB_NAME = "your_db_subdomain_here";
```

Throw up a static server in the root directory of this repo, and navigate your browser to `test/` to run tests.
