---
title: Understanding the observable type pt.2
date: 2016-11-05
description: by creating them out of arrays, events and promises
---

## In my previous post we went ahead [writing an observable from scratch](http://nick.balestra.ch/2016/Understanding-the-observable-type/) in order to fully understand it. This time we'll be exploring how to create observables from values, arrays, dom events and promises.

***

## Creating observables of...

We saw how every time the producer emit values, those get pushed to the observer. For that, we implemented a very basic producer that simply emit 2 strings ('Hello' and 'World') before signalling its completion. Here it is again:

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

Wouldn't be nice if we could have an easier and quicker way to create such an Observable ?

### Say hello to Observable.of()

As per the tc39 proposal, the signature for [Observbale.of](https://tc39.github.io/proposal-observable/#observable-of) will accept a list of items, returning a properly configured observable which will start emitting those value in sequence as soon as we subscribe to it.

Our implementation of the Observable.of method will rely on the constructor we already wrote earlier, and will allow us to hide away some complexity in order to:

**1 create a producer** (a subscriber function) in charge to:

- call observer.next for each of the given values
- signalling completion afterward and
- being able to handle errors if any occurs.
<br /><br />
**2 create and return a new observable** with the given producer
<br /><br />

{% highlight javascript %}
Observable.of = function(...values){
  // create a producer
  const producer = observer => {
    try {
      // call observer.next for each of the given values
      values.forEach(value => observer.next(value))
      // signaling completion aftwerward
      observer.complete()
    } catch(err) {
      // handle errors if any occurs
      observer.error(err)
    }
  }

  // create and return a new observable with the given producer
  return new Observable(producer)
}
{% endhighlight %}

Et voila, we can now simply use the proposed API to create our initial observableHelloWorld in a much simpler and effective way:

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


***

## Creating observables from array

In es6 rest parameters can be destructured, meaning that to create an observable from an array we could simply use Observbale.of passing an array to be destructured:

 {% highlight javascript %}
const words = ['Hello', 'world']
const observableHelloWorld = Observable.of(...words)
{% endhighlight %}

Observable.fromArray is therefore nothing more then a variation of Observable.of especially designed to only accept arrays:

{% highlight javascript %}
Observable.fromArray = function(array){
  // create a producer
  const producer = observer => {
    try {
      // call observer.next for each value in the array
      array.forEach(value => observer.next(value))
      // signaling completion aftwerward
      observer.complete()
    } catch(err) {
      // handle errors if any occurs
      observer.error(err)
    }
  }

  // create and return a new observable with the given producer
  return new Observable(producer)
}

const words = ['Hello', 'world']
const observableHelloWorld = Observable.fromArray(words)
{% endhighlight %}

[Play with the above code on jsBin](https://jsbin.com/lemoyikuko/1/edit?js,console
)

***

## Creating observables from dom events

Let's imagine we would like to be able to create an Observable that will emit a stream of click events, without having to deal with manually adding event listeners and handlers to do that. All the logic will be, again, hidden away behind the Observable.fromEvent method in order to:

**1 create a producer** (a subscriber function) in charge to:

- create an eventHandler
- add an event listener, so that:
- observer.next will be called whenever an event will happen
- handling errors when needed.
<br /><br />
**2 create and return a new observable** with the given producer so that upon subscribing it will
<br /><br />
**3 returns a function to unsubscribe** (which will handle removing the eventListener)
<br /><br />

{% highlight javascript %}
Observable.fromEvent = function(element, event){
  const producer = observer => {

    // create an eventHandler
    const eventHandler = e => {
      observer.next(e)
    }

    // adding the eventlistener
    try {
      element.addEventListener(event, eventHandler)
    } catch(err) {
      observer.error(err)
    }

    // return a function to unsusbcribe
    return {
      unsubscribe() {
        element.removeEventListener(event, eventHandler)
      }
    }
  }

  //create and return a new observable
  return new Observable(producer)
}
{% endhighlight %}

Et voila, we can now simply use the proposed API to create an observableClick in a very simple and effictive way:

{% highlight javascript %}
const observableClick = Observable.fromEvent(document, 'click')
{% endhighlight %}

Allowing us to subscribe and unsubscribe from it:

{% highlight javascript %}
const click$ = observableClick.subscribe({
  next(value) { console.log(`X: ${event.clientX}, Y: ${event.clientY}`) },
  error(err) { console.log('Error: ', err) },
  complete() { console.log('Done') }
})

// everytime we click we should see the co-ordinates of our mouse on the console:
// "X: 242, Y: 88"

// we can simply unsibscribe using:
const click$ = observableClick.subscribe(observer)

{% endhighlight %}

[Play with the above code on jsBin](https://jsbin.com/dixejebece/edit?js,console,output)

***

## Creating observables from promises

At this point, you probably wonder why not creating an observable from a promise?
All the logic hidden away behind the Observable.fromPromise method will need to:

**1 create a producer** (a subscriber function) in charge to:

- call observer.next once the promise resolve
- signalling completion afterwards
- handling errors when needed.
<br /><br />
**2 create and return a new observable** with the given producer
<br /><br />

{% highlight javascript %}
Observable.fromPromise = function(promise){
  const producer = observer => {

    promise
      // call observer.next once the promise resolve
      .then(value => {
        observer.next(value)
        // signaling completion
        observer.complete()
      })
      // handling errors
      .catch(reason => {
        observer.error(reason)
        observer.complete()
      })
  }

  //create and return a new observable
  return new Observable(producer)
}
{% endhighlight %}

We can now simply create a promise and an observable out of it that will emit the result of the promises as an error if the promise rejects or as the value if the promise resolves correctly:

{% highlight javascript %}
const request = fetch('https://null.jsbin.com')
const observable = Observable.fromPromise(request)

observable.subscribe({
  next(response){console.log(response.status)},
  complete(){console.log('done')},
  error(reason){console.log(reason)}
})
// -> 204
// -> "done"
{% endhighlight %}


[Play with the above code on jsBin](https://jsbin.com/wevucomoca/edit?js,console)

***

## Creating interval observables

Finally, let's implement a way to easily create an observable sequence that produces a value after each period. Nothing new, as already wrote in in my previous post, here it is again wrapped under the Observable.interval method instead:

{% highlight javascript %}
Observable.interval = function(period){

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
    }, period)

    return {
      unsubscribe
    }
  }

 return new Observable(intervalProducer)
}
{% endhighlight %}

