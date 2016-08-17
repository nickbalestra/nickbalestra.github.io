---
title: Npm executables
date: 2016-08-17
description: and how to run them locally 
---

A great feature of Npm and node is the easiness to create executable standalone programs.

***

I personally try to avoid installing executables globally (it happen every time you run `npm i xxx -g`). Instead, I favour having explicit dependencies end purely rely on those. In such cases, to run executables from locally installed modules, we need to get the path to the node_modules/.bin, the folder where Npm install executables.

I.e: to run mocha locally, we would do something like:

{% highlight javascript %}
$ node_modules/.bin mocha
{% endhighlight %}

### Npm bin

Npm come with a bin command that simply return the folder where executables are installed. 

{% highlight javascript %}
$ $(npm bin)/mocha
{% endhighlight %}

We don’t have to worry for the relative path now, as we can run this command anywhere within the project dir.

### Npm scripts
If you find your self doing a similar thing within your Npm scripts, i.e.: 

{% highlight javascript %}
"scripts": {
  "test": "node_modules/.bin mocha"
}
{% endhighlight %}

Just be aware that Npm scripts will resolve to locally installed executables out of the box, so the above could simply be written as:

{% highlight javascript %}
"scripts": {
  "test": "mocha"
}
{% endhighlight %}





