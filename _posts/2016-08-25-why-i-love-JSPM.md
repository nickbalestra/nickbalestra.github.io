---
title: "Why I love JSPM"
date: 2016-08-25
description: and why you should give it a go
---

Last year I wrote on getting [up and running with jspm](nick.balestra.ch/2015/up-and-running-with-jspm-for-react/). 
So, why another post about it? As Webpack seems the current status-quo for bundling, I wanted to simply share my thoughts on why I generally prefer [JSPM](https://jspm.io/) over Webpack and why you might want consider giving it a go.

***

### Fatigueless browser package management, and more.

You probably heard and read about the [JavaScript fatigue](https://medium.com/@ericclemmons/javascript-fatigue-48d4011b6fc4): Shortly: Too many tools. Too Much Configuration. I specifically feel js-fatigued anytime I have to wire building/transpiling/boundling tools, specifically: webpack. What if we could rely on a simple node-like process? Say hello to JSPM:

1. Install - `jspm install react`
2. Require - `import React from 'react'`
3. Boom! There is no point 3. Go building things!<br><br>

Yes, you are right, I was cheating, to get to that point we need to have jspm installed and configured. I assume we want to use JSX, Babel for transpiling, and all the ES6 cowabungas… easy:
<br><br>
{% highlight javascript %}
1 - $ npm i -D jspm
2 - $ $(npm bin)/jspm init // WTF? -> goo.gl/Ni1Ga8
3 - Boom! there is no point 3, again :P 
{% endhighlight %}

### Using ES6 Modules Today

We have all been writing [ES6 modules](http://wiki.ecmascript.org/doku.php?id=harmony:specification_drafts) for a while now, but for what? It is possible to use them directly in all browsers today using the [ES6 loader polyfill](https://github.com/ModuleLoader/es-module-loader). JSPM rely on the SystemJS universal module loader, that is built on top of it. Simply use systemjs to import your app.

{% highlight javascript %}
<script src="jspm_packages/system.js"></script>
<script>System.import(‘your-app’)</script>
{% endhighlight %}


### Load any module format, from any registry.

No need to publish an npm module when you could simply load it from github. With JSPM you can load any module format (ES6, AMD, CommonJS and globals) directly from any registry such as npm and GitHub.

{% highlight javascript %}
$ jspm install npm:your-module
$ jspm install github:your-module 
{% endhighlight %}

### Don’t make me think production/development workflows

Loading modules as separate files with ES6 and plugins compiled in the browser may be cool but definitely something you would rather want for development only, so what about production workflows? JSPM optimize modules into a bundle, layered bundles or a self-executing bundle with a single command:

{% highlight javascript %}
$ jspm bundle app/main build.js
{% endhighlight %}

Couple of options are available: 

#### Automatically loading bundle (—inject). 
If you don't want to include the bundle with a script tag, but rather load it only when it is needed:

{% highlight javascript %}
$ jspm bundle app/main main-bundle.js --inject
{% endhighlight %}

#### Make bundles with arithmetic, yo!
Creates a file `build.js` containing `app/main` and `moment` and all their dependencies, excluding `react` and all its dependencies:

{% highlight javascript %}
$ jspm bundle app/main - react + moment build.js
{% endhighlight %}
 
#### Self-executing bundles

{% highlight javascript %}
$ jspm bundle-sfx app/main.js app.js
{% endhighlight %}

For more advanced options, check http://jspm.io/docs/production-workflows.html

***

Those are the biggest reasons that make me favour jspm over webpack, If you want to play around with JSPM, feel free to fork my [jspm-playground repository](https://github.com/nickbalestra/jspm-playground) it comes with branches for:

- getting started
- css modules
- hot reloading
- production workflows
