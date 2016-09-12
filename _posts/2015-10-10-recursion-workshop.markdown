---
title: "Recursion, part 1: Linear Recursion and Iteration"
date: 2015-10-17
description: 'A workshop on recursion functions and processes'
---


This post is part of a series of posts in which I write about recursion. I'll Borrow some concepts from the [SICP](https://mitpress.mit.edu/sicp/) to talk about recursion in general, both in terms of functions and processes. I'll then explore some examples, define a general pattern and finally we'll get our hands dirty with a toy problem. All in JavaScript.

* * *


## Linear Recursion and Iteration

Let's start by implementing a function that will allow us to calculate the sum of two numbers without relying on the `+` operator.

For the sake of the example let's assume we have two functions that we can rely on: inc() wich increments its argument by 1 and dec(), which decrements its argument by 1:

{% highlight javascript %}
var inc = (n) => ++n;
var dec = (n) => --n;
{% endhighlight %}

We could define the sum of two numbers as the increment of the sum of the first number decremented and the other number. That sounds more complex then it is...
Let's see some code about this way to define sum in term of inc and dec:

{% highlight javascript %}
function sum(a, b) {
  if (a == 0) {
    return b;
  }
  return inc(sum(dec(a), b));
}
{% endhighlight %}



#### Visualizing the process

We can use the sobstitution model to watch this function in action computing `4 + 5` :

{% highlight javascript %}
// A linear recursive process for computing 4 + 5

sum(4, 5)
inc(sum(3, 5))
inc(inc(sum(2, 5)))
inc(inc(inc(sum(1, 5))))
inc(inc(inc(inc(sum(0, 5)))))
inc(inc(inc(inc(5))))
inc(inc(inc(6)))
inc(inc(7))
inc(8)

// -> 9
{% endhighlight %}

Now let's take a different perspective on computing sum. We could describe sum by specifying that we first decrement A and incremen B and then sum them together, and so on until we reach 0 + 9. Therefore, another possible, but still valid, way to define sum in term of inc and dec is:

{% highlight javascript %}
function sum(a, b) {
  if (a == 0) {
    return b;
  }
  return sum(dec(a), inc(b));
}
{% endhighlight %}

#### Visualizing the process

As before we can use the sobstitution model to visualize the process of computing `4 + 5` :

{% highlight javascript %}
// A linear iterative process for computing 4 + 5

sum(4, 5)
sum(3, 6)
sum(2, 7)
sum(1, 8)
sum(0, 9)

// -> 9
{% endhighlight %}

Compare the two processes. For the first implementation the substitution model reveals a shape of expansion followed by contraction. The expansion occurs as the process builds up a series of deferred operations (in this case a chain of increments). The contraction occurs as the operations are actually performed. This type of process, is called recursive process. The interpreter need to keep track of the operations that will need to be performed. Such amount of information grows linearly as the amount of steps needed to complete the process grows, this is why we refer to such process as linear recursive process.

By contrast the second process doesn't grow and shrink. At each step the interpreter only need to keep track of the current value of A and B. Such a process is called iterative process. Bacause this process also grow linearly compared to the steps needed, we call it linear iterative process.

This contrast between the two processes affect space complexity, as the memory needed to be allocated for the first process, the linear recursive process, grow linearly at each step, while stay constant in the case of a linear iterative process. If it happen that your interpreter run out of memory and you may encounter the following error message: 'Maximum call stack size exceeded', now you know why.

By now, you may be wondering why the word recursive only appear on the first process and not on the second and we instead call it iterative. We must be careful not to confuse the notion of recursive process with the notion of recursive function. When we describe a function as recursive we are referring to its syntax, or in other words, that the function definition refer to the function itself. When, instead, we describe a process to be, for example, linerly recursive, we are referring to how the process unfold and evolve over time, not how the syntax of the function that generates it was written.

* * *

#### All the recursion workshop articles:

- [Recursion, part 1: Linear Recursion and Iteration](http://nick.balestra.ch/2015/recursion-workshop)
- [Recursion, part 2: Tail-recursive processes in JS](http://nick.balestra.ch/2015/recursion-workshop-part2/)
- [Recursion, part 3: Tree recursion](http://nick.balestra.ch/2015/recursion-workshop-part3/)
- [Recursion, part 4: Anatomy of a recursion](http://nick.balestra.ch/2015/recursion-workshop-part4/)
