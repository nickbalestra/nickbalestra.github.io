---
title: Functions and the Processes They Generate
date: 2015-10-17
description: 'A workshop on recursion functions and processes' 
---

##The ability to visualize the consequences of the actions under consideration is crucial to becoming an expert programmer, just as it is in any synthetic, creative activity. ~ [_SICP_](https://mitpress.mit.edu/sicp/)

This post is my attempt to write about recursion. If you are familiar with SICP, you might have noticed that the title of this post mimic in fact the title of its 1.2 chapter. I will in fact follow parts of the text to talk about recursion in general, both in terms of functions and processes. We'll then explore some examples, define a general pattern and finally get our hands dirty with a toy problem. All in JavaScript.

* * *


## Linear Recursion and Iteration

Let's start by implementing a function that will allow us to calculate the sum of two numbers without relying on the `+` operator. Immagine we were to write a possible implementation of the `+` operator itself. 

For the sake of the example let's assume we have two functions that we can rely on: inc() wich increments its argument by 1 and dec(), which decrements its argument by 1:

{% highlight javascript linenos %}
function inc(n){
  return ++n;
}

function dec(n){
  return --n;
}
{% endhighlight %}

We could define the sum of two numbers as the increment of the sum of the first number decremented and the other number. That sounds more complex then it is... 
Let's see some code about this way to define sum in term of inc and dec:

{% highlight javascript linenos %}
function sum(a, b) {
  if (a == 0) {
    return b;
  }
  return inc(sum(dec(a), b));
}
{% endhighlight %}



#### Visualizing the process 

We can use the sobstitution model to watch this function in action computing `4 + 5` :

{% highlight javascript linenos %}
// A linear recursive process for computing 4 + 5

sum(4, 5)
inc(sum(3, 5))
inc(inc(sum(2, 5)))
inc(inc(inc(sum(1, 5))))
inc(inc(inc(inc(sum(0, 5)))))
inc(inc(inc(inc(5))))
inc(inc(inc(6)))
inc(inc(7))
inc(8)

// -> 9
{% endhighlight %}

Now let's take a different perspective on computing sum. We could describe sum by specifying that we first decrement A and incremen B and then sum them together, and so on until we reach 0 + 9. Therefore, another possible, but still valid, way to define sum in term of inc and dec could be:

{% highlight javascript linenos %}
function sum(a, b) {
  if (a == 0) {
    return b;
  }
  return sum(dec(a), inc(b));
}
{% endhighlight %}

#### Visualizing the process

As before we can use the sobstitution model to visualize the process of computing `4 + 5` :

{% highlight javascript linenos %}
// A linear iterative process for computing 4 + 5

sum(4, 5)
sum(3, 6)
sum(2, 7)
sum(1, 8)
sum(0, 9)

// -> 9
{% endhighlight %}

Compare the two process. For the first implementation the substitution model reveals a shape of expansion followed by contraction. The expansion occurs as the process builds up a series of deferred operations (in this case a chain of increments). The contraction occurs as the operations are actually performed. This type of preocess, is called recursive process. The interpreter need to keep track of the operations that will need to be performed. Such amount of information grows linearly as the amount of steps needed to complete the process grows, we refer to such process as linear recursive process.

By contrast the second process doesn't grow and shrink. At each step the interpreter only need to keep track of the current value of A and B. Such a process is called iterative process. Bacuse this process also grow linearly compared to the steps needed, we call it linear iterative process.

This contrast between the two processes affect space complexity, as the memory needed to be allocated for the first process, the linear recursive process, grow linearly at each step, while stay constant in the case of a linear iterative process. If it happen that your interpreter run out of memory you may encounter the following error message: 'Maximum call stack size exceeded', now you know why. 

By now you may be wondering why the word recursive only appear on the first process and not on the second, that we instead called iterative. We must be careful not to confuse the notion of recursive process with the notion of recursive function. When we describe a function as recursive we are referring to its syntax, or in other words, that the function definition refer to the function itself. When, instead, we describe a process to be, for example, linerly recursive, we are referring to how the process unfold and evolve over time, not how the syntax of the function that generates the process was written. 

####Tail-recursive processes in JavaScript

This distinctition between function and process may be confusing if you came from a language like JavaScript, as it's designed in a way that any recursive function will consume a growing amount of memory as the number of function calls grows, even when the process described may be iterative. As a consequence, JavaScript could describe iterative processes only by using 'looping constructs' like do/while and for. 

This is going to change with the new Ecmascript 6 specifications, where a so called tail-call optimzation has been added to the interpreter. This allows the interpreter to excecute an iterative process in constant space even if that iterative process is described by a recursive function. The tail-recursive implementation in ES6 allow to use the ordinary function invocation to express iterative processes so that special iteration constructs are useful only as syntactic sugar.

If you are familiar with functional libraries like lodash or underscore, you know how we can implement for example functions like each and reduce by relying on looping constructs. 
Following are two examples of a recursive implementation of each and reduce that take advantage of the upcoming ES6 tail-call optimization in order to express a liner iteration process.

#####Tail-recursive each (linear iterative process in ES6)

{% highlight javascript linenos %}
function each(list, iterator) {
  if (list.length === 0) {
    return undefined;
  }
  iterator(list[0]);
  return each(list.slice(1), iterator);
};
{% endhighlight %}

