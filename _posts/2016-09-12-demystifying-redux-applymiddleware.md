---
title: Demystifying Redux pt2
date: 2016-09-12
description: by reimplementing a naif version of applyMiddleware
---

## [Redux](https://github.com/reactjs/redux) is a predictable state container for JavaScript apps. It borrow ideas from both the [flux architecture](https://facebook.github.io/flux/) and the [elm architecture](http://guide.elm-lang.org/architecture/).

This article, is part of a series of posts aiming to explain the redux architecture and its internals by reimplementing parts of it (*Disclaimer: naif/not-optimized implementation ahead*). As we demystified createStore in the [previous post](http://nick.balestra.ch/2016/demystifying-redux-createstore/), this time we’ll take a look at : **applyMiddleware**

***

### applyMiddleware - express-like middleware for Redux

[Earlier](http://nick.balestra.ch/2016/demystifying-redux-createstore/) we saw that createStore returns an object, holding access to the state of our app, a getState method to retrieve it, a dispatch method responsible for changing our state by dispatching actions to reducers, and a subscribe mechanism allowing to be notified once an action has been dispatched.

Let’s see again how the dispatch logic work:

{% highlight javascript %}
const dispatch = action => {
  state = reducer(state, action)
}
{% endhighlight %}

The dispatch method has access to the state before and after state reduction happens. If you have some knowledge of [express.js](https://expressjs.com/) (& Co., like [Koa](http://koajs.com/) or [Hapi](http://hapijs.com/)) you might see where the middleware idea come from. In fact, this is the only part of Redux, that you won’t find in the Elm architecture. Redux allows you via applyMiddleware to enhance the store.dispatch logic.

#### applyMiddleware API

As we did with createStore, lets’ start from the API of applyMiddleware. We want applyMiddleware to take the middleware as the argument. It should return a function that we could call on createStore, that will return a new createStore function, with the middleware applied. In a sense, we want to configure how we create stores, by enhancing our createStore capabilities trough the middleware. Or simply put: hijack the dispatch method of createStore with an enhanced one, AKA: applying a middleware to our store.

{% highlight javascript %}
function applyMiddleware(middleware) {
  return createStore => 
    (reducer, initialState) => {
     …
     // return enhanced store
    }
}
{% endhighlight %}

To use it, simply:

{% highlight javascript %}
const createEnhancedStore = applyMiddleware(middleware)(createStore)
const store = createEnhancedStore(reducer, initialState)
{% endhighlight %}

#### Hijacking the dispatch method

In order to be able to hijack the dispatcher we first need to save  the original dispatch method. As we are doing this, let also keep a record to the getState method, as we want to be able to inject both into our middleware.

{% highlight javascript %}
function applyMiddleware(middleware) {
  return createStore => 
    (reducer, initialState) => {
      const store = createStore(reducer, initialState); // <-
      const dispatch = store.dispatch; // <-
      const getState = store.getState; // <-
    }
}
{% endhighlight %}

We can now inject those method into our middleware:

{% highlight javascript %}
function applyMiddleware(middleware) {
  return createStore => 
    (reducer, initialState) => {
      const store = createStore(reducer, initialState);
      const dispatch = store.dispatch;
      const getState = store.getState;
  
      const injectedMIddleware = middleware({ dispatch, getState }); // <-
      const enhancedDispatch = injectedMIddleware(dispatch); // <-
    }
}
{% endhighlight %}

and finally we can return the new enhanced store with the hijacked dispatcher:

{% highlight javascript %}
function applyMiddleware(middleware) {
  return createStore => 
    (reducer, initialState) => {
       const store = createStore(reducer, initialState);
       const dispatch = store.dispatch;
       const getState = store.getState;
    
      const injectedMIddleware = middleware({ dispatch, getState });
      const enhancedDispatch = injectedMIddleware(dispatch);
      
      return Object.assign({}, store,{ dispatch:enhancedDispatch }); // <-
    }
}
{% endhighlight %}

And that’s it! 
Let’s just stop for a moment and reason about what we just did:

**applyMiddleware = middleware -> createStore -> (reducer, initialState) => store**

Before putting the pieces together, lets see how a middleware will look like.

### Anatomy of a redux-middleware

Again, let start by the API. We saw that in order to inject the original dispatch and getState method we call our middleware passing those methods via an optionHash parameter:

**injectedMiddleware({dispatch, getstate})**

This should return a function that we can compose with our dispatcher in a chain-able way. Let’s see as an example a loggingMiddleware that will :

- Log the previous state before every action is dispatched
- Log the action type being dispatched
- Log the state after the action has been dispatched 
<br /><br /> 
{% highlight javascript %}
function createLoggingMiddleware({ getState }) {
  return dispatch => 
    action => {
      const previousState = getState();
      const dispatched = dispatch(action);
      const currentState = getState();
     
      console.log(`Previous state: ${previousState}`);
      console.log(`Action dispatched: ${action.type}`);
      console.log(`Current state: ${currentState}`);
      console.log("=======================")  

      return dispatched; 
    };
} 
{% endhighlight %}

#### Putting the pieces together:

Lets import our [createStore together with our counter reducer from the previous post](http://nick.balestra.ch/2016/demystifying-redux-createstore), together with our freshly coded applyMiddleware and createLoggingMiddleware:

{% highlight javascript %}
import createStore from ‘./naif-redux/createStore’
import applyMiddleware from ‘./naif-redux/applyMiddleware’
import counter from ‘./reducers’
import createLoggingMiddleware from ‘./middlewares/loggingMiddleware’
{% endhighlight %}

We can now wire all the pieces together and see that even without subscribing to our store in order to log its state after each action being dispatched, out store will automatically log previous and current state along with the type of action dispatched… powaaa!

{% highlight javascript %}

const createEnhancedStore = applyMiddleware(createLoggingMiddleware)(createStore);
const store = createEnhancedStore(counter)   
   
store.dispatch({ type: 'INCREMENT' })
// Previous state: 0
// Action dispatched: INCREMENT
// Current state: 1
// =======================

store.dispatch({ type: 'INCREMENT' })
// Previous state: 1
// Action dispatched: INCREMENT
// Current state: 2
// =======================

store.dispatch({ type: 'DECREMENT' })
// Previous state: 2
// Action dispatched: DECREMENT
// Current state: 1
// =======================
{% endhighlight %}

***

### Conclusion and extras

The above implementation doesn’t allow us to apply multiple middlewares if you want to add that feature take a look at the beautiful : [compose](https://github.com/reactjs/redux/blob/master/src/compose.js) helper that comes with Redux.

If you want to play around with the code above, feel free to fork my [redux-playground repository](https://github.com/nickbalestra/redux-playground).

#### Previous related post:

- [Demystifying Redux pt1](http://nick.balestra.ch/2016/demystifying-redux-createstore/)