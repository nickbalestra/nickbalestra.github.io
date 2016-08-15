---
title: "Recursion, part 4: Anatomy of a recursion"
date: 2015-10-17
description: 'A workshop on recursion functions and processes'
---


This post is part of a series of posts in which I write about recursion. I'll Borrow some concepts from the [SICP](https://mitpress.mit.edu/sicp/) to talk about recursion in general, both in terms of functions and processes. I'll then explore some examples, define a general pattern and finally we'll get our hands dirty with a toy problem. All in JavaScript.

* * *

## Anatomy of a recursion

A recursive behaviour is when you can define it by two properties:

- **The Base Case**: The terminating scenario that does not need to recurse anymore in order to produce an answer.
- **A set of rules** that reduce all the other cases toward the base case.

Let's mark with some comments all the base cases from our previous examples to make this even more clear:

{% highlight javascript %}
function sum(a, b) {
  if (a == 0) { // <- BASE CASE
    return b;
  }
  return inc(sum(dec(a), b));
}

function sum(a, b) {
  if (a == 0) { // <- BASE CASE
    return b;
  }
  return sum(dec(a), inc(b));
}

function each(list, iterator) {
  if (list.length === 0) { // <- BASE CASE
    return undefined;
  }
  iterator(list[0]);
  return each(list.slice(1), iterator);
}

function reduce(list, callback, accumulator) {
  if (list.length === 0) return accumulator; // <- BASE CASE

  accumulator === undefined ?
    accumulator = list[0] :
    accumulator = callback(accumulator, list[0]);

  return reduce(list.slice(1), callback, accumulator);
}

function fib(n) {
  if (n < 2) { // <- BASE CASE
    return n;
  }
  return fib(n - 1) + fib(n - 2);
}

var fib = (n, a = 0, b = 1) => (n === 0) ? a : // <- BASE CASE
fib(n-1, b, a + b);
{% endhighlight %}

As a note, is nice to mention the funny quote from Andrew Plotkin about recursions:
> If you already know what recursion is, just remember the answer. Otherwise, find someone who is standing closer to Douglas Hofstadter than you are; then ask him or her what recursion is.

* * *

##  Finding land on Civilization 3

To conclude this workshop on recursion, I would like to bring in a Toy Problem about matrix traversal, so that you can get your hands dirty with some recursions.

Let's immagine we have a matrix, representing the map of the virtual world we are going to play on in [Civilization 3](https://en.wikipedia.org/wiki/Civilization_III).

We want to start our game on the vastest territory possible to increase the fun, and be able in to just quite and restart a new game in case the territory we were radnomly given at the beginning of our game is too small to have some reasonable amount of fun within it.
To do that we'll need to define a function `continentCounter` that accept a world matrix, and xy coordinates as parameters. ContinentCounter will returns the number of 'tiles' that the land on the given coordinates x and y is comprised of, or 0 in case of water.
Land is defined as the continuos walkable surface not crossed by water.

### prompt:

- Implement continentCounter() using a recursive process.
- Identify which kind of recursive process your implementation of continentCounter() generates.
<br><br>
{% highlight javascript %}

var o = "water"; // water
var M = "land";  // land

var world = [
  [o,o,o,o,M,o,o,o,o,o],
  [o,o,o,M,M,o,o,o,o,o],
  [o,o,o,o,M,o,o,M,M,o],
  [o,o,M,o,M,o,o,o,M,o],
  [o,o,o,o,M,M,o,o,o,o],
  [o,o,o,M,M,M,M,o,o,o],
  [M,M,M,M,M,M,M,M,M,M],
  [o,o,M,M,o,M,M,M,o,o],
  [o,M,o,o,o,M,M,o,o,o],
  [M,o,o,o,M,M,o,o,o,o]
];

function continentCounter (world, x, y) {
  // YOUR CODE HERE
}

// Test
// continentCounter(world, 0, 4); // -> 32
// continentCounter(world, 3, 2); // -> 1
// continentCounter(world, 0, 0); // -> 0
{% endhighlight %}

 ***

## Final Notes

- The Civilization problem was given to me during a workshop on recursion that I attended as part of the Advanced Software Engineering Immersive program taught at [Hackreactor](http://www.hackreactor.com/). The worskhopp was given by the awesome [Ilona Budapesti](https://twitter.com/ilonabudapesti), and the problem has been taken from Chris Pine's great introductory Ruby book and ported to JS by her. You can find the solution to the problem [here](https://gist.github.com/nickbalestra/ebe0a86271c55d99f4f2).

- The recursive processes are inspired by the [Wizard Book](https://mitpress.mit.edu/sicp/): Hal Abelson's, Jerry Sussman's and Julie Sussman's Structure and Interpretation of Computer Programs (MIT Press, 1984; ISBN 0-262-01077-1), an excellent computer science text used in introductory courses at MIT. So called because of the wizard on the jacket. One of the bibles of the LISP/Scheme world. Also, less commonly, known as the Purple Book.

- Ecmascript 6 Tail call optmization are explained very well on Kyle Owen's post "[S6 Tail Call Optimization Explained](http://benignbemine.github.io/2015/07/19/es6-tail-calls/)", an awesome HR alumnus that also happen to read and appreciate the purple book.

* * *

####All the recursion workshop articles:

- [Recursion, part 1: Linear Recursion and Iteration](http://nick.balestra.ch/2015/recursion-workshop)
- [Recursion, part 2: Tail-recursive processes in JS](http://nick.balestra.ch/2015/recursion-workshop-part2/)
- [Recursion, part 3: Tree recursion](http://nick.balestra.ch/2015/recursion-workshop-part3/)
- [Recursion, part 4: Anatomy of a recursion](http://nick.balestra.ch/2015/recursion-workshop-part4/)
