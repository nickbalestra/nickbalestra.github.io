---
title: 'Good Old OOPrototypal'
date: 2015-15-10
description:
---

##Any construct that is capable of producing a fleet of similar instances conforming to some interface can be called a Class.

Without entering into the debate about the OOp aspects of the javaScript language specific to Classes, and if classes/instances truely exists in js, let's just rely on the above definition, and explore the the new syntactic sugar coming up in ES6 compared to the Pseudcoclassical Subclass Pattern.

* * *

### In general

/*
// Class
var Car = function(brand){
  this.brand = brand;
}

Car.prototype.horn = function(){return 'honk'};


// subClass
var Tesla = function(){
  // Extend the constructor class with the Car constructor properies
  // Remember that using the new contructor invocation keywoard on a function will
  // 1: this = Object.create(Tesla.prototype)
  Car.call(this, 'Tesla'); // -> this properties will be copied over for every instance of a class
  // 2: return this;

};
// We want to 
// 1 - create a new empty object, that 
// 2 - will delegate to the Car prototype and 
// 3- assign it to the Tesla prototype
Tesla.prototype = Object.create(Car.prototype);
// We also need to set the contructur property of te prototype to point back to Car
Tesla.prototype.constructor = Car;

// We can now add shared propertied to any instance of our class,
// by adding them to the prototype object
Tesla.prototype.ludacrisSpeed = function(){return true}; // those properties are shared, following the pseudoClassical Pattern


// --
var golf = new Car('wv');
console.log(golf.brand); // -> vw
console.log(golf.hasOwnProperty('brand'));
console.log(golf.hasOwnProperty('horn'));
console.log(golf.horn()); // -> honk
console.log(golf.hasOwnProperty('ludacrisSpeed'));
// console.log(golf.ludacrisSpeed()) // Err (not a function)


var modelS = new Tesla();
console.log(modelS.brand) // -> Tesla
console.log(golf.hasOwnProperty('brand'));
console.log(golf.hasOwnProperty('horn'));
console.log(modelS.horn()) // -> honk
console.log(golf.hasOwnProperty('ludacrisSpeed'));
console.log(modelS.ludacrisSpeed()) // -> true
*/


// let see how this will work in ES6:

class Car {
    constructor(brand) {
        this.brand = brand;
    }

    horn() {
        return 'honk';
    }
}


class Tesla extends Car {
    constructor() {
        super('Tesla'); //call the parent method with super
    }

    ludacrisSpeed() {
      return true;
    }
}

var golf = new Car('wv');
console.log(golf.brand); // -> vw
console.log(golf.hasOwnProperty('brand'));
console.log(golf.hasOwnProperty('horn'));
console.log(golf.horn()); // -> honk
console.log(golf.hasOwnProperty('ludacrisSpeed'));

var modelS = new Tesla();
console.log(modelS.brand) // -> Tesla
console.log(modelS.hasOwnProperty('brand'));
console.log(modelS.hasOwnProperty('horn'));
console.log(modelS.horn()) // -> honk
console.log(modelS.hasOwnProperty('ludacrisSpeed'));
console.log(modelS.ludacrisSpeed()) // -> true

console.log(modelS.__proto__ == golf.__proto__);//false
console.log(modelS.__proto__.__proto__ == golf.__proto__);//true

// modelS.__proto__.__proto__.horn = function(){return 'LOUDER HONK'};
Car.prototype.horn = function(){return 'LOUDER HONK'};
console.log(golf.horn()); // -> LOUDER HOONK
console.log(modelS.horn()) // -> honk





resources:
http://stackoverflow.com/questions/9959727/proto-vs-prototype-in-javascript
https://reinteractive.net/posts/235-es6-classes-and-javascript-prototypes