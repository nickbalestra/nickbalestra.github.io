---
title: Classes and Instantiation Patterns in javaScript
date: 2015-07-05
description: Functional, Functional-shared, Prototypal and Pseudoclassical
---

##Any construct that is capable of producing a fleet of similar instances conforming to some interface can be called a Class.

Without entering into the debate about the OOp aspects of the javaScript language specific to Classes, and if classes/instances truely exists in js, let's just rely on the above definition, and explore the different patterns available to us for defining Classes and instantiating them.

* * *

### In general

Every pattern need to take care of the basics:

- Create a new object
- Assign some properties and methods to it
- Return the new decorated object
<br>

* * *

### The Functional pattern

It does just what is suppose to do:

{% highlight javascript linenos %}
var FunctionalPattern = function(){
  // Create a new object
  var instance = {};

  // Add methods to it
  instance.aMethod = function(){};
  instance.anotherMethod = function(){};

  // Return the new decorated object
  return instance;
};

// Instantiation Pattern:
var instance = FunctionalPattern();
{% endhighlight %}



#### Pros & Cons

The functional pattern have the advantage of returning methods that were defined within the constructor's lexical scope, therefore creating closures (and making the object constrution pretty clear).
The very same principle that give the functional pattern the closure advantage also define its drawback. As the methods are declared within the the lexical scope of the constructor, instantiating a class via the functional pattern will result in duplicating all its methods, meaning new functions stored in new spots in memory.

* * *

### The Functional-shared pattern

Born as an effort to limit the drawbacks of the functional pattern, it moves the methods declaration outside the lexical scope of the class, extending it methods by reference. In this way each istance will have pointers to the same functions without duplicating them:

{% highlight javascript linenos %}
var FunctionalSharedPattern = function(){
  // Create a new object
  var instance = {};

  // Add methods to it
  instance.aMethod = functionalSharedMethods.aMethod;
  instance.anotherMethod = functionalSharedMethods.anotherMethod;

  // Return the new decorated object
  return instance;
};

// shared methods where each instance will point to
var functionalSharedMethods = {
  aMethod: function(){},
  anotherMethod: function(){},
};

// Instantiation Pattern:
var instance = FunctionalSharedPattern();
{% endhighlight %}

...definitely a WET solution.

> WET: Write Everything Twice *or* We Enjoy Typing

With the help of a little helper we could refactor the functional-shared pattern to be a little more DRY instead

> DRY: Don't Reapeat Yourself

{% highlight javascript linenos %}
var FunctionalSharedPattern = function(){
  // Create a new object
  var instance = {};

  // Add methods to it
  extend(instance, functionalSharedMethods);

  // Return the new decorated object
  return instance;
};

// shared methods where each instance will point to
var functionalSharedMethods = {
  aMethod: function(){},
  anotherMethod: function(){},
};

// Basic Extend Helper
function extend(instance, methods) {
  for (var key in methods) {
    if(!instance.hasOwnProperty(key))
    instance[key] = methods[key];
  }
}

// Instantiation Pattern:
var instance = FunctionalSharedPattern();
{% endhighlight %}

#### Pros & Cons

The functional-shared pattern solve the cons we saw in the functional pattern.
But, it still have a big drawback: the pointers in each instance are created during the instantiation only, meaning that if we add or edit shared methods those won't be available to our instances unless we update them (i.e. by invoking the extend helper again on each of our instances.).

* * *

### The Prototypal pattern

By now you probably noticing a trend in how this blog post is structured, we are building up a better pattern to solve the drawbacks of the previous one. And this is where the Prototypal pattern come handy. By relying on the prototypal delegation on which javaScript objects obey, it provide a handy way to keep in sync any istance with its class. Long story short: the prototypal pattern allow us to edit, add and remove methods on the Class and automagically have each istance updated accordingly.

{% highlight javascript linenos %}
var PrototypalPattern = function(){
  // Create a new object
  var instance = Object.create(PrototypalPattern.prototype);

  // Methods will be added to the prototype

  // Return the new decorated object
  return instance;
};

// PrototypalPattern will delegate to its prototype for methods lookup
PrototypalPattern.prototype.aMethod = function(){};
PrototypalPattern.prototype.anotherMethod = function(){};

// Instantiation Pattern:
var instance = PrototypalPattern();
{% endhighlight %}

#### Pros & Cons

There are'n much drawbacks with this Pattern, as it solve all our issues we encountered sofar. So, maybe there is not a better way, but just a sweeter way...

* * *

### Pseudoclassical
 Sometime an advantage could be given by the language itself and its syntactic sugar, and guess what, mr. Pseudoclassical pattern is very very sweet:

{% highlight javascript linenos %}
var PseudoclassicalPattern = function(){
  // Create a new object
  // var this = Object.create(PseudoclassicalPattern.prototype)
  // is automatically done by the interpreter when the function is
  // called with the constructor call

  this.aMethod = function(){};
  this.anotherMethod = function(){};
}

// Instantiation Pattern:
var instance = new PseudoclassicalPattern();
{% endhighlight %}

* * *

## Final thoughts and performance
The clear question we should all ask our self is: which pattern should we use? As for everything the answer is: it depends. It depends on the codebase you are working on and which pattern is most used there, in that case will be a good idea to stick to that. For example the Pseudoclassical is the preferred pattern in MVC’s such as backbone. Another interesting aspect to take into consideration when deciding which pattern to use could be related to performance, but this is a subject for a another blog post...

**Thanks to**:<br>
[Hack Reactor](http://www.hackreactor.com/) for being awesome.<br>
[Charles Crame](https://twitter.com/cpcrame) for helping in proof reading this post.<br>
[Ryan Atkinson](http://www.ryanatkinson.io/author/ryan/) for his inspirational blog.<br>
