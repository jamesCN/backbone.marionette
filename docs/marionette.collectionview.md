# Marionette.CollectionView

The `CollectionView` will loop through all of the models in the
specified collection, render each of them using a specified `itemView`,
then append the results of the item view's `el` to the collection view's
`el`.

CollectionView extends directly from Marionette.View. Please see
[the Marionette.View documentation](marionette.view.md)
for more information on available features and functionality.

Additionally, interactions with Marionette.Region
will provide features such as `onShow` callbacks, etc. Please see
[the Region documentation](marionette.region.md) for more information.

## Documentation Index

* [CollectionView's `itemView`](#collectionviews-itemview)
  * [CollectionView's `getItemView`](#collectionviews-getitemview)
  * [CollectionView's `itemViewOptions`](#collectionviews-itemviewoptions)
  * [CollectionView's `itemViewEventPrefix`](#collectionviews-itemvieweventprefix)
  * [CollectionView's `itemEvents`](#collectionviews-itemevents)
  * [CollectionView's `buildItemView`](#collectionviews-builditemview)
  * [CollectionView's `addItemView`](#collectionviews-additemview)
* [CollectionView's `emptyView`](#collectionviews-emptyview)
  * [CollectionView's `getEmptyView`](#collectionviews-getemptyview)
* [Callback Methods](#callback-methods)
  * [onBeforeRender callback](#onbeforerender-callback)
  * [onRender callback](#onrender-callback)
  * [onBeforeClose callback](#beforeclose-callback)
  * [onClose callback](#onclose-callback)
  * [onBeforeItemAdded callback](#onbeforeitemadded-callback)
  * [onAfterItemAdded callback](#onafteritemadded-callback)
  * [onItemRemoved callback](#onitemremoved-callback)
* [CollectionView Events](#collectionview-events)
  * ["before:render" event](#beforerender-event)
  * ["render" event](#render-event)
  * ["before:close" event](#beforeclose-event)
  * ["closed" / "collection:closed" event](#closed--collectionclosed-event)
  * ["before:item:added" / "after:item:added" event](#beforeitemadded--afteritemadded-event)
  * ["item:removed" event](#itemremoved-event)
  * ["itemview:\*" event bubbling from child views](#itemview-event-bubbling-from-child-views)
* [CollectionView render](#collectionview-render)
* [CollectionView: Automatic Rendering](#collectionview-automatic-rendering)
* [CollectionView: Re-render Collection](#collectionview-re-render-collection)
* [CollectionView's appendHtml](#collectionviews-appendhtml)
* [CollectionView's children](#collectionviews-children)
* [CollectionView close](#collectionview-close)

## CollectionView's `itemView`

Specify an `itemView` in your collection view definition. This must be
a Backbone view object definition, not an instance. It can be any
`Backbone.View` or be derived from `Marionette.ItemView`.

```js
MyItemView = Backbone.Marionette.ItemView.extend({});

Backbone.Marionette.CollectionView.extend({
  itemView: MyItemView
});
```

Item views must be defined before they are referenced by the
`itemView` attribute in a collection view definition. Use `getItemView`
to lookup the definition as child views are instantiated.

Alternatively, you can specify an `itemView` in the options for
the constructor:

```js
MyCollectionView = Backbone.Marionette.CollectionView.extend({...});

new MyCollectionView({
  itemView: MyItemView
});
```

If you do not specify an `itemView`, an exception will be thrown
stating that you must specify an `itemView`.

### CollectionView's `getItemView`
The value returned by this method is the `ItemView` class that will be instantiated when a `Model` needs to be initially rendered.

This method gives you the ability to customize your `ItemView` class based on attributes of each item `Model`.

```js
var FooBar = Backbone.Model.extend({
  defaults: {
    isFoo: false
  }
});

var FooView = Backbone.Marionette.ItemView.extend({
  template: '#foo-template'
});
var BarView = Backbone.Marionette.ItemView.extend({
  template: '#bar-template'
});


var MyCollectionView = Backbone.Marionette.CollectionView.extend({
  getItemView: function(item) {
    // Choose which view class to render,
    // depending on the properties of the item model
    if  (item.get('isFoo')) {
      return FooView;
    }
    else {
      return BarView;
    }
  }
});

var collectionView = new MyCollectionView();
var foo = new FooBar({
  isFoo: true
});
var bar = new FooBar({
  isFoo: false
});

// Renders a FooView
collectionView.collection.add(foo);

// Renders a BarView
collectionView.collection.add(bar);
```

### CollectionView's `itemViewOptions`

There may be scenarios where you need to pass data from your parent
collection view in to each of the itemView instances. To do this, provide
a `itemViewOptions` definition on your collection view as an object
literal. This will be passed to the constructor of your itemView as part
of the `options`.

```js
ItemView = Backbone.Marionette.ItemView({
  initialize: function(options){
    console.log(options.foo); // => "bar"
  }
});

CollectionView = Backbone.Marionette.CollectionView({
  itemView: ItemView,

  itemViewOptions: {
    foo: "bar"
  }
});
```

You can also specify the `itemViewOptions` as a function, if you need to
calculate the values to return at runtime. The model will be passed into
the function should you need access to it when calculating
`itemViewOptions`. The function must return an object, and the attributes
of the object will be copied to the `itemView` instance's options.

```js
CollectionView = Backbone.Marionette.CollectionView({
  itemViewOptions: function(model, index) {
    // do some calculations based on the model
    return {
      foo: "bar",
      itemIndex: index
    }
  }
});
```

### CollectionView's `itemViewEventPrefix`

You can customize the event prefix for events that are forwarded
through the collection view. To do this, set the `itemViewEventPrefix`
on the collection view.

```js
var CV = Marionette.CollectionView.extend({
  itemViewEventPrefix: "some:prefix"
});

var c = new CV({
  collection: myCol
});

c.on("some:prefix:render", function(){
  // item view was rendered
});

c.render();
```

The `itemViewEventPrefix` can be provided in the view definition or
in the constructor function call, to get a view instance.

### CollectionView's `itemEvents`
You can specify an `itemEvents` hash or method which allows you to capture all bubbling itemEvents without having to manually set bindings. The values of the hash can either be a function or a string that is the name of a method on the collection view.

```js
itemEvents: {
  "render": function() {
    console.log("an itemView has been rendered");
  },
  "onItemClose": "someFn" // where the collection view has a method `someFn`
}
```

You can also use a method for `itemEvents` that returns a hash.

```js
itemEvents: function() {
  return {
    "render": function() {
      console.log("an itemView has been rendered");
    }
  }
}
```

### CollectionView's `buildItemView`

When a custom view instance needs to be created for the `itemView` that
represents an item, override the `buildItemView` method. This method
takes three parameters and returns a view instance to be used as the
item view.

```js
buildItemView: function(item, ItemViewType, itemViewOptions){
  // build the final list of options for the item view type
  var options = _.extend({model: item}, itemViewOptions);
  // create the item view instance
  var view = new ItemViewType(options);
  // return it
  return view;
},
```

### CollectionView's `addItemView`

The `addItemView` method is responsible for rendering the `itemViews` and adding them to the HTML for the `collectionView` instance. It is also responsible for triggering the events per `ItemView`. In most cases you should not override this method. However if you do want to short circut this method, it can be accomplished via the following.

```js
Backbone.Marionette.CollectionView.extend({
  addItemView: function(item, ItemView, index){
    if (item.shouldBeShown()) {
      Backbone.Marionette.CollectionView.prototype.addItemView.apply(this, arguments);
    }
  }
});
```

## CollectionView's `emptyView`

When a collection has no items, and you need to render a view other than
the list of itemViews, you can specify an `emptyView` attribute on your
collection view.

```js
NoItemsView = Backbone.Marionette.ItemView.extend({
  template: "#show-no-items-message-template"
});

Backbone.Marionette.CollectionView.extend({
  // ...

  emptyView: NoItemsView
});
```

### CollectionView's `getEmptyView`

If you need the `emptyView`'s type chosen dynamically, specify `getEmptyView`:

```js
Backbone.Marionette.CollectionView.extend({
  // ...

  getEmptyView: function() {
    // custom logic
    return NoItemsView;
  }
```

This will render the `emptyView` and display the message that needs to
be displayed when there are no items.

If you want to control when the empty view is rendered, you can override
`isEmpty`:

```js
Backbone.Marionette.CollectionView.extend({
  isEmpty: function(collection) {
    // some logic to calculate if the view should be rendered as empty
    return someBoolean;
  }
});
```

## Callback Methods

There are several callback methods that can be provided on a
`CollectionView`. If they are found, they will be called by the
view's base methods. These callback methods are intended to be
handled within the view definition directly.

### onBeforeRender callback

A `onBeforeRender` callback will be called just prior to rendering
the collection view.

```js
Backbone.Marionette.CollectionView.extend({
  onBeforeRender: function(){
    // do stuff here
  }
});
```

### onRender callback

After the view has been rendered, a `onRender` method will be called.
You can implement this in your view to provide custom code for dealing
with the view's `el` after it has been rendered:

```js
Backbone.Marionette.CollectionView.extend({
  onRender: function(){
    // do stuff here
  }
});
```

### onBeforeClose callback

This method is called just before closing the view.

```js
Backbone.Marionette.CollectionView.extend({
  onBeforeClose: function(){
    // do stuff here
  }
});
```

### onClose callback

This method is called just after closing the view.

```js
Backbone.Marionette.CollectionView.extend({
  onClose: function(){
    // do stuff here
  }
});
```

### onBeforeItemAdded callback

This callback function allows you to know when an item / item view
instance is about to be added to the collection view. It provides access to
the view instance for the item that was added.

```js
Backbone.Marionette.CollectionView.extend({
  onBeforeItemAdded: function(itemView){
    // work with the itemView instance, here
  }
});
```

### onAfterItemAdded callback

This callback function allows you to know when an item / item view
instance has been added to the collection view. It provides access to
the view instance for the item that was added.

```js
Backbone.Marionette.CollectionView.extend({
  onAfterItemAdded: function(itemView){
    // work with the itemView instance, here
  }
});
```

### onItemRemoved callback

This callback function allows you to know when an item / item view
instance has been deleted or removed from the
collection.

```js
Backbone.Marionette.CollectionView.extend({
  onItemRemoved: function(itemView){
    // work with the itemView instance, here
  }
});
```

## CollectionView Events

There are several events that will be triggered during the life
of a collection view. Each of these events is called with the
[Marionette.triggerMethod](./marionette.functions.md) function,
which calls a corresponding "on{EventName}" method on the
view instance (see [above](#callback-methods)).

### "before:render" event


Triggers just prior to the view being rendered. Also triggered as
"collection:before:render" / `onCollectionBeforeRender`.

```js
MyView = Backbone.Marionette.CollectionView.extend({...});

var myView = new MyView();

myView.on("before:render", function(){
  alert("the collection view is about to be rendered");
});

myView.render();
```

### "render" event

A "collection:rendered" / `onCollectionRendered` event will also be fired. This allows you to
add more than one callback to execute after the view is rendered,
and allows parent views and other parts of the application to
know that the view was rendered.

```js
MyView = Backbone.Marionette.CollectionView.extend({...});

var myView = new MyView();

myView.on("render", function(){
  alert("the collection view was rendered!");
});

myView.on("collection:rendered", function(){
  alert("the collection view was rendered!");
});

myView.render();
```

### "before:close" event

Triggered just before closing the view. A "collection:before:close" /
`onCollectionBeforeClose` event will also be fired

```js
MyView = Backbone.Marionette.CollectionView.extend({...});

var myView = new MyView();

myView.on("collection:before:close", function(){
  alert("the collection view is about to be closed");
});

myView.close();
```

### "closed" / "collection:closed" event

Triggered just after closing the view, both with corresponding
method calls.

```js
MyView = Backbone.Marionette.CollectionView.extend({...});

var myView = new MyView();

myView.on("collection:closed", function(){
  alert("the collection view is now closed");
});

myView.close();
```

### "before:item:added" / "after:item:added" event

The "before:item:added" event and corresponding `onBeforeItemAdded`
method are triggered just after creating a new itemView instance for
an item that was added to the collection, but before the
view is rendered and added to the DOM.

The "after:item:added" event and corresponding `onAfterItemAdded`
method are triggered after rendering the view and adding it to the
view's DOM element.

```js
var MyCV = Marionette.CollectionView.extend({
  // ...

  onBeforeItemAdded: function(){
    // ...
  },

  onAfterItemAdded: function(){
    // ...
  }
});

var cv = new MyCV({...});

cv.on("before:item:added", function(viewInstance){
  // ...
});

cv.on("after:item:added", function(viewInstance){
  // ...
});
```

### "item:removed" event

Triggered after an itemView instance has been closed and
removed, when its item was deleted or removed from the
collection.

```js
cv.on("item:removed", function(viewInstance){
  // ...
});
```

### "itemview:\*" event bubbling from child views

When an item view within a collection view triggers an
event, that event will bubble up through the parent
collection view with "itemview:" prepended to the event
name.

That is, if a child view triggers "do:something", the
parent collection view will then trigger "itemview:do:something".

```js
// set up basic collection
var myModel = new MyModel();
var myCollection = new MyCollection();
myCollection.add(myModel);

// get the collection view in place
colView = new CollectionView({
  collection: myCollection
});
colView.render();

// bind to the collection view's events that were bubbled
// from the child view
colView.on("itemview:do:something", function(childView, msg){
  alert("I said, '" + msg + "'");
});

// hack, to get the child view and trigger from it
var childView = colView.children[myModel.cid];
childView.trigger("do:something", "do something!");
```

The result of this will be an alert box that says
"I said, 'do something!'".

Also note that you would not normally grab a reference to
the child view the way this is showing. I'm merely using
that hack as a way to demonstrate the event bubbling.
Normally, you would have your item view listening to DOM
events or model change events, and then triggering an event
of its own based on that.

## CollectionView render

The `render` method of the collection view is responsible for
rendering the entire collection. It loops through each of the
items in the collection and renders them individually as an
`itemView`.

```js
MyCollectionView = Backbone.Marionette.CollectionView.extend({...});

new MyCollectionView().render().done(function(){
  // all of the children are now rendered. do stuff here.
});
```

## CollectionView: Automatic Rendering

The collection view binds to the "add", "remove" and "reset" events of the
collection that is specified.

When the collection for the view is "reset", the view will call `render` on
itself and re-render the entire collection.

When a model is added to the collection, the collection view will render that
one model in to the collection of item views.

When a model is removed from a collection (or destroyed / deleted), the collection
view will close and remove that model's item view.

## CollectionView: Re-render Collection

If you need to re-render the entire collection, you can call the
`view.render` method. This method takes care of closing all of
the child views that may have previously been opened.

## CollectionView's appendHtml

By default the collection view will append the HTML of each ItemView
into the element buffer, and then call jQuery's `.append` once at the
end to move the HTML into the collection view's `el`.

You can override this by specifying an `appendHtml` method in your
view definition. This method takes three parameters and has no return
value.

```js
Backbone.Marionette.CollectionView.extend({

	// The default implementation:
  appendHtml: function(collectionView, itemView, index){
    if (collectionView.isBuffering) {
      // buffering happens on reset events and initial renders
      // in order to reduce the number of inserts into the
      // document, which are expensive.
      collectionView.elBuffer.appendChild(itemView.el);
    }
    else {
      // If we've already rendered the main collection, just
      // append the new items directly into the element.
      collectionView.$el.append(itemView.el);
    }
  },

  // Called after all children have been appended into the elBuffer
  appendBuffer: function(collectionView, buffer) {
    collectionView.$el.append(buffer);
  },

  // called on initialize and after appendBuffer is called
  initRenderBuffer: function() {
    this.elBuffer = document.createDocumentFragment();
  }

});
```

The first parameter is the instance of the collection view that
will receive the HTML from the second parameter, the current item
view instance.

The third parameter, `index`, is the index of the
model that this itemView instance represents, in the collection
that the model came from. This is useful for sorting a collection
and displaying the sorted list in the correct order on the screen.

Overrides of `appendHtml` that don't take into account the element
buffer will work fine, but won't take advantage of the 60x performance
increase the buffer provides.

## CollectionView's children

The CollectionView uses [Backbone.BabySitter](https://github.com/marionettejs/backbone.babysitter)
to store and manage its child views. This allows you to easily access
the views within the collection view, iterate them, find them by
a given indexer such as the view's model or collection, and more.

```js
var cv = new Marionette.CollectionView({
  collection: someCollection
});

cv.render();


// retrieve a view by model
var v = cv.children.findByModel(someModel);

// iterate over all of the views and process them
cv.children.each(function(view){

  // process the `view` here

});
```

For more information on the available features and functionality of
the `.children`, see the [Backbone.BabySitter documentation](https://github.com/marionettejs/backbone.babysitter).

## CollectionView close

CollectionView implements a `close` method, which is called by the
region managers automatically. As part of the implementation, the
following are performed:

* unbind all `listenTo` events
* unbind all custom view events
* unbind all DOM events
* unbind all item views that were rendered
* remove `this.el` from the DOM
* call an `onClose` event on the view, if one is provided

By providing an `onClose` event in your view definition, you can
run custom code for your view that is fired after your view has been
closed and cleaned up. This lets you handle any additional clean up
code without having to override the `close` method.

```js
Backbone.Marionette.CollectionView.extend({
  onClose: function(){
    // custom cleanup or closing code, here
  }
});
```
