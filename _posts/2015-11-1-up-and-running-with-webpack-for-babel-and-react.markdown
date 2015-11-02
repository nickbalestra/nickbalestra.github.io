---
title: Up and running with Webpack for Babel and React
date: 2015-11-1
description: Setting up Webpack as your build tool.
---

##[Webpack](https://webpack.github.io/) is a module bundler. It takes modules with dependencies and emits static assets representing those modules.

If you rely on a build tool with your development team (and you definitely should), most probably you are relying on Make, Grunt or Gulp. But when it come to bundling assets and dependencies for the browser, and you want to rely on javascript modules (either following the commonJS, AMD, or ES6 module pattern), simply concatenating JavaScript files won't be that easy anymore. Luckily we have tools that help us in this direction: [Browserify](http://browserify.org/), [Webpack](https://webpack.github.io/) and [JSPM](http://jspm.io/). In this post, we'll go over setting up Webpack for developing with react using [Babel](http://babeljs.io/) to transpile [ES6](http://www.ecma-international.org/ecma-262/6.0/) and [JSX](https://facebook.github.io/jsx/) code.

If all of that still confusing to you, read surviveJS's article ['Webpack compared']( http://survivejs.com/webpack_react/webpack_compared/) to understand Webpack better by seeing it explained into a historical context.

***

### A simple two pages app scenario

Let's imagine that inside an /app folder we have two javascript files:

- Profile.js
- Home.js
<br><br>
Those two apps don't do much, apart rendering the respective header on the page using react-dom.
Both have react and react-dom as dependencies ,and both use ES6 and/or JSX syntax.

Home.js:

{% highlight javascript linenos %}
import React from 'react';
import ReactDOM from 'react-dom';

var Home = React.createClass({
  render () {
    return <h1>Home</h1>
  }
});

ReactDOM.render(<Home/>, document.getElementById('app'));
{% endhighlight %}

and Profile.js

{% highlight javascript linenos %}
var React = require('react');
var ReactDOM = require('react-dom');

var { h1 } = React.DOM;

var Profile = React.createClass({
  render () {
    return h1({}, "Profile");
  }
});

ReactDOM.render(<Profile/>, document.getElementById('app'));
{% endhighlight %}

Note: Just for the sake of illustrating some example variations, Home.js rely on a JSX syntax while Profile.js not. Home.js rely on the ES6 module pattern while Profile.js uses the CommonJS one.

***

### Defining npm dependencies

Before being able to configure webpack we want to make sure that we have everything we need.
We therefore define our dependencies and dev-dependenciesInside package.json:

{% highlight javascript linenos %}
"dependencies": {
  "react": "^0.14.1",
  "react-dom": "^0.14.1"
},
"devDependencies": {
  "babel-core": "^6.0.14",
  "babel-loader": "^6.0.0",
  "babel-preset-es2015": "^6.0.14",
  "babel-preset-react": "^6.0.14",
  "webpack": "^1.12.2"
}
{% endhighlight %}

***

### Configuring webpack

Webpack looks for a file named webpack.config.js in the root of our project. In our case we could have something like:

{% highlight javascript linenos %}
var webpack = require('webpack');

var plugins = [
  new webpack.optimize.CommonsChunkPlugin('public/shared.js'),
];

module.exports = {
  entry: {
    Home: './app/Home.js',
    Profile: './app/Profile.js'
  },

  output: {
    filename: 'public/[name].js'
  },

  plugins: plugins,

  module: {
    loaders: [
      {test: /\.js$/, loader: "babel", query: {presets:['react', 'es2015']}}
    ]
  }
};
{% endhighlight %}

Let's walk through it:

{% highlight javascript linenos %}
var webpack = require('webpack');
{% endhighlight %}

We require all the modules we may need to build our configuration file.

{% highlight javascript linenos %}
var plugins = [
  new webpack.optimize.CommonsChunkPlugin('public/shared.js'),
];
{% endhighlight %}

The CommonsChunkPlugin allows us to build all the shared common javascript modules into a separate single js file. In this case, we'll want to output a file named shared.js inside the public directory. At this moment webpack still won't do anything with it, as the only thing that it will look for is what come within its exports, in other words, the webpack.config.js is nothing more then a js module itself:

{% highlight javascript linenos %}
module.exports = {
  ...
}
{% endhighlight %}

We expose the configuration following the export module pattern

{% highlight javascript linenos %}
entry: {
  Home: './app/Home.js',
  Profile: './app/Profile.js'
},

output: {
  filename: 'public/[name].js'
},
{% endhighlight %}

We export a reference to the entries that we want to bundle telling webpack to compile a different file for each of them in the public directory.

{% highlight javascript linenos %}
plugins: plugins,
{% endhighlight %}

Exporting the plugin array we declared earlier.

{% highlight javascript linenos %}
module: {
  loaders: [
    {test: /\.js$/, loader: "babel", query: {presets:['react', 'es2015']}}
  ]
}
{% endhighlight %}

We make sure to tell webpack to use the babel-loader for any .js file to correctly transipile our files. As we rely on the latest Babel 6 that come with presets plugin, we want to make sure to pass the ones we need as a query object. [Learn more about Babel 6](http://babeljs.io/blog/2015/10/29/6.0.0/).

If we did everything right, we could now run webpack in the terminal (make sure you installed webpack globally first)

{% highlight javascript linenos %}
$ npm i -g webpack
$ webpack
{% endhighlight %}

and we should find 3 new files builded for us in the public directory:

- shared.js
- Home.js
- Profile.js
<br><br>

We can now require those final assets from their respective HTML files (home.html and profile.html):

home.html

{% highlight javascript linenos %}
<!doctype html>
<html>
<meta charset="utf-8">
<title>Home</title>
<div id="app"></div>
<script src="./shared.js"></script>
<script src="./Home.js"></script>
{% endhighlight %}

profile.html

{% highlight javascript linenos %}
<!doctype html>
<html>
<meta charset="utf-8">
<title>Profile</title>
<div id="app"></div>
<script src="./shared.js"></script>
<script src="./Profile.js"></script>
{% endhighlight %}

***

## Some webpack extras.

While you can run webpack with the --watch flag so that webpack watches all dependencies and recompile on change, you can go one step further and install the [WEBPACK DEV SERVER](https://webpack.github.io/docs/webpack-dev-server.html) that will run a little express server for you serving the webpack bundle.

Enabling some extra plugins to optimize code for production:

{% highlight javascript linenos %}
var plugins = [
  new webpack.optimize.CommonsChunkPlugin('public/shared.js'),
  new webpack.DefinePlugin({
    'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV)
  })
];

if (process.env.NODE_ENV === 'production') {
  plugins.push(new webpack.optimize.DedupePlugin());
  plugins.push(new webpack.optimize.UglifyJsPlugin());
}
{% endhighlight %}

By adding the Dedupe and UglifyJs plugins we can optimize our bundles from:

- public/Home.js  773 bytes
- public/Profile.js  551 bytes
- public/shared.js     675 kB


<br>to

- public/Home.js  325 bytes
- public/Profile.js  207 bytes
- public/shared.js     133 kB

<br>Read more about [optimizations with webpack](https://github.com/webpack/docs/wiki/optimization)
***

## Final thoughts and resources.

Webpack is an awesome tool that may not solve anything, but it does solve the difficult problem of bundling.
Coupled with react and babel makes it a very good development tool.

[Download the code of this post](https://github.com/nickbalestra/webpack-react)
