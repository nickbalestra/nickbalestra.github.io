---
title: JS Functional Programming Libraries
date: 2015-06-13
description: From underscore to lodash and Ramda, from chaining to composition and auto-currying
---

##In computer science, Functional Programming (“FP”) is a programming paradigm—a style of building the structure and elements of computer programs—that treats computation as the evaluation of mathematical functions and avoids changing-state and mutable data.

Shortly, FP is a coding style based on the following two best practices, that together form the concept of fuction's purity:

1. **Immutability** and
2. **Statelessness**
<br><br>

In JavaScript, we have plenty of libraries with a functional flavour to choose from. While each has it's own set of goals, in the following paragraphs we'll see which are the ones we should all know about and why.

***

## Underscore/lodash

> [Underscore.js](http://underscorejs.org/) is a utility-belt library for JavaScript that provides support for the usual functional suspects (each, map, reduce, filter...) without extending any core JavaScript objects.

> [Lodash](https://lodash.com/) is a JavaScript utility library delivering consistency, modularity, performance, & extras.

### Why should we care

As of today, **lodash is the most dependended-upon package on npm** while **underscore is the third most dependended-upon**.

[Lodash](https://www.npmjs.com/package/lodash) in numbers (impressive):

> - 500,000+ daily downloads
> - 13,000,000+ monthly downloads
> - [11,300+](https://www.npmjs.com/browse/depended/lodash) depended-upon packages

<br>
[Underscore](https://www.npmjs.com/package/underscore) in numbers (still pretty impressive):

> - 300,000+ daily downloads
> - 6,900,000+ monthly downloads
> - [9,400+](https://www.npmjs.com/browse/depended/underscore) depended-upon packages

<span></span>

### Things worth knowing about underscore
#### Chaining
Underscore come with a chain method. It allow us to use _.chain() to wrap a collection and then call other underscore functions on it before finally calling _.value() to unwrap the final result.

Taking a movies collection :

{% highlight javascript linenos %}
var movies = [
  { title: "Mad Max: Fury Road", ratings: [8.9, 9.9, 7.9], year: 2015 },
  { title: "Tomorrowland", ratings: [6.0, 7.0, 8.0], year: 2015 },
  { title: "Jurassic World", ratings: [7.2, 8.2, 9.2], year: 2015 },
  { title: "Inception", ratings: [7.5, 7.5, 8.5], year: 2010 }
];
{% endhighlight %}

and some basic helpers

{% highlight javascript linenos %}
var sum = function(numbers) {
    return _.reduce(numbers, function(sum, n){
        return sum + n }, 0);
};

var average = function (numbers) {
    return sum(numbers) / numbers.length;
};

var isRecommended = function (movie) {
    return average(movie.ratings) >= 8;
};
{% endhighlight %}

We can now create a complex operation by simply chaining together underscore functions. For example, to discover the lowest rating ever given to a recomended movie:

{% highlight javascript linenos %}
_.chain(movies)
  .filter(isRecommended)
  .pluck('ratings')
  .flatten()
  .min()
  .value()
// → 7.2
{% endhighlight %}

#### Composition

If \_.chain seems well known and commonly used, fewer may be aware of an underscore hidden gem: compose. Compose is as close as we can get to function composition within underscore, by allowing us to returns the composition of a list of functions, where each function consumes the return value of the function that follows.

In math terms, composing the functions f(), g(), and h() produces f(g(h())).

Given some generic functions

{% highlight javascript linenos %}
var greet = function(name){
    return "hi: " + name;
};

var exclaim  = function(statement){
    return statement.toUpperCase() + "!";
};
{% endhighlight %}

We can now compose them in order to create a more powerful function out of the two:

{% highlight javascript linenos %}
var welcome = _.compose(greet, exclaim);
welcome('there')
// → hi: THERE!
{% endhighlight %}
*Note: compose invokes the provided functions from right to left.*

### Things worth knowing about lodash
#### Composition left

Lodash is born as a fork of underscore, therefore everything we just saw can also be achieved with lodash. For example we may like compose but it may feel weird for its right to left math derived invocation. Lodash [still permit that mathematical approach](https://lodash.com/docs#flowRight) but also features a left to right version of the method that is not called composeLeft but simply [_.flow](https://lodash.com/docs#flow).

Lodash, being sort of underscore gen2 offer better performances thanks to advanced code optimizations. It can be seen as a drop-in replacement for underscore, and it won't be hard to replace the first with the latter.

Among the extra perks that comes with lodash one is definitely worth exploring:

#### Currying

In JavaScript a function takes a number of arguments, and returns a value.
We can call a function with:

- fewer paramaters then the function's arity (*arguments.length<function.length*), most probably with odd results, or
- with too many, exceding the function's arity (*arguments.length>function.length*) with the result that the arguments in excess normally get ignored.

<br>
A curried function is a function that may be invoked with fewer arguments then its original arity, returning a function ready to be invoked with the remaining ones.

Given a sum3 function, that when invoked with 3 arguments will return their sum, we can create its curried version as follows:

{% highlight javascript linenos %}
var sum3 = function(a, b, c) {
  return a + b + c;
};

var curriedSum = _.curry(sum3);

curriedSum(1)(2)(3);
// → 6

curriedSum(1, 2)(3);
// → 6

curriedSum(1, 2, 3);
// → 6

// using empty placeholders
curriedSum(1)(_, 3)(2);
// → 6
{% endhighlight %}

If you love the taste of curry (and you should, it's damn tasty) then you'll
probably need to know about Ramda.

***

## Ramda

> [Ramda](http://ramdajs.com/) is a practical functional library for Javascript programmers.

Ramda is not a drop-in replacement for underscore or lodash, as it has a more focused goal. It's a library designed specifically for a functional programming style, one that makes it easy to create functional pipelines, one that never mutates user data.

### Things worth knowing about Ramda
#### Function first, data last API

The parameters to Ramda functions are arranged to make it convenient for currying as the data to be operated on is generally supplied last.

lodash map vs rampda map functions
{% highlight javascript linenos %}
// lodash
_.map(collection, iteratee)

// ramda
R.map(iteratee, collection)
{% endhighlight %}

#### Auto-currying

Ramda functions are automatically curried. This allows you to easily build up new functions from old ones simply by not supplying the final parameters. This coupled with the function first, data last API make ramda not just functional purer, but definitely purely awesome.

Let's then rebuild part of the underscore/lodash code we saw earlier with ramda instead.

{% highlight javascript linenos %}
// Underscore/lodash style:
var recommendedMovies = function(movies) {
  return _.filter(movies, isRecommended);
};

// Ramda style:
var recommendedMovies = R.filter(isRecommended);
{% endhighlight %}

Because every ramda function automatically curry, omitting to pass the collection argument to R.filter return us the curryed function so that we are ready to go.
***

In this post we looked at few different flavours of JavaScript functional libraries together with some nice to know methods for each of them.

We now know why we should care, how to do chaining, composition and currying with them. For more info check their respective documentations:

- [Underscore on DevDocs.io](http://devdocs.io/underscore/)
- [lodash on DevDocs.io](http://devdocs.io/lodash/)
- [Ramda docs](http://ramdajs.com/docs/)


