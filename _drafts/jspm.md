---
title: "Why I love JSPM"
date: 2016-08-14
description: and why you should give it a go
---

Last year I wrote on getting [up and running with jspm](nick.balestra.ch/2015/up-and-running-with-jspm-for-react/). 
So, why another post about it? As Webpack seems the current status-quo for package management, I wanted to simply share my thoughts on why I generally prefer [JSPM](https://jspm.io/) over webpack and why you should give it a go.

***

If you were just looking for a quick and minimal starter setup, go ahead and [fork my jspm-react starter kit](https://github.com/nickbalestra/jspm-react-starter-kit), otherwise continue reading as I'll walk you trough that setup.

### Installing jspm

Install jspm

{% highlight javascript %}
$ npm install -g jspm
{% endhighlight %}

Initialize jspm for the project

{% highlight javascript %}
$ jspm init
{% endhighlight %}

Add jsx to extend the es6 module loader to support .jsx files

{% highlight javascript %}
$ jspm install jsx
{% endhighlight %}

Pull react and react-dom via jspm

{% highlight javascript %}
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


<br>{% highlight javascript %}
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



{% highlight javascript %}
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

{% highlight javascript %}
$ jspm i -g jspm-server
{% endhighlight %}

Make sure to set the System trace property to true from the html file to enable live reloading via web sockets.

{% highlight javascript %}
..
<script>
    System.trace = true;
    System.import('../app/index').catch(console.error.bind(console))
</script>
...
{% endhighlight %}

***

To help you kickstart your development with react using JSPM fork the [jspm-react starter kit](https://github.com/nickbalestra/jspm-react-starter-kit)
