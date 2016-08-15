---
title: "Recursion, part 3: Tree recursion"
date: 2015-10-17
description: 'A workshop on recursion functions and processes'
---


This post is part of a series of posts in which I write about recursion. I'll Borrow some concepts from the [SICP](https://mitpress.mit.edu/sicp/) to talk about recursion in general, both in terms of functions and processes. I'll then explore some examples, define a general pattern and finally we'll get our hands dirty with a toy problem. All in JavaScript.

* * *


## Tree Recursion

Tree recursion is another common computational pattern. An often used example in this direction is computing the Fibonacci numbers, in which each number is the sum of the proceeding two:

> 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, ...

Let's see a recursive function to computing Fibonacci numbers, that will result in a tree recursive process:

{% highlight javascript %}
function fib(n) {
  if (n < 2) {
    return n;
  }
  return fib(n - 1) + fib(n - 2);
}
{% endhighlight %}

#### Visualizing the process

Similary to what we did before we can use the sobstitution model to visualize the process of computing fib(5) :

{% highlight javascript %}
// A tree recursive process for computing fib(5)

                                     fib(5)
                            /                    \
                     fib(4)                        fib(3)
                  /         \                    /        \  
            fib(3)           fib(2)           fib(2)       fib(1)
          /       \         /      \         /      \          |
       fib(2)     fib(1) fib(1)   fib(0)  fib(1) fib(0)        1
     /       \        |      |        |       |      |
fib(1)       fib(0)   1      1        0       1      0
    |            |
    1            0

// -> 5
{% endhighlight %}

While this function show us a typical tree recursion process is a terrible way to compute Fibonacci numbers, as it does a lot of rendundant calulculations. The process uses a number of steps that grows exponentially with the input. The space complexity on the other side only grows linearly because we need to keep track only of which nodes are above us in the tree at any point in the computation. In other words the space required is proportional to the maximum depth of the tree.

If you read my post on [DepthFirst and breadthFirst tree walking](http://nick.balestra.ch/2015/depthFirst-vs-breadthFirst/) you may notice that a Depth-First Search algorithm rely on a Stack datastructure. I'll leave it to the reader to find any parallelism with the call stack used by the interpreter to keep track of the deferred operations and why the space required in a tree recursion is proportional to the maximum depth of the tree...

We can also define a recursive function to compute the Fibonacci numbers that will instead generate a linear iterative process (as in ES6, thanks to its tail-call optimization):

{% highlight javascript %}
var fib = (n, a = 0, b = 1) => (n === 0) ? a : fib(n-1, b, a + b);
{% endhighlight %}

Now that we saw different recursive processes we may start noticing some pattern emerging in how we define the recursive function responsibles for such processes.


* * *

####All the recursion workshop articles:

- [Recursion, part 1: Linear Recursion and Iteration](http://nick.balestra.ch/2015/recursion-workshop)
- [Recursion, part 2: Tail-recursive processes in JS](http://nick.balestra.ch/2015/recursion-workshop-part2/)
- [Recursion, part 3: Tree recursion](http://nick.balestra.ch/2015/recursion-workshop-part3/)
- [Recursion, part 4: Anatomy of a recursion](http://nick.balestra.ch/2015/recursion-workshop-part4/)
