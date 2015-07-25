---
title: Binding Backbone models & collections to Firebase
date: 2015-07-24
description: Syncing Backbone apps, effortless.
---

##[Backbone.js](http://backbonejs.org/) gives structure to web applications by providing models with key-value binding and custom events, collections with a rich API of enumerable functions, views with declarative event handling, and connects it all to your existing API over a RESTful JSON interface.

##[Firebase](https://www.firebase.com/) is a PaaS for building mobile and web applications. It provides a realtime JSON database for storing and sync your app's data.

Let's explore how Backbone can seamlessly integrate with Firebase, making clients applications syncing effortless.

***

### A little colorful app

![FireBone Colors](https://raw.githubusercontent.com/nickbalestra/nickbalestra.github.io/master/assets/images/firebone.png)

To better illustrate through code how easy it can be to bind Backbones models or collections to Firebase and therefore allow for syncronizing data with it, let's meet fireBone.
Firebone is a little application, built with backbone.js. The only thing that it does is generating random colors; let's see how:


The application architecture is composed of a model, a collection and a view.

### Color model
The color model is just a stripped Backbone.Model. This will serve as a constructor to later instantiate all of our colors.

{% highlight javascript linenos %}
var Color = Backbone.Model.extend({});
{% endhighlight %}

### Colors collection
Here is where all the action will happen. Our application will allow us to add colors to a colors collection. The collection will act similarly to a stack: we'll be adding colors on top of the stack and removing from the top of it, following a FILO (First In Last Out) approach.

{% highlight javascript linenos %}
var Colors = Backbone.Collection.extend({
  model: Color,
});
{% endhighlight %}

### App view
Inside the view we defined the template logic together with its rendering. For the sake of this post, we used a simple underscore template and some jquery to handle the rendering.

In an events hash we specify a set of DOM events that we will bound to relative methods (addColor, removeColor and resetColors).

On initialization, when our view will be instantiated, we set up a listener to listen on add, remove and reset events on our collection, triggering the render method when they fire.

Our methods operate on the backbone collection.

addColor() will instantiate a color using the Color model as a constructor. We'll rely on the [pleasejs](https://github.com/Fooidge/PleaseJS) library to generate random colors storing the hexadecimal values as the Color.hexColor property. removeColor() will just pop a color from the collection, while resetColors() will reset the whole collection.

The code of our view:

{% highlight javascript linenos %}
var AppView = Backbone.View.extend({
  template: _.template('<div class="row" style="background-color:<%= hexColor %>"><span><%= hexColor %></span></div>'),

  events: {
    'click #add': 'addColor',
    'click #remove': 'removeColor',
    'click #reset': 'resetColors'
  },

  initialize: function(){
    this.collection.on('add remove reset', this.render, this);
  },

  render: function(){
    $view = this.$el.html('<div class="addColor"><span id="add">Gimme moar</span><span id="remove">Gimme less</span><span id="reset">Gimme me none</span></div><div id="colors"></div>');

    this.collection.each(function(color){
      $view.find('#colors').prepend(this.template(color.attributes));
    }, this);

    return $view;
  },

  addColor: function(){
    var randomHex = Please.make_color({base_color: 'cyan'});
    this.collection.add({hexColor: randomHex});
  },

  removeColor: function(){
    this.collection.pop();
  },

  resetColors: function(){
    this.collection.reset();
  }
});
{% endhighlight %}

* * *

##The Firebase binding
So, wasn't this post about binding backbone models and collections to Firebase?  Right, to do that we just need two things:

###Dependencies
First, let's add the right dependencies and load the firebase and backbonefire js libraries in our index.html.

{% highlight javascript linenos %}
$ bower install backbonefire --save
{% endhighlight %}

{% highlight javascript linenos %}
<script src="bower_components/firebase/firebase.js"></script>
<script src="bower_components/backbonefire/dist/backbonefire.js"></script>
{% endhighlight %}

###Binding backbone model/collections

Once we have sorted our dependencies and the needed libraries are in place, we can now access some special models and collections: Backbone.Firebase.Collection and Backbone.Firebase.Model. As for any backbone model and collection they support a url property (Add there your Firebase-app url). Our code for the collection code should now look something like:

{% highlight javascript linenos %}
var Colors = Backbone.Firebase.Collection.extend({
  url: 'https://<YOUR_FIREBASE_APP>.firebaseio.com/colors',
  model: Color,
});
{% endhighlight %}

Guess what? We are done, refresh your app and Boom, our Backbone collection is now synced in RT among all the clients running and your Firebase database. Neat!

Our little app running the Firebase binding on the colors collection:

![FireBone Colors](https://raw.githubusercontent.com/nickbalestra/nickbalestra.github.io/master/assets/images/firebone.gif)

* * *

## Final thoughts and resources.

Be aware that a Backbone.Firebase.Model should not be used with a Backbone.Firebase.Collection. Use a regular Backbone.Model with a Backbone.Firebase.Collection instead.

Personally I really like Backbone, its way of of MVCing, and its unopinionated approach. Binding with Firebase just opens up endless possibilities.

Download the [Backbone binding for Firebase](https://github.com/firebase/backbonefire)<br>
Fork the [FireBone Colors App](https://github.com/nickbalestra/fireBone)
