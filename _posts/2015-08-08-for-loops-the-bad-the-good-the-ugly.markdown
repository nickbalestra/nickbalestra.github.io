---
title: "For* Loops: The Bad, The Good And The Ugly"
date: 2015-08-08
description:  "Javascript's control flow for* statements"
---

##A loop is a sequence of statements which is specified once but which may be carried out several times in succession. The code "inside" the loop is obeyed a specified number of times, or once for each of a collection of items, or until some condition is met, or indefinitely...meh.

There are 2+1 ways to do for* loops in javaScript. While each has its pros, in the following paragraphs we'll see which one is the one we should try to stick with and why.
* * *

## The Bad

{% highlight javascript linenos %}
for (var i = 0, len = nums.length; i < len; i++) {
  doSomethingWith(nums[i])
}
{% endhighlight %}

We all know him: it's a pain to type, a risk for our i's (unless we rely on ES6 and use 'let' within the for signature when declaring the index variable) and a scope polluter in general. The for loop , also known as the 'for semicolon' loop, is definitely the most common used control flow for looping in js

***

## The Good

{% highlight javascript linenos %}
nums.forEach(function(number){
  doSomethingWith(number)
})
{% endhighlight %}

When I said that "there are 2+1 ways to do for* loops in javaScript" I was referring to forEach(). This higher-order-function-way of looping is available on almost any js environment as a native Array method (and it comes together with some awesome bros like map() and reduce()).

Why is it The Good? First it relieves us from The Bad's pains, Second it brings moar awesomeness to the table.

When looping using higher order functions the lexical scope work to our advantage . Because the inner function lives inside the environment of the outer one, it can access its closure scope. The bodies of such inner functions can then access the variables around them. They can play a role similar to the {} blocks used in regular loops and conditional statements, with an important difference: variables declared inside inner functions do not end up in the environment of the outer function. And that is usually a good thing.

For those reasons you will never be trapped by the infamous async loop nightmare, aka:

{% highlight javascript linenos %}
var nums = [1, 2, 3];

// The Bad vs Async
for (var i = 0, len = nums.length; i < len; i++) {
  setTimeout(function() {
    console.log(nums[i]);
  }, i * 1000);
}
// -> undefined logged 3 times

// The Good vs Async
nums.forEach(function(number, i){
  setTimeout(function() {
    console.log(number);
  }, i * 1000);
})
// -> 1 2 3
{% endhighlight %}

Lastly, by being more expressive it represents our intent to iterate through each element of an array, allowing us to abstract over actions, not just values.This allow us to properly name such intent instead of having to rely on meaningless indexes:

{% highlight javascript linenos %}
nums.forEach(function(number){
  doSomethingWith(number)
})
{% endhighlight %}

Yay, that's good!

***

##  The Ugly

{% highlight javascript linenos %}
for (var number in nums) {
  doSomethingWith(nums[number]);
}
{% endhighlight %}

For in is the ugly one. I bet you can now easily spot all the negative aspects we just encountered earlier for its counterpart for semicolon, like lack of expressiveness and scope pollution. For in is ugly-evil too as it will loop through prototypal properties as well. This is ok if your intent is explicitly to loop trough the prototypal chain properties as well, but that's rarely the case. The underscore method allKeys does just that, as its name clearly communicate...

{% highlight javascript linenos %}
_.allKeys = function(object){
  var keys = [];
  for(var key in object) {
    keys.push(key)
  }
  return keys;
}
{% endhighlight %}

Another way to avoid using The Ugly for in loop to iterate trough your objects key/value pairs is by using Object.keys(obj) to build an array of its own properties and than use that array with the Bad for semicolon loop...cause bad is better then ugly.

***
##Final thoughts

It's always worth knowing with who are we dealing with: the bad the good or the ugly. We should also always strive, wherever possible for the good, meaning relying on a functional library like underscore, lodash or ramda. [Read more about functional libraries](http://nick.balestra.ch/2015/javascript-functional-libraries/) and why you should care.

You can also implement your own little higher order function to iterate through lists and collections if you believe that a whole library will just be an overkill dependency. If that's the case I strongly suggest you to just implement your own each() and reduce() functions, as everything else, like map() filter() etc can be implemented on top of those two extremely powerful workhorses.
