---
title: 'ES6: Arrows, array-like objects and defaults'
date: 2015-10-05
description: Taking advantage of some of the new ES6 features.
---

## ECMAScript 2015 (6th Edition), commonly referred to as "ES6", is the current version of the ECMAScript Language Specification standard.

In this post I go through leveraging some of its new feature to rewrite a recursive implementation of the document API method getElementsByClassName().  

* * *

getElementsByClassName() returns an array-like object of all child elements which have all of the given class names. When called on the document object, the complete document is searched, including the root node. You may also call getElementsByClassName() on any element; it will return only elements which are descendants of the specified root element with the given class names.

A recursive implementation could look something like this:

{% highlight javascript linenos %}
var getElementsByClassName = function(className, node){
  
  node = node || document.body;
  var results = [];

  if (node.classList.contains(className)) {
    results.push(node);
  }
  _.each(node.childNodes, function(node) {
    results = results.concat(getElementsByClassName(className, node));
  });

  return results;
};
{% endhighlight %}

Let's see how ES6 could come handy:

## Arrows functions

Arrow functions are always anonymous. Also known as fat arrow function, have a shorter syntax compared to function expressions and lexically binds the this value. Let's refactor the previous version using Arrow functions expressions instead:

{% highlight javascript linenos %}
var getElementsByClassName = (className, node) => {
  
  node = node || document.body;
  var results = [];

  if (node.classList.contains(className)) {
    results.push(node);
  }
  _.each(node.childNodes, node => {
    results = results.concat(getElementsByClassName(className, node));
  });

  return results;
};
{% endhighlight %}

Not much but we shaved off some characters anyway.


## Array.from()

We are all familiar with those array-like objects, especially if we deal with the DOM. The Array.from() method creates a new Array instance from an array-like or iterable object. Forgot about those? Here a refresher:

{% highlight javascript linenos %}
Array.isArray(document.body.childNodes);
// > false
{% endhighlight %}

This is why in our earlier implementation we relied on underscore (_.each()) to iterate through that array-like object. Lets refactor again and get rid of that underscore dependency:


{% highlight javascript linenos %}
var getElementsByClassName = (className, node) => {
  
  node = node || document.body;
  var results = [];

  if (node.classList.contains(className)) {
    results.push(node);
  }
  
  Array.from(node.children).forEach(child => {
    elements = elements.concat(getElementsByClassName(className, child))
  });

  return results;
};
{% endhighlight %}

## Defaults

In javascript we all rely on short-circuiting the OR operator to assign some default values to the function arguments, in case those were not definied. ES6 come with syntax sugar for this, allowing to set defaults directly within the function signature:

{% highlight javascript linenos %}
var getElementsByClassName = (className,  node = document.body) => {
  var results = [];

  if (node.classList.contains(className)) {
    results.push(node);
  }
    
  Array.from(node.children).forEach(child => {
    results = results.concat(getElementsByClassName(className, child))
  });
  
  return results;
};
{% endhighlight %}

To run and test ES6 code on chrome make sure you switch on the harmony setting. To do so, just enter in your chrome address bar the folloing url: [chrome://flags/#enable-javascript-harmony](chrome://flags/#enable-javascript-harmony).

**Note** that defaults are not yet supported in chrome, therefore you won't be able to fully test this ES6 refactored solution out of the box. Because of that, I strongly encourage you to use the [babel](http://babeljs.io) transpiler.



