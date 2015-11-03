---
title: Jspm and React
date: 2015-11-2
description: Setting up jspm for developing with react.
---

##[Jspm](https://jspm.io/) is a package manager for the SystemJS universal module loader, built on top of the dynamic ES6 module loader.

In my last post, I walked through the process of [setting up webpack for react](http://nick.balestra.ch/2015/up-and-running-with-webpack-for-babel-and-react/) development.  This time I'll show an alternative with jspm that come with few advantages:

- Write code and run it without a building step.
- Babel comes with JSPM.
- Easily installs React component.
- ES6 modules support other extensions other then .js (like .jsx)

***

### Installing jspm

Install jspm

{% highlight javascript linenos %}
$ npm install -g jspm
{% endhighlight %}

Initialize jspm for the project

{% highlight javascript linenos %}
$ jspm init
{% endhighlight %}

Add jsx to extend the es6 module loader to support .jsx files

{% highlight javascript linenos %}
$ jspm install jsx
{% endhighlight %}

Pull react and react-dom via jspm

{% highlight javascript linenos %}
$ jspm install react
$ jspm install react-dom
{% endhighlight %}

That's it, no configuration needed, jspm will do its magic for us.
We can now just require and use react to build ...
***

### A simple app

Let's imagine that we have the following javascript file:

- index.js
<br><br>
It doesn't do much, apart rendering a header on the page using react-dom.
It has react and react-dom as dependencies ,and use ES6 and JSX syntax.


<br>{% highlight javascript linenos %}
import React from 'react';
import ReactDOM from 'react-dom';

class Home extends React.Component {
  constructor() {
    super()

    this.state = {
      subject: 'World!'
    }
  }

  render () {
    return <h1>Hello {this.state.subject}</h1>
  }
}

ReactDOM.render(<Home />, document.getElementById('app'));
{% endhighlight %}

Note: Instead of relying on React.createClass, we rely on the ES6 classes syntax.
We can now require the app from a HTML file:



{% highlight javascript linenos %}
<!doctype html>
<html>
<head>
<meta charset="utf-8">
<title>Home</title>  
</head>
<body>
<div id="app"></div>
<script src="jspm_packages/system.js"></script>
<script src="config.js"></script>
<script>
    System.import('../app/index').catch(console.error.bind(console))
</script>
</body>
{% endhighlight %}

That's it, just load the html in the browser and enjoy!

***

## Some jspm extras.

Jspm doesn't come with a dev-server as the one you can find in webpack.
No worry, you can quickly install the jspm-server and have it serve your app within seconds live reload included!

{% highlight javascript linenos %}
$ jspm i -g jspm-server
{% endhighlight %}

Make sure to set the System trace property to true from the html file to enable live reloading via web sockets.

{% highlight javascript linenos %}
..
<script>
    System.trace = true;
    System.import('../app/index').catch(console.error.bind(console))
</script>
...
{% endhighlight %}

***

[Download the code of this post](https://github.com/nickbalestra/jspm-react)
