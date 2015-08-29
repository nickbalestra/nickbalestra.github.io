---
title: "Case based Macros for javaScript"
date: 2015-08-29
description:  "Crafting javaScript Syntax without transpiling"
---

##[Sweet.js](http://sweetjs.org/) allow to change the syntax of JavaScript via Scheme-inspired hygienic macros. Based on plain 'ol javascript, If compared to transpiling alternatives, it favors composition over inheritance.

Recently for a project I had the pleasure to work with the Mozilla sweet.js project. In this post I'll go through the basics of sweet.js's case based macros.

* * *

## Macros

So, what is a macro? Macros give you snippets in the programming language that allow you to augment the way the language works. You can think of macros as powerful templates that can expand inline over some matching patterns. Sweet.js support different kind of macros. Let's quickly explore the differences between a rule base macro and a case based macro.


### Rule Based Macros : **MACRO > OUTPUT**

{% highlight javascript linenos %}
macro <name> {
  rule { <pattern> } => { <template> }
}
{% endhighlight %}
Our rule base macro definition contain a pattern and a template that is output excactly as it was written inside the macro definition. For example an identity rule bases macro definition will look something like:
{% highlight javascript linenos %}
macro id {
  rule { $x } => { $x }
}
// writing id 1 will result in
// a macro invokation that will compile to
// 1;
{% endhighlight %}

### Case Based Macros: **MACRO > CODE-EXEC > OUTPUT**

{% highlight javascript linenos %}
macro <name> {
  rule { <pattern> } => {
    return #{ <template> }
  }
}
{% endhighlight %}

The main difference is that case based macros execute code before returning the template, meaning they
have their own lexical scope and allow for more complex logic. So if we compare with the id rule base macro we saw above, in sweet js we will re-write the id macro as following:

{% highlight javascript linenos %}
macro id {
  rule { _ $x } => {
    return #{ $x }
  }
}
// writing id 1 will result in
// a macro invokation that will compile to
// 1;
{% endhighlight %}

Well this doesn't show much of the potential of a case base macro, so let's change this id macro into a randomid macro that will generate a random id;

{% highlight javascript linenos %}
macro rid {
  rule { _ $x } => {
    var r = Math.random();
    letstx $r = [makeValue(r, #{$x})];
    return #{ var $x = $r }
  }
}
// writing rid x; will result in
// a macro invokation that will compile to
// var x = 0.8185508847236633;
{% endhighlight %}

***
##Final thoughts

Macros are very powerful ways to extend the syntax of a language. Learn more about them, and how to use sweet.js:

- [Sweet.js official documentation](http://sweetjs.org/doc/main/sweet.html) - By Mozilla
- [Rule based macros? That's sweet.js!](http://lukesavage.me/technical/2015/08/29/sweetjs-and-rule-based-macros/) - By Luke Savage
- [Holy macro-roni!](http://gregrv.github.io/eqex/2015/08/29/holy-macro-roni.html) - By Greg Varias
- [Sweet.js Losing your hygienity](http://zachsebag.com/2015/08/29/sweet.js-losingyourhygienity.html) - By Zach Sebag
