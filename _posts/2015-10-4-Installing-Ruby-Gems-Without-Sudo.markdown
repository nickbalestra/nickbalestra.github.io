---
title: Install ruby gems without sudo on OSX
date: 2015-10-04
description: Avoid using sudo when gem install 
---


## Quick guide to install latest version of ruby and avoid using sudo to install ruby gems.

If you encountered the following error when trying to install a ruby gem on MAC OSX, it's only because of some persmission related to how/where the ruby version that come packed OSX is installed. To fix it, is enough to rely on rbenv, a ruby version manager.

{% highlight javascript linenos %}

$ gem install jekyll
$ Fetching: liquid-2.6.3.gem (100%)
$ ERROR:  While executing gem ... (Gem::FilePermissionError)
{% endhighlight %}

We'll be using the Z shell (zsh) instead of bash. Replace any reference to `.zshrc` with `.bash_profile/.bashrc` in case you are running on bash instead. 

* * *

## Install rbenv

We'll install rbenv via homebrew:

{% highlight javascript linenos %}
$ brew install rbenv ruby-build
{% endhighlight %}

Add the following references to your `.zshrc`

{% highlight javascript linenos %}
$ echo 'if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi' >> ~/.zshrc
$ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.zshrc
$ echo 'eval "$(rbenv init -)"' >> ~/.zshrc
{% endhighlight %}


## Install Ruby

Install a more recent ruby version via rbenv 

{% highlight javascript linenos %}
$ rbenv install 2.2.3
{% endhighlight %}

Set the globa version to the latest installed version of ruby, i.e:

{% highlight javascript linenos %}
rbenv global 2.2.3
{% endhighlight %}


To verify if everything installed successfully just run the following commands:

{% highlight javascript linenos %}
$ ruby -v 
// check version of ruby installed
{% endhighlight %}


Enjoy! You should now be able to install ruby gems  without having to use sudo!


