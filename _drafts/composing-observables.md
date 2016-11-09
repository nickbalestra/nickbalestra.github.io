---
title: Composing Observables
date: 2015-06-09
description:
---

const Observable = function(producer) {
  this.subscribe = producer
}

Observable.of = function(...values){
  const producer = observer => {
    try {
      values.forEach(value => observer.next(value))
      observer.complete()
    } catch(err) {
      observer.error(err)
    }
  }

  return new Observable(producer)
}

Observable.prototype.map = function(fn){
    const producer = this.subscribe;
  //   return new Observable(function(observer){
  //   })

  return new Observable(function(observer){
    producer({
      next(value){observer.next(fn(value))},
      error(err){observer.error(err)},
      complete(){observer.complete()}
    })
  })
}


const observableHelloWorld = Observable.of(10, 20)

const observer = {
  next(value) { console.log(value) },
  error(err) { console.log('Error: ', err) },
  complete() { console.log('Done') }
}

// console.log(observableHelloWorld.map(function(x){return x * 2}))
const mappedObservable = observableHelloWorld.map(function(x){return x * 2})
const doubleMappedObservable = mappedObservable.map(function(x){return x + 3})
// console.log(mappedObservable)
doubleMappedObservable.subscribe(observer)
// observableHelloWorld.subscribe(observer)
