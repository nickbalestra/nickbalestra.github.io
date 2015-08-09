---
title: For Loops in js
date: 2015-08-08
description:  The Bad, The Good and The Ugly
---

##A loop is a sequence of statements which is specified once but which may be carried out several times in succession. The code "inside" the loop is obeyed a specified number of times, or once for each of a collection of items, or until some condition is met, or indefinitely...sigh

* * *

### The Bad

We all know him, is a pain to type, a risk for our declared i's, but probably the most
common used control flow for looping in js.

{% highlight javascript linenos %}
for (var i = 0; i < list.length; i++) {
  doSomethingTo(list[i])
}
{% endhighlight %}

### The Good

We never use forEach() enough, is a native method available in almost any js environmnt
it allow to focus on what we do instead of wasting times and energy declaring how to do it,
it come packed with closure scope for free, so you will never be trapped by the infamous async loop nightmare, aka:

{% highlight javascript linenos %}
var nums = [1, 2, 3];
for (var i = 0; i < nums.length; i++) {
  setTimeout(function() {
    console.log(nums[i]);
  }, i * 1000);
}
{% endhighlight %}

Plus readability and legibility is so much more awesome with forEach:

{% highlight javascript linenos %}
list.forEach(function(element){
  doSomethingTo(Element)
})
{% endhighlight %}


###  The Ugly

Yes, for in is ugly, not because how it looks, it actually look quite cleaner then it's
cousin for;; but can be ugly-evil as it will loop through prototypal properties as well. To avoid any risk, use it only when you want such behaviour ie the _.allKeys method of underscore, where all Keys will mean ALLKEYS:

{% highlight javascript linenos %}
_.allKeys = function(object){
  var keys = [];
  for(var key in object) {
    keys.push(key)
  }
  return keys;
}
{% endhighlight %}

You can also avoid it by relying on Object.keys(obj) or as suggest earlier, with an higher order function like each, that may also accept objects , [see my article](http://nick.balestra.ch/2015/javascript-functional-libraries/) on ramda, lodash, underscore
