---
title: Installing npm packages global on OSX
date: 2015-10-04
description: Avoid using sudo when npm i -g
---

## Quick guide to install node and npm globally and avoid using sudo to install npm packages.

We'll be using the Z shell (zsh) instead of bash. Replace any reference to `.zshrc` with `.bash_profile/.bashrc` in case you are running on bash instead. 

* * *

## Install Node

We'll install node via homebrew. If you have Node installed already via brew, remove it, then:

{% highlight javascript linenos %}
$ brew install node --without-npm
{% endhighlight %}


## Install NPM

Create a directory for your global packages.

{% highlight javascript linenos %}
$ mkdir .npm-packages
{% endhighlight %}

Add a reference to the directory into `zshrc`


{% highlight javascript linenos %}
$ echo NPM_PACKAGES="${HOME}/.npm-packages" >> ${HOME}/.zshrc
{% endhighlight %}

Tell NPM where to install global packages (in our case ~/.npm-packages)

{% highlight javascript linenos %}
$ echo prefix=${HOME}/.npm-packages >> ${HOME}/.npmrc
{% endhighlight %}

Install the latest version of NPM

{% highlight javascript linenos %}
$ curl -L https://www.npmjs.org/install.sh | sh
{% endhighlight %}

Add the following lines to your .zshrc to be ensure that node will find the packages and that you'll find the installed binaries:

{% highlight javascript linenos %}
$ echo NODE_PATH=\"\$NPM_PACKAGES/lib/node_modules\:\$NODE_PATH\" >> ${HOME}/.zshrc
$ echo PATH=\"\$NPM_PACKAGES/bin\:\$PATH\" >> ${HOME}/.zshrc
{% endhighlight %}

To verify if everything installed successfully just run the following commands:

{% highlight javascript linenos %}
$ node -v 
// check version of node installed

$ npm -v to 
// check version of npm installed

$ npm list -g --depth=0
// check global npm packages installed (should just be npm for now)
{% endhighlight %}

Enjoy! You should now be able to install npm modules globally without having to use sudo!


