---
title: Demystifying Redux pt2
date: 2016-09-12
description: by reimplementing a naif version of applyMiddleware
---

## [Redux](https://github.com/reactjs/redux) is a predictable state container for JavaScript apps. It borrow ideas from both the [flux architecture](https://facebook.github.io/flux/) and the [elm architecture](http://guide.elm-lang.org/architecture/).

This article, is part of a series of posts aiming to explain the redux architecture and its internals by reimplementing parts of it (*Disclaimer: naif/not-optimized implementation ahead*). As we demistified createStore in the previous post, this time we’ll take a look at : **applyMiddleware**

***

### applyMiddleware - express-like middlewares for state management

In the previous post of this series, we saw that createStore will return an object, holding access to the state of our app, a getState method to retrieve it, a dispatch method responsible for changing our state by dispatching actions to reducers, and a subscribe mechanism allowing to be notified once an action has been dispatched (and potentially, a related state change happened).

Let’s see again how the dispatch logic work:

{% highlight javascript %}
const dispatch = action => {
  state = reducer(state, action)
}
{% endhighlight %}

The dispatch method has access to the state before and after state reduction happens. If you have some knowledge of express.js (and similars, like koa or hapi) you might see where the middleware idea come from. In fact this is the only part of redux, that you won’t find in the elm architecture (redux borrows many of it’s ideas and the overall architecture from the elm architecture)

As we did with createStore in the previous post, lets’ start by thinking about the API of applyMiddleware. We want applyMiddleware to being able accept the middleware. It should return a function that we could call on createStore, that will return a new createStore function, with the middleware applied so that we can finally use to instantiate our store. In a sense we want to configure how we create stores, by enhancing our createStore capabilities trough the middleware.

{% highlight javascript %}
function applyMiddleware(middleware) {
  return createStore => 
    (reducer, initialState) => {
     ..
     // return enhanced Store dispatcher
    }
}
{% endhighlight %}

To use it we could simply do:

{% highlight javascript %}
const createEnhancedStore = applyMiddleware(enhancingMiddleware)(createStore)
const store = createEnhancedStore(rootReducer, initialState)
{% endhighlight %}

