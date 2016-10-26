---
title: "E2: Elm + Express for full-stack web development"
date: 2016-10-26
description: A Simple, Scalable and Easy starting point
---

## [Elm](http://elm-lang.org/) is a delightful purely functional language for the browser. [Express](https://expressjs.com/) is a fast, unopinionated, minimalist web framework for Node.js. Put together they are a simple, scalable and easy starting point for full stack web development.

Following [Christian Alfoni](https://twitter.com/christianalfoni)'s example with his [ultimate Webpack setup](https://github.com/christianalfoni/webpack-express-boilerplate), the following post to share with you a really awesome setup to start developing full stack web application using the E2 stack (Elm + Express).

**Tl:dr: [github.com/nickbalestra/E2](https://github.com/nickbalestra/E2)**

{% highlight bash %}
$ git clone https://github.com/nickbalestra/E2.git
$ cd E2
$ npm install
$ npm start
$ open http://localhost:3000
{% endhighlight %}

***

## File structure

Before we get started let's get an overview of the directory we are going to work with:

{% highlight bash %}
E2/
├── server/
│   ├── main.js
│   ├── routes.js
│   └── webpackServeBundle.js
├── src/
│   ├── elm/
│   │   └── Main.elm
│   └── static/
│       ├── styles/
│       ├── index.html
│       └── main.js
├── tests/
├── .gitignore
├── .node-version
├── README.md
├── elm-package.json
├── package.json
└── webpack.config.js
{% endhighlight %}

## The Express server

If you are just going to use the Node server as a development tool for prototyping or actually run it in production you will need something to handle the requests from the browser. Express is great for that so lets go ahead and set up the server:

server/main.js

{% highlight javascript %}
const path = require('path')
const express = require('express')
const app = express()
const isDevelopment = process.env.NODE_ENV !== 'production'
const port = isDevelopment ? 3000 : process.env.PORT

// API endpoints
const routes = require('./routes')
app.set('json spaces', 2)
app.get('/color', routes.color)

// Serving compiled elm client
if (isDevelopment) {
  require('./webpackServeBundle')(app)
} else {
  app.use(express.static(path.join(__dirname, '/../dist')))
  app.get('*', (req, res) =>
    res.sendFile(path.join(__dirname, '/../dist/index.html'))
  )
}

// Starting express
if (!module.parent) {
  app.listen(port, err => {
    if (err) console.log(err)
    console.log(`⚡  Express started on port ${port}`)
  })
}
{% endhighlight %}

Two things are worth noticing here:

First we structured our routes following the [routes separarion pattern](https://github.com/expressjs/express/blob/master/examples/route-separation), allowing us to easily extend and structure our API as we develop the project further. A quick look into out routes file will show that at the moment the server expose a single endpoint 'color':

{% highlight javascript %}
exports.color = (req, res) => {
  const elmColors = [
    '#5A6379',
    '#5CB5CD',
    '#F2AE00',
    '#7CD32B'
  ]
  const random = Math.floor(Math.random() * elmColors.length)
  return res.json(elmColors[random])
}
{% endhighlight %}

Second, we made our server behaving differenly in order to support both dev and production environments.

{% highlight javascript %}
...
// Serving compiled elm client
if (isDevelopment) {
  require('./webpackServeBundle')(app)
} else {
  app.use(express.static(path.join(__dirname, '/../dist')))
  app.get('*', (req, res) =>
    res.sendFile(path.join(__dirname, '/../dist/index.html'))
  )
}
...
{% endhighlight %}

If we are in development mode, we require the webpackServeBundle.js and initialize it by passing our express app. As the name suggest, it will handle configuring webpack, registering the usage of the webpackMiddleware and webpackHotMiddleware middlewares, so that webpack will compile and bundle our elm application in memory, serving it directly the in-memory build once we request the app trhough our browser.

Else, if in the production environment, we simply serve the app located in the dist folder that will be created once we run `npm run build`.

### Development workflow
It come with Hot reload enabled, to get it up and running simply type in your terminal:

{% highlight javascript %}
$ npm start
$ open http://localhost:3000.
{% endhighlight %}


Now start editing any file in ./src (i.e .elm source files or sass/css files).

### Production workflow

{% highlight javascript %}
$ npm run build
$ npm run prod
$ open http://localhost:3000
{% endhighlight %}


This will:
- compile elm
- compile sass, extract css applying prefixer on postprocessing
- copy all the related bundles and files into a /dist directory.
- start the server serving the production-ready assets


## The Elm Client

A basic elm app structured around the Elm Architecture (Model-View-Update)hit the server on the http://localhost:3000/color endpoint (Which as we seen return a random Elm color in the exadecimal format). It then uses the returned color to simply set the background color of the whole app.

{% highlight javascript %}
view : Model -> Html Msg
view model =
    div [ class "container", style [ ( "height", "100%" ), ( "background-color", model.color ) ] ]
        [ div [ class "actions" ]
            [ button [ class "btn", onClick ChangeColor ]
                [ span [] [ text "Elm, Gimme Colors!" ] ]
            ]
        ]
{% endhighlight %}

***

As [Elm-format](https://github.com/avh4/elm-format) will take care of everything related to elm. For anything javascript related we relied on standardJS. On npm test standard will check every .js file inside the /server and /src directories making sure they all comply to the standard javascript style.

***

Please feel free to [fork the repo](https://github.com/nickbalestra/E2) and create your own boilerplate for prototyping using Elm and Express!
