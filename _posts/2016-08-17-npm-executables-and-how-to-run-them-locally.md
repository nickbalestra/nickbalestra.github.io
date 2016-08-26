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

***

### Conclusion
Try avoiding installing global packages as much as possible
If you find yourself having hard times typing $(npm bin)/whataver you can just install one global package: npm-run (a nicely written wrapper around npm bin)

{% highlight javascript %}
$ npm i -g npm-run
{% endhighlight %}

feel free to add an alias to your bashprofile, I personally like to have ‘nr’ so that I can simply type:

{% highlight javascript %}
$ nr jspm init // will run the locally resolved jspm bin executable like $(npm bin)/jspm init
{% endhighlight %}

Wondering what packages you have already installed globally? Simply type:

{% highlight javascript %}
$ npm list -g —depth=0 // leave out the depth flag to get scared
{% endhighlight %}



