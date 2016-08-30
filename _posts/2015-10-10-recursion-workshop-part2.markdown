---
title: "Recursion, part 2: Tail-recursive processes in JS"
date: 2015-10-17
description: 'A workshop on recursion functions and processes'
---


This post is part of a series of posts in which I write about recursion. I'll Borrow some concepts from the [SICP](https://mitpress.mit.edu/sicp/) to talk about recursion in general, both in terms of functions and processes. I'll then explore some examples, define a general pattern and finally we'll get our hands dirty with a toy problem. All in JavaScript.

* * *

## Tail-recursive processes in JavaScript

This distinctition between function and process may be confusing if you came from a language like JavaScript, as JS is among those languages designed in a way that any recursive function will consume a growing amount of memory as the number of function calls grows, even when the process described may be iterative. As a consequence, JavaScript could describe iterative processes only by using 'looping constructs' like do/while and for.

This is going to change with the new Ecmascript 6 specifications, where a so called tail-call optimzation has been added to the interpreter. This allows the interpreter to excecute an iterative process in constant space even if that iterative process is described by a recursive function. The tail-recursive implementation in ES6 allow to use the ordinary function invocation to express iterative processes so that special iteration constructs are useful only as syntactic sugar.

If you are familiar with functional libraries like lodash or underscore, you know how we can implement functions like each and reduce by relying on looping constructs.
Following are two examples of a recursive implementation of each and reduce that take advantage of the upcoming ES6 tail-call optimization in order to express a liner iteration process.

#####Tail-recursive each (linear iterative process in ES6)

{% highlight javascript %}
function each(list, iterator) {
  if (list.length === 0) {
    return undefined;
  }
  iterator(list[0]);
  return each(list.slice(1), iterator);
}
{% endhighlight %}

#####Tail-recursive reduce (linear iterative process in ES6)

{% highlight javascript %}
function reduce(list, callback, accumulator) {
  if (list.length === 0) return accumulator;

  accumulator === undefined ?
    accumulator = list[0] :
    accumulator = callback(accumulator, list[0]);

  return reduce(list.slice(1), callback, accumulator);
}
{% endhighlight %}

Boom! With ES6 we can have linear iterative process in constant space
by calling a recursive function, thanks the tail-recursive implementation of the interpreter. Read more about tail-call optimization:

- [ES6 tail call optimization](http://benignbemine.github.io/2015/07/19/es6-tail-calls/)
- [EcmaScript Tail-position calls specs](http://www.ecma-international.org/ecma-262/6.0/#sec-tail-position-calls)
- [ES6 Tail-call optimization implementation status](https://kangax.github.io/compat-table/es6/#test-proper_tail_calls_(tail_call_optimisation))
<br>


* * *

####All the recursion workshop articles:

- [Recursion, part 1: Linear Recursion and Iteration](http://nick.balestra.ch/2015/recursion-workshop)
- [Recursion, part 2: Tail-recursive processes in JS](http://nick.balestra.ch/2015/recursion-workshop-part2/)
- [Recursion, part 3: Tree recursion](http://nick.balestra.ch/2015/recursion-workshop-part3/)
- [Recursion, part 4: Anatomy of a recursion](http://nick.balestra.ch/2015/recursion-workshop-part4/)
