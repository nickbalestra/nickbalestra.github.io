---
title: "JS's Prototype-based inheritance and classes demystified"
date: 2015-10-15
description: Understanding what's lie underneath the ES6 sugar
---

##JavaScript classes are introduced in ECMAScript 6 and are syntactical sugar over JavaScript's existing prototype-based inheritance. 

Note: The class syntax is not introducing a new object-oriented inheritance model to JavaScript, so what is happening behind all this new sugar? Let's digg how ES6 classes works compared to the pseudcoclassical subclass pattern we are used to work with within ES5.

* * *

### In ECMAScript 5

Let's immagine a simple scenario, where we have a `Car` class. We want the constructor function to decorate each generated instance, for example by adding a `brand` property to the car. We also want each instanciated object to have access to some class methods, for example a horn() function.

{% highlight javascript linenos %}
// Constructor function for the Car class.
var Car = function(brand){
  this.brand = brand;
};

Car.prototype.horn = function(){return 'honk'};
{% endhighlight %}

Now, let's add a subClass and let's call it Tesla (Being it a truely new type of car, and therefore a subClass of `Car`). Of course, we want each Tesla to be branded 'tesla' as well, but we also want it to have some unique features, like for example the ludacrisSpeed function.

{% highlight javascript linenos %}
// Constructor function for the Tesla subClass.
var Tesla = function(){
  // Decorates subClass istances with Car properties
  // Note: using the contructor call 'new' 
  // when invoking a function will result in:
  // this = Object.create(Tesla.prototype)
  // Allowing to extend this with the properties we 
  // definied wihin the Car constructor:
  Car.call(this, 'Tesla');
  // and finally return this;
};

// To finish the subclass constructor, we want to make
// sure that the prototypal chain delegates correctly
// to the Car prototype. For that we need to:
// 1 - create a new empty object, that 
// 2 - will delegate to the Car prototype and 
// 3- assign it to the Tesla prototype
Tesla.prototype = Object.create(Car.prototype);
// We also need to set the constructor property of
// the prototype to point back to the Car constructor.
Tesla.prototype.constructor = Car;

// We can now add shared methods to any Tesla instance,
// by adding them to the prototype object following 
// the pseudoClassical Pattern: 
Tesla.prototype.ludacrisSpeed = function(){return true}; 
{% endhighlight %}

Without all the comments the Tesla subclass constructor should look something like:

{% highlight javascript linenos %}
// Constructor function for the Tesla subClass.
var Tesla = function(){
  Car.call(this, 'tesla');
};

Tesla.prototype = Object.create(Car.prototype);
Tesla.prototype.constructor = Car;

Tesla.prototype.ludacrisSpeed = function(){return true}; 
{% endhighlight %}

Running some tests, to make sure that everything work as expected:

{% highlight javascript linenos %}
var golf = new Car('wv');

golf.brand; // -> vw
golf.hasOwnProperty('brand'); // -> true
golf.hasOwnProperty('horn'); // -> false
golf.horn(); // -> honk
golf.hasOwnProperty('ludacrisSpeed'); // -> false
golf.ludacrisSpeed(); // throw err 


var modelS = new Tesla();

modelS.brand; // -> tesla
modelS.hasOwnProperty('brand'); // -> true
modelS.hasOwnProperty('horn'); // -> false
modelS.horn(); // -> honk
modelS.hasOwnProperty('ludacrisSpeed'); // -> false
modelS.ludacrisSpeed() // -> true
{% endhighlight %}

### In ECMAScript 6

Now that we did brush up on the pseudoClassical subclass pattern, let see how the above code will look like in ES6 using the new Syntactic sugar for classes. Refer to the code above to see what is going on under the hood, it should now be pretty self explanatory.

{% highlight javascript linenos %}
// Class Car
class Car {
  constructor(brand) {
    this.brand = brand;
  }

  horn() {
    return 'honk';
  }
}

// SubClass Tesla
class Tesla extends Car {
  constructor() {
    super('Tesla');
  }

  ludacrisSpeed() {
    return true;
  }
}
{% endhighlight %}

Running some tests, to make sure that everything work as expected:

{% highlight javascript linenos %}
var golf = new Car('wv');

console.log(golf.brand); // -> vw
console.log(golf.hasOwnProperty('brand'));
console.log(golf.hasOwnProperty('horn'));
console.log(golf.horn()); // -> honk
console.log(golf.hasOwnProperty('ludacrisSpeed'));


var modelS = new Tesla();

modelS.brand; // -> tesla
modelS.hasOwnProperty('brand'); // -> true
modelS.hasOwnProperty('horn'); // -> false
modelS.horn(); // -> honk
modelS.hasOwnProperty('ludacrisSpeed'); // false
modelS.ludacrisSpeed(); // -> true

modelS.__proto__ == golf.__proto__ // -> false
modelS.__proto__.__proto__ == golf.__proto__ // -> true

Car.prototype.horn = function(){return 'LOUDER HONK'};
golf.horn(); // -> LOUDER HONK
modelS.horn(); // -> LOUDER HONK
{% endhighlight %}

To run and test ES6 code on chrome make sure you switch on the harmony setting. To do so, just enter in your chrome address bar the folloing url: [chrome://flags/#enable-javascript-harmony](chrome://flags/#enable-javascript-harmony).

**Note** class and super are not yet supported in chrome, therefore you won't be able to fully test this ES6 out of the box. I strongly encourage you to use the [babel](http://babeljs.io) transpiler, or play with its [online REPL](https://babeljs.io/repl/)

