---
title: JavaScript swap in-place array elements
date: 2015-11-22
description: From bitwise operators to ES6 destructuring
---

No matter what sorting algorithm we are using, it's more likely than not that we'll need to rely on some swapping techniques, let's explore a couple of in-place swapping helpers.

![index page](https://raw.githubusercontent.com/nickbalestra/nickbalestra.github.io/master/assets/images/swap-in-place.png)

* * *

### Memo swap

The memo swap technique relies on a variable to temporarily hold one of the two values that we want to swap.

{% highlight javascript %}
function memoSwap(list, iA, iB) {
  var memo = list[iA];
  list[iA] = list[iB];
  list[iB] = memo;
  return list; 
}

// memoSwap([1,2,5,4,3], 2, 4)
// -> [1,2,3,4,5]
{% endhighlight %}

### Bitwise swap - Integers only

Bitwise operators can be used to solve different problems (it's worth reading this [paper by Martin Richards](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.51.7113&rep=rep1&type=pdf), where among others things he suggests an interesting algorithm based on bitwise operators to solve the n-queens problem). A more common usage is to swap integers in place using the bitwise operator XOR (exclusive OR). 

{% highlight javascript %}
function bitwiseSwap(list, iA, iB) {
  list[iA] ^= list[iB];
  list[iB] ^= list[iA];
  list[iA] ^= list[iB];
  return list; 
}

// bitwiseSwap([1,2,5,4,3], 2, 4)
// -> [1,2,3,4,5]
{% endhighlight %}

Note: the bitwise swap method only works with integers

### Swap in-place without memo

While the memo technique may look simple and to the point, sometime we just want to avoid creating an extra variable just to hold one of the values that we are going to swap. To solve this we can rely on swapping helpers that don't need an extra variable to do their job.

{% highlight javascript %}
function swapInPlaceWithoutMemo(list, iA, iB){
  list[iB] = list[iA] + (list[iA] = list[iB]) - list[iB];
  return list;
}

// swapInPlaceWithoutMemo([1,2,5,4,3], 2, 4)
// -> [1,2,3,4,5]
{% endhighlight %}

If you wonder how the swapInPlaceWithoutMemo() function works: 
<br />the expression (list[iA] = list[iB]) does two things:

- it assigns to list[iA] the value stored at list[iB]
- returns the value stored at list[iB]<br />

<br />
### Swap in-place using Splice

We could also rely on the native splice method, to swap in place without need of a memo variable:

{% highlight javascript %}
function swapSpliceInPlaceWithotMemo(list, iA, iB){
  list[iA] = list.splice(iB, 1, list[iA])[0];
  return list;
}

// swapSpliceInPlaceWithotMemo([1,2,5,4,3], 2, 4)
// -> [1,2,3,4,5]
{% endhighlight %}


### Destructuring swap - ES6 only

ES6, come with an awesome feature called [destructuring assignment](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment), that we can take advantage of for putting together an awesome in-place swapping helper function:

{% highlight javascript %}
function destructuringSwap(list, iA, iB){
  [list[iA], list[iB]] = [list[iB], list[iA]];
  return list;
}

// destructuringSwap([1,2,5,4,3], 2, 4)
// -> [1,2,3,4,5]
{% endhighlight %}

Note: the destructuring swap method only work with ES6, [check compatibility](https://kangax.github.io/compat-table/es6/#test-destructuring) or use an ES6 transpiler like [babel](https://babeljs.io/) that will compile the deconstructuring swap to a memo swap.

* * *

####Other ES6 related posts:

- [ES6: JS's Prototype-based inheritance and classes demystified](http://nick.balestra.ch/2015/ES6-classes-and-prototype-based-inheritance/)
- [ES6: Arrows, array-like objects and defaults](http://nick.balestra.ch/2015/ES6-Arrows-Arrays-and-Defaults/)
- [Tail-recursive processes in JS](http://nick.balestra.ch/2015/recursion-workshop-part2/)

***

#### Other Sorting algorithms, related articles:

- [Simple O(n^2) sorting algorithms in JavaScript](http://nick.balestra.ch/2015/quadratic-time-sorting-algorithms)
- [O(nlogn) sorting algorithms in JavaScript](http://nick.balestra.ch/2015/logarithmic-time-sorting-algorithms)
