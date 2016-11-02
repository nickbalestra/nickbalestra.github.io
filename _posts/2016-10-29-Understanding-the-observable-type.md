---
title: Understanding the observable type
date: 2016-10-29
description: by reimplementing a naive version of it
---

## The Observable type is used to model push-based data sources. Observables are compositional and lazy: can be composed with higher-order combinators and do not start emitting data until an observer has subscribed.

Observables are everywhere: we can find them used in various codebases, from [Cycle.js](https://cycle.js.org/) to [Angular2](https://angular.io/), from [Horizon](http://horizon.io/) to [Falcor](http://netflix.github.io/falcor/), from [Mobx](https://mobxjs.github.io/mobx/) to [Ecmascript proposal](https://tc39.github.io/proposal-observable/), you name it. If you followed the [ReactiveConf](https://reactiveconf.com/) in Bratislava this week, I'm sure you noticed how observables were mentioned in almost every talk. I started playing with them a couple of weeks ago as I began using cycle.js (A functional and reactive JavaScript framework for cleaner code). Observables cannot be ignored, so, [let's try](https://twitter.com/mbrochh/status/792223833914642432) to wrap our head around it.

As Ben Lesh [wrote](https://medium.com/@benlesh/learning-observable-by-building-observable-d5da57405d87#.ev2dy0b9f): If you really want to understand observable, you could simply write one. In this post, I'll try to do exactly that.

***

## A basic observable

> An observable, boiled down to it’s smallest parts, is no more than a specific type of function with a specific purpose. ~ [@benlesh](https://twitter.com/BenLesh)

So what is the purpose of an observable? Simply connecting an observer to a producer.

{% highlight javascript %}
const Observable = function(producer) {
  this.subscribe = producer
}
{% endhighlight %}

In the above code that contract is perhaps not that clear yet. Where does the observer come into play? To understand that let's explore the anatomy of a producer.

### The producer

We can imagine the producer as being the observable own firehoose. In fact, as the name suggest, it produces and emits data. Every time the producer emit values, those get pushed to the observer. The contract to make this work require the observer to have 3 methods: 'next', 'error' and 'complete'.

Let's see a very basic producer that will simply emit 2 strings ('Hello' and 'World') before signaling its completion.

{% highlight javascript %}
const helloWorldProducer = function(observer) {
  observer.next('Hello')
  observer.next('World')
  observer.complete()
}
{% endhighlight %}

Now that the contract within a producer and an observable is more explicit and visible, we can go ahead
and create an observable using our helloWorldProducer as its data source.

{% highlight javascript %}
const observableHelloWorld = new Observable(helloWorldProducer)
{% endhighlight %}

### The observer

Before proceeding forward watching our new shiny observable in action, we still need to dissect the observer. As mentioned earlier an observer is nothing more than an object with 3 methods: 'next', 'error' and 'complete'.

{% highlight javascript %}
const observer = {
  next(value) { console.log(value) },
  error(err) { console.log('Error: ', err) },
  complete() { console.log('Done') }
}
{% endhighlight %}

### Subscribing

To put it all together, we just have to subscribe to our observable passing in the observer

{% highlight javascript %}
observableHelloWorld.subscribe(observer)
// -> "Hello"
// -> "World"
// -> "Done"
{% endhighlight %}

[Play with the above code on jsBin](https://jsbin.com/mebiqowove/edit?js,console)

***

## Handling errors

Sofar our code doesn't handle errors. Lets try to fix this:

{% highlight javascript %}
const helloWorldProducer = function(observer) {
  try {
    observer.next('Hello')
    // let's simulate something going wrong...
    throw new Error('Oups')
    observer.next('World')
    observer.complete()
  } catch(err) {
    observer.error(err)
  }
}
{% endhighlight %}

Let's subscribe again and see what happens:

{% highlight javascript %}
observableHelloWorld.subscribe(observer)
// -> "Hello"
// -> "Error: [object Error]"
{% endhighlight %}

[Play with the above code on jsBin](https://jsbin.com/sihijuzuqi/edit?js,console)

Is worth noting that an observer doesn’t have to have all of the methods implemented.

>'next', 'error' and 'complete' are all actually optional. You don’t have to handle every value, or errors or completions. You might just want to handle one or two of those things. ~ [@benlesh](https://twitter.com/BenLesh)

***

## Unsubscribing

Subscribing to our observable doesn't return anything yet, normally we would want to return a dispose function allowing us to unsubscribe from the observable, let's add this in.

But first, in order to better understand why this could be useful, let's refactor our hello word producer adding a bit of asyncness to it. Let's transform it into a producer that will keep emitting an incremental counter every second:

{% highlight javascript %}
const intervalProducer = function(observer) {
  let counter = 0
  const timer = setInterval(() => {
    try {
      observer.next(counter++)
    } catch(err) {
      observer.error(err)
    }
  }, 1000)
}
{% endhighlight %}

I guess you can now see why being able to unsubscribe from our observable is very important... Exactly, subscribing to an observable made with the produced we just defined, will keep pushing data to us, for ever, without us being able to stop it.

To solve this, let's make sure that we return an unsubscribe function for that:

{% highlight javascript %}
const intervalProducer = function(observer) {
  let counter = 0

  const unsubscribe = function() {
    clearInterval(timer)
  }

  const timer = setInterval(() => {
    try {
      observer.next(counter++)
    } catch(err) {
      unsubscribe() // <- on error we also want to unsusbcribe
      observer.error(err)
    }
  }, 1000)

  return {
    unsubscribe
  }
}
{% endhighlight %}

we can now subscribe and at anytime we want also unsubscribe

{% highlight javascript %}
const observableInterval = new Observable(intervalProducer)

const sub = observableInterval.subscribe(observer)

setTimeout(() => {
  sub.unsubscribe()
}, 6000)

// -> 0
// -> 1
// -> 2
// -> 3
// -> 4
// -> 5
{% endhighlight %}

[Play with the above code on jsBin](https://jsbin.com/pafuqipeko/edit?js,console)

***

In the next posts, we'll go through

- Implementing different observables, like 'fromArray' or 'fromEvent'
- Understanding Hot vs Cold Observables and multicasting
- Implementing some of the basic operators you could find in a stream library like 'rxjs'

***

Resources worth checking:

Many thanks go to [André Staltz](https://twitter.com/andrestaltz), [Jafar Husain](https://twitter.com/jhusain), [Ben Lesh](https://twitter.com/BenLesh) and all the great people that wrote great articles and produced great resources that are helping me better understand the topic. I'm still fresh on the subject, so I probably misunderstood something or got some things wrong, if so, please do let me know.

Further must-read resources that I highly recommend:

- [The introduction to Reactive Programming you've been missing](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)
- [Learning Observable By Building Observable](https://medium.com/@benlesh/learning-observable-by-building-observable-d5da57405d87)
- [RxJS Beyond the Basics: Creating Observables from scratch](https://egghead.io/courses/rxjs-beyond-the-basics-creating-observables-from-scratch)
