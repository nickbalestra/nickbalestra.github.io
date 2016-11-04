---
title: Understanding the observable type pt.2
date: 2016-11-05
description: by reimplementing part of the tc39 proposal
---

## This post is part of a series in which I write about observables. In my [previous article](http://nick.balestra.ch/2016/Understanding-the-observable-type/) I went ahead and wrote an observable from scratch in order to fully understand it. This time I'll be exploring how to create observables from values, arrays, dom events and promises.

***

## Creating observables of...

We saw how every time the producer emit values, those get pushed to the observer. For that we implemented a very basic producer that simply emit 2 strings ('Hello' and 'World') before signaling its completition. Here it is again:

{% highlight javascript %}
const helloWorldProducer = function(observer) {
  try {
    observer.next('Hello')
    observer.next('World')
    observer.complete()
  } catch(err) {
    observer.error(err)
  }
}

const observableHelloWorld = new Observable(helloWorldProducer)
{% endhighlight %}

Wouldn't be nice if we could have an easier and quicker way to create Observables that will emit a stream of predefined values,without us having to worry about signalign completion or handling error?

Say hello to **Observable.of**!

As the tc39 proposal suggest the API for [Observbale.of](https://tc39.github.io/proposal-observable/#observable-of) could look something like:

{% highlight javascript %}
const observableHelloWorld = Observable.of('Hello', 'World')
{% endhighlight %}

The implementation for this method will mainly use the constructor we already wrote earlier hiding away some complexity in order to:

**1 create a producer** in charge to:

- call observer.next for each of given values
- signaling completion aftwerward and
- being able to handle errors if any occurs.
<br /><br />
**2 create a new observable** with the given producer
<br /><br />
{% highlight javascript %}
Observable.of = function(...values){
  // create a producer
  const producer = observer => {
    try {
      // call observer.next for each of given values
      values.forEach(value => observer.next(value))
      // signaling completion aftwerward
      observer.complete()
    } catch(err) {
      // handle errors if any occurs
      observer.error(err)
    }
  }

  // create a new observable with the given producer
  return new Observable(producer)
}
{% endhighlight %}

Et voila, we can now simply use the proposed API to create our initial observableHelloWorld in a much simpler and effictive way:

{% highlight javascript %}
const observableHelloWorld = Observable.of('Hello', 'world')

observableHelloWorld.subscribe({
  next(value) { console.log(value) },
  error(err) { console.log('Error: ', err) },
  complete() { console.log('Done') }
})
// -> "Hello"
// -> "world"
// -> "Done"

{% endhighlight %}

[Play with the above code on jsBin](https://jsbin.com/nanivageze/edit?js,console
)








For each value we want

If you think about it, the idea is to si call the observer.next method pa

So what is the purpose of an observable? Simply connecting an observer to a producer.

{% highlight javascript %}
const Observable = function(producer) {
  this.subscribe = producer
}
{% endhighlight %}

In the above code that contract is perhaps not that clear yet. Where does the observer come into play? To understand that let's explore the anatomy of a producer.

### The producer

We can imagine the producer as being the observable own firehoose. In fact, as the name suggest, it produce and emit data. Every time the producer emit values, those get pushed to the observer. The contract to make this work require the observer to have 3 methods: 'next', 'error' and 'complete'.

Let's see a very basic producer that will simply emit 2 strings ('Hello' and 'World') before signaling its completition.

{% highlight javascript %}
const helloWorldProducer = function(observer) {
  observer.next('Hello')
  observer.next('World')
  observer.complete()
}
{% endhighlight %}

Now that the contract within a producer and an observable is more explicit and visible, we can go ahead
and create an observable using our helloWorldProducer as its datasource.

{% highlight javascript %}
const observableHelloWorld = new Observable(helloWorldProducer)
{% endhighlight %}

### The observer

Before proceding forward watching our new shiny observable in action, we still need to disect the observer. As mentioned earlier an observer is nothing more then an object with 3 methods: 'next', 'error' and 'complete'.

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

## Unsusbscribing

Subscribing to our observable doesn't return anything yet, normally we would want to return a dispose function allowing us to unsubscribe from the observable, let's add this in.

But first, in order to better understand why this could be useful, let's refactor our hello word producer adding a bit of asyncness to it. Let's transform it into a producer that will keep emit an incremental counter every second:

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

I guess you can now see why being able to unsusbcribe from our observable is very important... Excactly, subscribing to an observable made with the produced we just defined, will keep pushing data to us, for ever, without us being able to stop it.

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

we can now subscribe and at anytime we want also unsusbcribe

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

In the next posts we'll go through

- Implemening different observables, like 'fromArray' or 'fromEvent'
- Understaning Hot vs Cold Observables and multicasting
- Implementing some of the basic operator you could find in a stream library like 'rxjs'

***

Resources worth checking:

Many thanks goes to [André Staltz](https://twitter.com/andrestaltz), [Jafar Husain](https://twitter.com/jhusain), [Ben Lesh](https://twitter.com/BenLesh) and all the great people that wrote great articles and produced great resources that are helping me better understand the topic. I'm still fresh on the subject, so I probably misunderstood something or got some things things wrong, if so, please do let me know.

Further must-read resources that I highly recommend:

- [The introduction to Reactive Programming you've been missing](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)
- [Learning Observable By Building Observable](https://medium.com/@benlesh/learning-observable-by-building-observable-d5da57405d87)
- [RxJS Beyond the Basics: Creating Observables from scratch](https://egghead.io/courses/rxjs-beyond-the-basics-creating-observables-from-scratch)

