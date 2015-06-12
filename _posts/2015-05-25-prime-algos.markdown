---
title: Trial division, Sieve of Eratosthenes and Factorization Wheel
date: 2015-05-25
description: A short post on prime algorithms
---

##In number theory, **integer factorization** is the decomposition of a composite number into a product of smaller integers. If these integers are further restricted to prime numbers, the process is called **prime factorization**.

* * *
Lets quickly walk through some basic integer factorization algorithms.

### Trial division
The first among those algorithms, most laborious, but easiest to understand is the trial divison.

> Given an integer n (where n refers to "the integer to be factored"), trial division consists of systematically testing whether n is divisible by any smaller number


If n is not divisible by any smaller number, then n is a prime. Lets put this algorithm into code:

{% highlight javascript linenos %}
var isPrime = function(n){
  var divisor;
  for(divisor = 2; divisor < n; divisor++) {
    if (n % divisor === 0) {
      return false;
    }
  }
  return true;
};
{% endhighlight %}

Furthermore, the trial factors need go no further than √n because, if n is divisible by some number p, then n = p × q and if q were smaller than p, n would have earlier been detected as being divisible by q or a prime factor of q.
We could then rewrite an optimised version of the isPrime function based on the trial division in the following way:

{% highlight javascript linenos %}
var isPrime = function(n){
  var divisor;
  for(divisor = 2; divisor <= Math.sqrt(n); divisor++) {
    if (n % divisor === 0) {
      return false;
    }
  }
  return true;
};
{% endhighlight %}

Understandably this algorythm can be pretty resource intensive. A better approach could be to sieve a set of numbers using one of the many sieve algorithms available, like for example the:

* * *

### Sieve of Eratosthenes

In mathematics, the sieve of Eratosthenes, one of a number of prime number sieves, is a simple, ancient algorithm for finding all prime numbers up to any given limit. It does so by iteratively marking as composite (i.e., not prime) the multiples of each prime, starting with the multiples of 2.

The algorythm can be braken down into the following steps:

1. Create a list of consecutive integers from 2 through n: (2, 3, 4, ..., n).
2. Initially, let p equal 2, the first prime number.
3. Starting from p, enumerate its multiples by counting to n in increments of p, and mark them in the list (these will be 2p, 3p, 4p, ... ; the p itself should not be marked).
4. Find the first number greater than p in the list that is not marked. If there was no such number, stop. Otherwise, let p now equal this new number (which is the next prime), and repeat from step 3.

Assuming we already have a list of consecutive integers from 2 through n, we could put the sieving into the following code:

{% highlight javascript linenos %}
var sieve = function(list, p) {

  var sieved = list;

  // STEP 2 - Initially, let p equal 2, the first prime no.
  p = p || 2;

  // STEP 3 - Starting from p, enumerate its multiples to n
  // in increments of p and remove them from the list.
  // p itself should not be removed.
  var init = 2*p;
  var n = sieved[sieved.length - 1];
  for (var i = init; i <= n; i += p ) {
    if (sieved.indexOf(i) !== -1 ) {
      sieved.splice(sieved.indexOf(i), 1);
    }
  }

  // STEP 4 - Find the first unmarked number greater than p.
  var nextPrime = sieved[sieved.indexOf(p) + 1]

  // If there was no such number, stop.
  // Otherwise, let p now equal this new number,
  // and epeat from step 3.
  if (nextPrime) {
    return sieve(sieved, nextPrime);
  } else {
    return sieved;
  }

};
{% endhighlight %}
Again, as the original algorithm suggest there are few optimization that could be done:

- As a refinement, it is sufficient to mark the numbers in step 3 starting from p^2, as all the smaller multiples of p will have already been marked at that point. This means that the algorithm is allowed to terminate in step 4 when p^2 is greater than n.
- Another refinement is to initially list odd numbers only, (3, 5, ..., n), and count in increments of 2p in step 3, thus marking only odd multiples of p.

and here the refined code:

{% highlight javascript linenos %}
var sieve = function(oddList, p) {
  var sieved = oddList;
  p = p || 3;

  var init = p*p;
  var n = sieved[sieved.length - 1];

  for (var i = init; i <= n; i += (p*2) ) {
    if (sieved.indexOf(i) !== -1 ) {
      sieved.splice(sieved.indexOf(i), 1);
    }
  }

  var nextPrime = sieved[sieved.indexOf(p) + 1]

  if (p*p > n) {
    return sieved;
  } else if ( nextPrime ) {
    return sieve(sieved, nextPrime);
  } else {
    return sieved;
  }

};
{% endhighlight %}



As the sieving range gets larger, it is necessary to change the implementation to only sieve primes per page segment. In other words we should paginates the list to optimize for larger cases, and this is excactly what a **wheel factorization** algorithm does.

* * *

Learn more about the [wheel factorization](http://en.wikipedia.org/wiki/Wheel_factorization) and discover [other interesting Integer factorization algos](http://en.wikipedia.org/wiki/Integer_factorization) like [Fermat's](http://en.wikipedia.org/wiki/Fermat%27s_factorization_method), [Euler's](http://en.wikipedia.org/wiki/Euler%27s_factorization_method) and [Dixon's](http://en.wikipedia.org/wiki/Dixon%27s_factorization_method)

