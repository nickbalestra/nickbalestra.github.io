---
title: "Simple O(n^2) sorting algorithms in JavaScript"
date: 2015-11-24
description: BubbleSort, SelectSort and InsertSort
---

No need to rely on logarithmic time sorting algorithms If you are sorting small datasets.
Instead, you are more then safe to fallback on a naif quadratic algorithm like insertSort or selectSort.
In this post, we'll walk through them.

***

### Bubble Sort

Is a simple sorting algorithm that repeatedly compares adjacents items and swap them if needed.
Although the logic is pretty simple, is way to slow to be used in a real scenario, even compared to insertSert.
It's ok to use it only in cases where the data that need to be sorted is already almost sorted and may only occasionally present few unsorted items.

Let's see it in action:

{% highlight javascript linenos %}
function bubbleSort(list) {
  function swap(list, iA, iB) {
    list[iA] = list[iB] + (list[iB] = list[iA]) - list[iA];
  }
  for (var i = 0, len = list.length; i < len; i++) {
    var swapped = false;
    for (var j = 1; j < len - i; j++) {
      if (list[j-1] > list[j]) {
        swap(list, j, j-1);
        swapped = true;
      }
    }
    if (!swapped) {
      return list;
    }
  }
  return list;
}

// bubbleSort([1, 3, 2, 4, 6, 5])
// -> [1, 2, 3, 4, 5, 6]

// time complexity: O(n^2)
{% endhighlight %}

If you wonder about swapping helpers, check-out the post on [JavaScript swap in-place array elements](http://nick.balestra.ch/2015/swapping-array-elements-in-place/)

***

### Select Sort

The idea behind select sort is to repeatedly select the minimum value from the array pushing it to a new array, this way the newly created array will result in being sorted. 

{% highlight javascript linenos %}
function selectSort(list) {
  var sorted = [];

  for (var i = 0, len = list.length; i < len; i++) {
    var minValue = Infinity;
    var minIndex;
    for (var j = i; j < len; j++) {
      if (list[j] < minValue) {
        minValue = list[j];
        minIndex = j;
      }
    }
    sorted[i] = minValue;
  }
  return sorted;
}

// selectSort([1, 3, 2, 4, 6, 5])
// -> [1, 2, 3, 4, 5, 6]

// time complexity: O(n^2)
{% endhighlight %}

Normally select sort is being used as an in-place comparison algorithm, allowing to save memory. For the sake of variation, we are going to select the Max value this time for the in-place example of selectSort:

{% highlight javascript linenos %}
function selectSortInPlace(list) {
  function swap(list, iA, iB) {
    list[iA] = list[iB] + (list[iB] = list[iA]) - list[iA];
  }

  for (var i = 0, len = list.length; i < len; i++) {
    var maxValue = -Infinity;
    var maxIndex;
    for (var j = 0; j < len - i; j++) {
      if (list[j] > maxValue) {
        maxValue = list[j];
        maxIndex = j;
      }
    }
    swap(list, maxIndex, len-1-i);
  }
  return list;
}

// selectSortInPlace([1, 3, 2, 4, 6, 5])
// -> [1, 2, 3, 4, 5, 6]

// time complexity: O(n^2)
{% endhighlight %}

***

### Insert Sort

Insert Sort take each element of the array and insert it into a new array in a sorted position:

{% highlight javascript linenos %}
function insertSort(list) {
  var sorted = [];

  for (var i = 0, len = list.length; i < len; i++) {
    var insert = false;
    var valueToInsert = list[i];
    for (var j = 0; j < sorted.length; j++ ) {
      if (valueToInsert < sorted[j]) {
        sorted.splice(j, 0, valueToInsert);
        insert = true;
        break;
      }
    } 
    if (!insert) {
      sorted.push(list[i]);
    }
  }
  return sorted;
}

// insertSort([1, 3, 2, 4, 6, 5])
// -> [1, 2, 3, 4, 5, 6]

// time complexity: O(n^2)
{% endhighlight %}

As we did for select sort we can have an insertion sorting in-place:

{% highlight javascript linenos %}
function insertSortInPlace(list) {
  function swap(list, iA, iB) {
    list[iA] = list[iB] + (list[iB] = list[iA]) - list[iA];
  }

  for (var i = 1, len = list.length; i < len; i++) {
    var insert = false;
    var valueToInsert = list[i];
    for (var j = 0; j < i; j++ ) {
      if (valueToInsert < list[j]) {
        swap(list, i, j)
        insert = true;
        break;
      }
    } 
    if (!insert) {
      swap(list, i, j);
    }
  }
  return list;
}

// insertSortInPlace([1, 3, 2, 4, 6, 5])
// -> [1, 2, 3, 4, 5, 6]

// time complexity: O(n^2)
{% endhighlight %}

***

####Sorting algorithms, related articles:

- [O(nlogn) sorting algorithms in JavaScript](http://nick.balestra.ch/2015/logarithmic-time-sorting-algorithms)
- [JavaScript swap in-place array elements](http://nick.balestra.ch/2015/swapping-array-elements-in-place/)