We can now easily create an obserbale that will emit an incremental value every second:

{% highlight javascript %}
const count$ = Observable.interval(1000)

const sub = count$.subscribe({
  next(value){console.log(value)},
  complete(){console.log('done')},
  error(err){console.log(err)}
})

setTimeout(()=> sub.unsubscribe(), 5000)
// ..after 1 sec -> 0
// ..after 1 sec -> 1
// ..after 1 sec -> 2
// ..after 1 sec -> 3
// ..after 1 sec -> 4
{% endhighlight %}

[Play with the above code on jsBin](https://jsbin.com/xajopokoru/edit?js,console)

***

## Extras

I leave it up to the reader to implement the following operators:

- Observable.empty: returning an observable sequence with no elements
- Observable.never: returning an observable sequence whose observers will never get called

***

Resources worth checking:

Many thanks go to [André Staltz](https://twitter.com/andrestaltz), [Jafar Husain](https://twitter.com/jhusain), [Ben Lesh](https://twitter.com/BenLesh) and all the great people that wrote great articles and produced great resources that are helping me better understand the topic. I'm still fresh on the subject, so I probably misunderstood something or got some things wrong, if so, please do let me know.

Further must-read resources that I highly recommend:

- [The introduction to Reactive Programming you've been missing](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)
- [Learning Observable By Building Observable](https://medium.com/@benlesh/learning-observable-by-building-observable-d5da57405d87)
- [RxJS Beyond the Basics: Creating Observables from scratch](https://egghead.io/courses/rxjs-beyond-the-basics-creating-observables-from-scratch)

