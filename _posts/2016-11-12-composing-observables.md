---
title: Understanding the observable type pt.3
date: 2016-11-12
description: by writing composition operators
---

## This post is the last of a series of posts in which I write about the Observable type. In the first post, we went ahead [writing an observable from scratch](http://nick.balestra.ch/2016/Understanding-the-observable-type/) in order to fully understand it. We then explored how to [create observables from values, arrays, dom events and promises](http://nick.balestra.ch/2016/creating-observables/). This time we'll focus on compositions by rewriting some basic composition operators.

***

## Observable.map

Mapping is a commong pattern:

  - Unwrap an item from a container
  - Apply a transformation to it
  - Wrap the result of the transformation back to a similar container
<br /><br />
For example, let's assume our container to be an array, our wrapped item a number and the transformation we want to apply is to square it:
<br /><br />
{% highlight javascript %}
const three = Array.of(3)
const square = (num) => num * num

three.map(square)
// -> [9]
{% endhighlight %}

Let's now imagine our item to be wrapped in an observable instead. Mapping over it, due to the lazy nature of observables will means:

  - Unwrap the producer from the observable
  - Hijack it so that it will apply the transformation to its outputs
  - Return a new observable with the hijacked producer.
<br /><br />

Talk is cheap, let's write an implementation for the map method:

{% highlight javascript %}
Observable.prototype.map = function(transformation){
  // Unwrap the producer from the observable
  const originalProducer = this.subscribe;

  // Hijack it so that it will apply the transformation to its outputs
  const newProducer = function(observer){
    originalProducer({
      next (value) {observer.next(transformation(value))},
      error (err) {observer.error(err)},
      complete () {observer.complete()}
    })
  }

  // Return a new observable with the hijacked producer.
  return new Observable(newProducer)
}
{% endhighlight %}

Boom! We can now rewrite our earlier array example using observables instead. Our container is going to be an Observable, our wrapped item and the transformation remains unchanged.
:

{% highlight javascript %}
const three = Observable.of(3)
const square = (num) => num * num

const nine = three.map(square)

nine.subscribe({
  next(value) { console.log(value) },
  error(err) { console.log('Error: ', err) },
  complete() { console.log('Done') }
})
// -> 9
// -> 'Done'
{% endhighlight %}

[Play with the above code on jsBin](https://jsbin.com/dodajiwapo/edit?js,console)

***

## Observable.mapTo

Sometimes we want to simply map values statically. For example, imagine we created an Observable of click events and we want to map those clicks to redux-like actions:

{% highlight javascript %}
{ type: 'INCREASE'}
{% endhighlight %}

We can build a mapTo method on top of the map method we just implemented:

{% highlight javascript %}
Observable.prototype.mapTo = function(value){
  return this.map(() => value)
}
{% endhighlight %}

Et voila!. We are now able to create a stream of redux-like actions out of click events:

{% highlight javascript %}
const increaseButton  = document.getElementById('increase')

const action$ = Observable
  .fromEvent(increaseButton, 'click')
  .mapTo({ type: 'INCREASE'})

const click$ = action$.subscribe({
  next(value) { console.log(value) },
  error(err) { console.log('Error: ', err) },
  complete() { console.log('Done') }
})

// everytime we click we get the object representing the action:
//
// [object Object] {
//   type: "INCREASE"
// }
{% endhighlight %}

[Play with the above code on jsBin](https://jsbin.com/johulucofa/edit?js,console,output)

***

## Observable.filter

Filter an observable sequence according to a predicate.

{% highlight javascript %}
Observable.prototype.filter = function(predicate){
  // Unwrap the producer from the observable
  const originalProducer = this.subscribe;

  // Hijack it so that it will push only the value passing the predicate test
  const newProducer = function(observer){
    originalProducer({
      next (value) {
        if (predicate(value) === true) {
          observer.next(value)
        }
      },
      error (err) {observer.error(err)},
      complete () {observer.complete()}
    })
  }

  // Return a new observable with the hijacked producer.
  return new Observable(newProducer)
}
{% endhighlight %}

Let's immagine we have an observable interval and we want to filter it so that only even numbers are produced:

{% highlight javascript %}
const isEven = (num) => num % 2 === 0

evenCount$ = Observable
  .interval(500)
  .filter(isEven)

evenCount$.subscribe({
  next(value){console.log(value)},
  complete(){console.log('done')},
  error(err){console.log(err)}
})

// ..after 1 sec -> 0
// ..after 1 sec -> 2
// ..after 1 sec -> 4
// ..after 1 sec -> 6
// ..after 1 sec -> 8
// ..
{% endhighlight %}

[Play with the above code on jsBin](https://jsbin.com/buzofulisu/edit?js,console)

***

## Observable.takeUntil

Returns an observable sequence that stop emitting values as soon as a predicate test pass.

{% highlight javascript %}
Observable.prototype.takeUntil = function(predicate){
  // Unwrap the producer from the observable
  const originalProducer = this.subscribe;

  // Hijack it so that it will push values until the predicate test pass
  const newProducer = function(observer){
    const interval = originalProducer({
      next (value) {
        if (predicate(value) !== true) {
          observer.next(value)
        } else {
          interval.unsubscribe()
        }
      },
      error (err) {observer.error(err)},
      complete () {observer.complete()}
    })
  }

  // Return a new observable with the hijacked producer.
  return new Observable(newProducer)
}
{% endhighlight %}

We can now easily create an observable that will emit an incremental value every second until 3 is emitted.

{% highlight javascript %}
const isGreaterThenThree = (num) => num > 3 === true

const countTillThree$ = Observable.interval(1000)
  .takeUntil(isGreaterThenThree)

const sub = countTillThree$.subscribe({
  next(value){console.log(value)},
  complete(){console.log('done')},
  error(err){console.log(err)}
})
// -> 0
// -> 1
// -> 2
// -> 3
{% endhighlight %}

[Play with the above code on jsBin](https://jsbin.com/visexuruya/edit?js,console)

***

## Conclusion

Although very naively, we just implemented a very basic stream library. I leave it up to the reader to implement it further, maybe taking some inspiration from the following libraries:

- [RxJS](https://github.com/Reactive-Extensions/RxJS/)
- [xstream](https://github.com/staltz/xstream)
- [most](https://github.com/cujojs/most)

***

Resources worth checking:

Many thanks go to [André Staltz](https://twitter.com/andrestaltz), [Jafar Husain](https://twitter.com/jhusain), [Ben Lesh](https://twitter.com/BenLesh) and all the great people that wrote great articles and produced great resources that are helping me better understand the topic. I'm still fresh on the subject, so I probably misunderstood something or got some things wrong, if so, please do let me know.

Further must-read resources that I highly recommend:

- [The introduction to Reactive Programming you've been missing](https://gist.github.com/staltz/868e7e9bc2a7b8c1f754)
- [Learning Observable By Building Observable](https://medium.com/@benlesh/learning-observable-by-building-observable-d5da57405d87)
- [RxJS Beyond the Basics: Creating Observables from scratch](https://egghead.io/courses/rxjs-beyond-the-basics-creating-observables-from-scratch)