#####Tail-recursive reduce (linear iterative process in ES6)

{% highlight javascript linenos %}
function reduce(list, callback, accumulator) {
  if (list.length === 0) return accumulator;

  accumulator === undefined ? 
    accumulator = list[0] :
    accumulator = callback(accumulator, list[0]);
  
  return reduce(list.slice(1), callback, accumulator);
};
{% endhighlight %}

So with ES6 we can have linear iterative process in constant space
by calling a recursive function, thanks the toe tail-recursive implementation of the interpreter. To read more about tail-call optimization:

- [ES6 tail call optimization](http://benignbemine.github.io/2015/07/19/es6-tail-calls/)
- [EcmaScript Tail-position calls specs](http://www.ecma-international.org/ecma-262/6.0/#sec-tail-position-calls)
- [ES6 Tail-call optimization implementation status](https://kangax.github.io/compat-table/es6/#test-proper_tail_calls_(tail_call_optimisation))
<br>

* * *

## Tree Recursion

Tree recursion is another commong computational pattern. An often used example in this direction is computing the Fibonacci numbers, in which each number is the sum of the proceeding two:

> 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, ...

A recursive function to computing Fibonacci numbers, that will result in a tree recursive process:

{% highlight javascript linenos %}
function fib(n) {
  if (n < 2) {
    return n;
  }
  return fib(n - 1) + fib(n - 2);
};
{% endhighlight %}

#### Visualizing the process

Similary to what we did before we can use the sobstitution model to visualize the process of computing fib(5) :

{% highlight javascript linenos %}
// A tree recursive process for computing fib(5)

                                     fib(5)
                            /                    \
                     fib(4)                        fib(3)
                  /         \                    /        \  
            fib(3)           fib(2)           fib(2)       fib(1)
          /       \         /      \         /      \          |
       fib(2)     fib(1) fib(1)   fib(0)  fib(1) fib(0)        1
     /       \        |      |        |       |      |
fib(1)       fib(0)   1      1        0       1      0
    |            |
    1            0

// -> 5
{% endhighlight %}

While this function show us a typical tree recursion process is a terrible way to compute Fibonacci numbers, as it does a lot of rendundant calulculations. The process uses a number of steps that grows exponentially with the input. The space complexity on the other side only grows linearly because we need to keep track only of which nodes are above us in the tree at any point in the computation. In other words the space required is proportional to the maximum depth of the tree. 

If you read my post on [DepthFirst and breadthFirst tree walking](http://nick.balestra.ch/2015/depthFirst-vs-breadthFirst/) you may notice that a Depth-First Search algorithm rely on a Stack datastructure. I'll leave it to the reader to find any parallelism with the call stack used by the interpreter to keep track of the deferred operations and why the space required in such a tree recursion is proportional to the maximum depth of the tree...

We can also define a recursive function to compute the Fibonacci numbers that will instead generate a linear iterative process (as in ES6, thanks to its tail-call optimization):

{% highlight javascript linenos %}
var fib = (n, a = 0, b = 1) => (n === 0) ? a : fib(n-1, b, a + b);
{% endhighlight %}

Now that we saw different recursive processes we may start noticing some pattern emerging in how we define those recursive function responsibles for such processes.

* * *

## Anathomy of a recursive process

A recursive behaviour is when you can define it by two properties:

- **The Base Case**: The terminating scenario that does not need to recurse anymore in order to produce an answer.
- **A set of rules** that reduce all the other cases toward the base case.

Let's mark with some comments all the base cases from our previous examples:

{% highlight javascript linenos %}
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
};

function reduce(list, callback, accumulator) {
  if (list.length === 0) return accumulator; // <- BASE CASE

  accumulator === undefined ? 
    accumulator = list[0] :
    accumulator = callback(accumulator, list[0]);
  
  return reduce(list.slice(1), callback, accumulator);
};

function fib(n) {
  if (n < 2) { // <- BASE CASE
    return n;
  }
  return fib(n - 1) + fib(n - 2);
};

var fib = (n, a = 0, b = 1) => (n === 0) ? a : // <- BASE CASE
fib(n-1, b, a + b);
{% endhighlight %}

An alternative form to remember this base case is the following quote from Andrew Plotkin: 
> If you already know what recursion is, just remember the answer. Otherwise, find someone who is standing closer to Douglas Hofstadter than you are; then ask him or her what recursion is.

* * *

##  Finding land on Civilization 3

To conclude this workshop on recursion, I would like to bring in a Toy Problem where we are going to traverse a Matrix, so that you can get your hands dirty with some recursions.

Let's immagine we have a matrix, representing the map of the virtual world we are going to play on in [Civilization 3](https://en.wikipedia.org/wiki/Civilization_III).

We want to start our game on the vastest territory possible to increase the fun. 
To do that we'll need to define a function `continentCounter` that accept a world matrix, an X and Y cordinate as parameters. ContinentCounter returns the number of 'tiles' that the land you are standing on in a specific place with coordinate x and y is comprised of. 
Land is defined as the continuos walkable surface not crossed by water.

### prompt:

- Implement continentCounter() using a recursive process.
- Identify which kind of recursive process your implementation of continentCounter() generates.
<br><br>
{% highlight javascript linenos %}

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





