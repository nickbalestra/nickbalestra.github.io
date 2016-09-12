---
title: Demystifying Redux pt1
date: 2016-09-4
description: by reimplementing a naif version of createStore
---

## [Redux](https://github.com/reactjs/redux) is a predictable state container for JavaScript apps. It borrow ideas from both the [flux architecture](https://facebook.github.io/flux/) and the [elm architecture](http://guide.elm-lang.org/architecture/). Shortly:

> **state = f(state, action)**

This article, is part of a series of posts aiming to explain redux architecture and its internals by reimplementing parts of it (*Disclaimer: naif/not-optimized implementation ahead*). We’ll start from the core, main logic: **createStore**

***

### createStore - the redux state container

By a quick look at the [official README](https://github.com/reactjs/redux/blob/master/README.md), the gist showing the basic concepts of redux purely rely on createStore:

{% highlight javascript %}
import { createStore } from 'redux’
{% endhighlight %}

As the name suggest it creates a Redux store holding the state of the app. Its API is { subscribe, dispatch, getState }. It requires a reducer as its only mandatory argument:

{% highlight javascript %}
let store = createStore(reducer)
{% endhighlight %}

This give us enough information to start writing some code.

{% highlight javascript %}
function createStore(reducer) {
  let state // holding the state of the app
  
  const getState = () => {}
  const dispatch = () => {}
  const subscribe = () => {}
  
  return { getState, dispatch, subscribe} // API
}
{% endhighlight %}

Now that we have a skeleton in place lets implement the createStore APIs.

#### getState()

Not much to be said about this method, it will simply return the enclosed state:

{% highlight javascript %}
const getState = () => state
{% endhighlight %}

#### dispatch()

This is how redux mutate the state. By dispatching an action (nothing more then a simple [POJO](https://en.wikipedia.org/wiki/Plain_Old_Java_Object)) that express the mutation intent. (In this regards I prefer how the elm architecture call it a message instead of an action, as it just carry the information but doesn’t perform the actual action itself’)

The dispatch signature will then look something like:

{% highlight javascript %}
const dispatch = action => {}
{% endhighlight %}


So if the dispatch method take an action as its only argument, and that action is nothing more then an object carrying some informations, you’ll probably be wondering how is the state going to be mutated… Say hello to the reducer! The role of the reducer is to reduce the state tree to a new tree with the applied transformation in place, aka: mutate the state.

**reducer = (state, action) => state**

Pretty simple eh? We stated earlier that createStore require only a reducer as mandatory parameter, so, as it is already defined we can simply use it inside our method:

{% highlight javascript %}
const dispatch = action => {
  state = reducer(state, action)
}
{% endhighlight %}

Et voila! the store dispatcher now does what it says: dispatches an action to the reducer. The reducer will analyse the message (action) and purely mutate the state accordingly, returning the new state.

## subscribe()

Up to this point we are able to get the state from our store by calling getState() and we are able to mutate the state by dispatching actions via dispatch(). To get automatically informed about this changes we need a subscription logic. Subscribe will indeed allows us to register listeners so that each time an action get dispatched, and most probably the state get mutated all the subscribed clients (our listeners) will be invoked.
On subscribing we will return a dispose function allowing us to unsubscribe.

**subscribe = listener => {dispose}**

First lets add a container to hold the active listeners, and the relative logic to add/remove them:

{% highlight javascript %}
let subscriptions = []

const subscribe = listener => {
  subscriptions.push(listener)
  return {
    dispose() { 
      subscriptions.splice(subscriptions.indexOf(listener),1)
    }
  }
}
// I prefer concat over push and slice over splice
// but for the sake of the post I believe this make the intent clearer.
{% endhighlight %}

We now need to trigger our listeners once a dispatch occurs:

{% highlight javascript %}
const dispatch = action => {
  state = reducer(state, action)
  subscriptions.forEach(listener => listener())
}
{% endhighlight %}

#### Initialize the store

Before exposing the API’s we’ll now just need to initialise the store, by dispatching an init action, that will set the initial state of the store.

{% highlight javascript %}
dispatch({type: 'INIT'})
{% endhighlight %}

***

### Conclusions

That’s it, you can now create a store, dispatch actions to mutate it and have all the subscribed clients be notified about any changes. This is the main overall architecture of redux. Simple and elegant. 

Our final implementation of createStore:

{% highlight javascript %}
function createStore(reducer) {
  let state
  let subscriptions= []

  const getState = () => state

  const dispatch = action => {
    state = reducer(state, action)
    subscriptions.forEach(listener => listener())
    return action
  }

  const subscribe = listener => {
    subscriptions.push(listener)
    return {
      dispose() {
        subscriptions.splice(subscriptions.indexOf(listener),1)
      }
    }
  }
	
  dispatch({type: 'INIT'})

  return { getState, dispatch, subscribe}
}
{% endhighlight %}

There are some more complex nuances like applyMiddleware, combineReducers and finally how redux can be used with a view layer like react (think react = state => f(state)) but I’ll cover those in the upcoming articles.

You can now follow the basic gist in the official redux documentation and it should work as expected:

{% highlight javascript %}
// create a reducer
function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1
  case 'DECREMENT':
    return state - 1
  default:
    return state
  }
}

// Create a Redux store holding the state of your app.
let store = createStore(counter)
// You can use subscribe() to update the UI in response to state changes.
store.subscribe(() =>
  console.log(store.getState())
)

// The only way to mutate the internal state is to dispatch an action.
// The actions can be serialized, logged or stored and later replayed.
store.dispatch({ type: 'INCREMENT' })
// 1
store.dispatch({ type: 'INCREMENT' })
// 2
store.dispatch({ type: 'DECREMENT' })
// 1
{% endhighlight %}

Finally we want to be able to specify an initial state when instantiating our store. Simple: move the state definition from the body of the function to its signature:

{% highlight javascript %}
function createStore(reducer, state) {
  ...
}
{% endhighlight %}

***

If you want to play around, feel free to fork my [redux-playground repository](https://github.com/nickbalestra/redux-playground)

#### Related post:

- [Demystifying Redux pt2](http://nick.balestra.ch/2016/demystifying-redux-applymiddleware/)
