---
title: Installing and managing Node versions with n
date: 2015-10-26
description: Quick tutorial to get up and running with n
---

We'll be using the Z shell (zsh) instead of bash. Replace any reference to `.zshrc` with `.bash_profile/.bashrc` in case you are running on bash instead. 

* * *


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

## Install n

We'll install node via [n](https://github.com/tj/n): a Node version management tool. If you have Node installed already via brew, remove it, then:

{% highlight javascript linenos %}
$ npm i -g n
{% endhighlight %}

Install a version of node (ie the latest stable)

{% highlight javascript linenos %}
$ n stable
{% endhighlight %}


To verify if everything installed successfully just run the following commands:

{% highlight javascript linenos %}
$ node -v 
// check version of node installed

$ npm -v to 
// check version of npm installed

$ npm list -g --depth=0
// check global npm packages installed (should just be npm and n for now)
{% endhighlight %}

Enjoy! You should now be able to install and manage different node versions on your machine. [Check the n documentation](https://github.com/tj/n)!


