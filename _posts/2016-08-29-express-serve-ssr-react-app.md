---
title: Express middleware serving SSR React apps
date: 2016-08-29
description: Express, React, JSPM and HML
---

## [Expressjs](https://expressjs.com/) is a fast, unopinionated, minimalist web framework for Node.js. In the expresses idiom a middleware, is, simply put, a function like: (req, res, nextMiddleware) => {}. 

While the following code with some minor adjustment would work fine for any [generic node based server](http://nick.balestra.ch/2015/build-a-simple-api-server-with-node/), this post and its relative code is tailored specifically toward expressjs.

**TLDR;** If you need a starter boiler-plate package for an express middleware serving a react app, with server-side rendering, jspm, and hot-reloading, just clone my [express-serve-app](https://github.com/nickbalestra/express-serve-app) repo, otherwise keep reading.

We’ll be looking at how to wire together and engineer such a middleware. It will offer a minimal configuration upon initialization (i.e. like enabling hot-module-load, setting the endpoint where the app will be served and allowing us to pass parameters to the react app).

Why may you want something like this? Imagine you are working on a discretely complex app, and you need some kind of dev-tool in the form of a GUI driven app, simply hook the middleware in, hit the desired URL end there you go. Of course, that’s only one among many possible scenarios…

The stack is based around express, react, jspm.
 
***

### Setting up the server

Nothing fancy, just spin off a basic express server.

{% highlight javascript %}
const express = require('express')
const app = express()

const server = app.listen(8080, () => 
  console.log(` Server listening on port ${server.address().port}`)
)
{% endhighlight %}

#### Hot Module Loading

For the hot-module-load we'll rely on [systemjs-hot-reloader](https://github.com/capaj/systemjs-hot-reloader) on the client and [chokidar-socket-emitter](https://github.com/capaj/chokidar-socket-emitter) on the server. The latest is a simple chokidar watcher wrapper which emits events to all connected socket.io clients. Systemjs-hot-reloader, on the client, subscribes to the chowkidar emitter, hot reloading modules when needed.

Let’s add an HML environment variable that we’ll use as feature toggle for hot-module-loading. If the HML toggle is set to true we’ll simply spin off the chokidar-socket-emitter, let’s update our lil’ server setup:

{% highlight javascript %}
const express = require('express')
const app = express()
const hml = process.env.HML || false // <—

const server = app.listen(8080, () => 
  console.log(` Server listening on port ${server.address().port}`)
)

if (hml) require('chokidar-socket-emitter')({port: 5776}) // <—
{% endhighlight %}

#### Configuring and using the middleware

As we haven’t yet start building the middleware let just focus at its interface, we want to be able to configure it with:

- endpoint (where the app will be served, i.e.: /boom)
- hml toggle (enabling hot module loading on the client)
<br><br>
Once configured we should be able to register the middleware using the express use() method. Server.js is now complete:<br><br>

{% highlight javascript %}
const express = require('express')
const app = express()
const hml = process.env.HML || false
const serveApp = require('./middleware')({endPoint: '/boom', hml}) // <—

app.use(serveApp) // <—
const server = app.listen(8080, () => 
  console.log(` Server listening on port ${server.address().port}`)
)

if (hml) require('chokidar-socket-emitter')({port: 5776})
{% endhighlight %}

### Writing the middleware

First, as we want the middleware to handle server side rendering (SRR) with react, we need:

{% highlight javascript %}
import {renderToString as render} from 'react-dom/server'
import React from 'react'
import App from '../app/app’
{% endhighlight %}

Earlier we stated that the middleware should offer a minimal configuration upon initialization. Our module will therefore expose a function, taking as argument the options we need to initialize it, returning a function (the configured middleware, ready to be used).

{% highlight javascript %}
module.exports = (options) => {
  const serveApp = (req, res, next) => {
    // Use options to configure our middleware
  }
  return serveApp
}
{% endhighlight %}

The logic for our middleware is pretty straightforward:

1. 1. Check if the requested url match the configuration
2. 2. Render the app server side
3. 3. Respond by sending the page, with the rendered app and the required assets to mount the app on the client
<br><br>
Check the requested URL:
<br><br>
{% highlight javascript %}
if (req.url === options.endPoint)
{% endhighlight %}

Render the app server side:
{% highlight javascript %}
const ssrApp = render(<App />)
{% endhighlight %}

Send the final rendered page and relative assets:
{% highlight javascript %}
const html = `
  <!DOCTYPE html>
  <div id="app">${ssrApp}</div>
  <script src="jspm_packages/system.js"></script>
  <script src="config.js"></script>
  <script>System.import('index.js'))</script>
`
res.status(200).send(html)
{% endhighlight %}

Two things going on here: 

First, as we’ll be using JSPM for managing our bundling client-side, we’ll be relying on a injected bundle workflow, so passing down system.js and config.js will be enough(the bundle map will be injected in config, so that we don’t need to use another script to declare it). We’ll then be using System js to import our app. If you wonder about JSPM, here is [why I love JSPM](http://nick.balestra.ch/2016/why-i-love-JSPM/) and why you should give it a go.

Second, you probably noticed that we are using es6, in order to avoid a build phase, we transpile in memory via the [babel require hook](https://babeljs.io/docs/usage/require/)

#### Hot Module Loading

As we did for the server we want to make sure we enable hot module loading programmatically. To do so, let’s rely on the hml toggle passed with our optionHash earlier:

{% highlight javascript %}
…
const hml = `
  System.trace = true
  window.hml = System.import('systemjs-hot-reloader/default-listener')
`
…
<script>
  ${ options.hml ? hml : ''}
  Promise.resolve(window.hml)
    .then(() => System.import('index.js'))
</script>
{% endhighlight %}

This will set system.trace to true and append the promise of having imported the systems-hot-reloaded module to the global window object. We can then check against the resolution of that promise, and in the case of hml not being active (window.hml == undefined), the promise will still resolve correctly.

Finally, we need to compose our middleware with the static middleware in order to allow our express server to serve the proper static assets. The final result will be something like:

{% highlight javascript %}
import express from 'express'
import {renderToString as render} from 'react-dom/server'
import React from 'react'
import App from '../app/app'

module.exports = (options) => {

  const serveApp = (req, res, next) => {
    if (req.url === options.endPoint) {
      const ssrApp = render(<App  options={options}/>)
      const hml = `
        System.trace = true
        window.hml = System.import('systemjs-hot-reloader/default-listener')
      `
      const html = `
        <!DOCTYPE html>
        <div id="app">${ssrApp}</div>
        <script src="jspm_packages/system.js"></script>
        <script src="config.js"></script>
        <script>
          ${ options.hml ? hml : ''}
          Promise.resolve(window.hml)
            .then(() => System.import('index.js'))
        </script>
      `
      res.status(200).send(html)
    }
    next()
  }
  
  const serveStatics = express.static('./app')

  return (req, res, next) => 
    serveStatics(req, res, () =>
      serveApp(req, res, next)
    )
}
{% endhighlight %}

### The React App

I won’t discuss any specific about the react app, for the sake of the experiment I’ll simply rely on a hello-world kind of app, you can take it further from there. What is important here is to simply rely on JSPM for all the dependencies, and the bundling. 

Finally, we can take advantage of the option we use to initialise the middleware to pass anything we may need to props one server side rendering the app:

{% highlight javascript %}
const ssrApp = render(<App  options={options}/>)
{% endhighlight %}

***

That’s it! We now have a generic middleware for express then can serve a react app, taking advantage of SSR and hot-module-loading workflow. Feel free to fork my [express-serve-app](https://github.com/nickbalestra/express-serve-app) repository and to run just:

{% highlight javascript %}
$npm i
$npm run dev // —> hot module loading ON
$npm start //  —> hot module loading OFF
{% endhighlight %}

And go to:

{% highlight javascript %}
http://localhost/boom // —> Shakalaka!
{% endhighlight %}