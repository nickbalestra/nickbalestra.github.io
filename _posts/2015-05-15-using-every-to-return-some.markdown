---
title: Using every() to return some()
date: 2015-05-15
description: Functional programming helpers
---

##[Underscore](http://underscorejs.org/) is a JavaScript library that provides a whole mess of useful functional programming helpers without extending any built-in objects.

Rewriting some of its core methods is a very fun and useful excercise to better understand it while getting more comfortable with some of the functional programming paradigms. One fun excercise worth trying is re-using functions as much as possible. For example, there is a very clever way to re-use every() inside of some().

* * *

### Every()

It accept a collection and an iterator as a test function. It should return true if every element of the collection pass the test.

### Some()
Similarly to every(), it accept a collection and a test iterator, and, as the name suggest:

> It should return true if at least one element of the collection pass the test.


We can slightly rephrase that definition into something like:

> some() should return true if not every element of the collection fail the test

Can you see now how every() made it through that some() definition? Let's pseudo code this further:

{% highlight javascript linenos %}
some = function(collection, test) {
        // not every element of the collection
        return !every(collection, function(item){
            // fail the test
            return !test(item);
        });
    });
}
{% endhighlight %}

If you have every() at you disposal and you need to get some() this is definitely a short and smart way to get there (For a slighltly longer solution use reduce() instead).


For a better break-down and for an extensive example on using every() to implement some(), read the post on [The Importance of Clear Communication](http://hmfoster.github.io/The-Importance-of-Clear-Communication/) by Hailey Foster.





