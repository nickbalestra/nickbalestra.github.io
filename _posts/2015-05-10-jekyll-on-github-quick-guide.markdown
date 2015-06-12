---
title: Jekyll on Github - Quick guide
date: 2015-05-10
description: Static site generator for Github pages
---

##Jekyll is a simple static site generator, open source and written in Ruby by [Github](https://github.com/)'s co-founder [Tom Preston-Werner](https://twitter.com/mojombo).

***
Github fully support Jekyll, allowing for a great workflow and UX. This make the jekyll + github combo a good candidate for blogging. Some added benefits:

- Don't worry about hosting anymore thanks to Github pages.
- Write and work on your blog offline thanks to GIT.
- Take advantage of GIT Versioning + GITHUB PullReqiests.
- Use markdown and the editor of your choice to write your posts.
- Reduce switching costs thanks to a future proof Open Source Stack.

***
To set up your blog using jekyll and github pages just follow the next steps. You'll be up and running in seconds.

### Step 1: Fork a theme
Many great and free themes can be found on [jekyllthemes.org](http://jekyllthemes.org/).

- [Kactus](https://github.com/nickbalestra/kactus) - A port of Cactus’s  default theme. *The one I use for this site*.
- [Mediator](https://github.com/dirkfabisch/mediator) - A medium inspired Jekyll blog theme.
- [Pixyll](https://github.com/johnotander/pixyll) - A simple Jekyll theme that emphasizes content
- [Kasper](https://github.com/rosario/kasper) - A port of Ghost’s Casper default theme.<br><br>
Just pick the theme you prefere and fork it.<br><br>


### Step 2: Rename your repository
Rename the repository you just forked as ```username.github.io``` where username is your username on GitHub.

### Step 3 - Enjoy
There is no step 3, your new blog is now live and reachable at the following address:

{% highlight javascript linenos %}
http://username.github.io
{% endhighlight %}


##### Settings
To setup your blog, edit the ```_config.yml``` file.
You can also customize your theme by modifing any of the _layouts or _includes files.

##### Publishing
To publish a new article just add a new markdown file in the _posts directory, is that simple.

##### Testing locally
First make sure to have Jekyll installed.

{% highlight javascript linenos %}
$ gem install jekyll
{% endhighlight %}

and then you can simply run it from the terminal

{% highlight javascript linenos %}
jekyll serve
{% endhighlight %}

For more details check the [documentation](http://jekyllrb.com/docs/home/).
Alternatively, you could [install Jekyll via Github's own bundle](https://help.github.com/articles/using-jekyll-with-pages/). This will allow you to ensure that your computer most closely matches the GitHub Pages settings.


Check out the [Jekyll docs](http://jekyllrb.com) for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll's GitHub repo](https://github.com/mojombo/jekyll).

For more informations also check [Github Pages](https://pages.github.com/).
