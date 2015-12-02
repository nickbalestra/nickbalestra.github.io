---
title: "O(nlogn) sorting algorithms in JavaScript"
date: 2015-11-26
description: mergeSort and quickSort
---

If time complexity is not such a big deal when dealing with small datasets you definitely want to rely on sorting algorithms that performs much better than quadratic time as the input scale. The two most common and performant algorithms in this regards, which utilize a divide and conquer approach to achieve logarithmic time O(nlogn), are mergesort and quicksort. One difference worth noticing is stability: while mergesort is a stable sorting algorithm, quick sort isn't.

***

### Merge sort

The idea behind merge sort, is quite simple: recursively divide the unsorted list into two halves until you have n lists, each of length one. Then, walking the call stack binary tree backward, merge each pair of lists into a single sorted list. The whole logic of merge sort, as the name suggest, rely therefore upon the merging and sorting process. Because we start the process from single element lists, on each step the lists that we are going to merge can be safely assumed to be already sorted, making the process of merging and sorting  straightforward, as it will be enough to compare the firsts elements of each list popping out the smaller one.

#### Visualizing the process

{% highlight javascript linenos %}
// Merge sort process

              [1, 3, 2, 4, 6, 5]
             /                  \
    [1, 3, 2]                    [4, 6, 5]
   /         \                  /         \
 [1]          [3, 2]         [4]           [6, 5]
   |          /     \         |           /     \
   |      [3]       [2]       |         [6]      [5]
   |         \     /          |           \     /
   |          [2, 3]          |            [5, 6]
    \         /                \          /
     [1, 2, 3]                   [4, 5, 6]
              \                 / 
               [1, 2, 3, 4, 5, 6] 

{% endhighlight %}

The code could look something along the following lines:

{% highlight javascript linenos %}
function mergeSort(list) {
  
  function merge(listR, listL) {
    var output = [];
    while (listL.length && listR.length) {
      listL[0] < listR[0] ? output.push(listL.shift()) : output.push(listR.shift())
    }
    return output.concat(listL).concat(listR);
  }

  if (list.length < 2) {
    return list;
  };

  var pivot = Math.floor(list.length / 2);
  var listL = list.slice(0, pivot);
  var listR = list.slice(pivot);

  return merge(mergeSort(listL), mergeSort(listR));
}

// mergeSort([1, 3, 2, 4, 6, 5])
// -> [1, 2, 3, 4, 5, 6]

// time complexity: O(nlogn)
{% endhighlight %}

***

### Quick Sort

Quicksort is the sorting algorithm behind the native array sort method in javascript, although different browsers may rely on mergesort instead. The idea is to pick recursively the last element and set it as the pivot. Then sort the list by positioning all the elements with a value less then the pivot's value on the left and the rest on the right of the pivot. At this point, the pivot can be safely assumed to be sorted with its proper final index assigned. By recursively applying the process to each side we'll soon find ourself with n lists, each of length 1, that only need to be concatenated back together while walking backward the call stack.

#### Visualizing the process
{% highlight javascript linenos %}
// Merge sort process

                        [1, 3, 2, 4, 6, 5]
                                |
                        [1, 3, 2, 4, 5, 6]
                       /        |         \
              [1, 3, 2, 4]     [5]         [6]
             /      |           |           |
    [1, 3, 2]      [4]          |           | 
        |           |           |           |
    [1, 2, 3]       |           |           |
   /    |    \      |           |           |
[1]    [2]    [3]   |           |           |
   \    |    /      |           |           |
    [1, 2, 3]       |           |           |
             \      |           |           | 
             [1, 2, 3, 4]       |           |
                         \      |          /
                         [1, 2, 3, 4, 5, 6]
              
{% endhighlight %}

Again, the code could look something along the following lines:

{% highlight javascript linenos %}
function quickSort(list) {
  if (list.length < 2) {
    return list;
  }
  var pivot = list[list.length - 1];
  var listL = [];
  var listR = [];
  for (var i = 0; i < list.length - 1; i++ ) {
    if (list[i] < pivot) {
      listL.push(list[i]);
    } else {
      listR.push(list[i]);
    }
  }
  return quickSort(listL).concat(pivot).concat(quickSort(listR));
}

// quickSort([1, 3, 2, 4, 6, 5])
// -> [1, 2, 3, 4, 5, 6]

// time complexity: O(nlogn)
{% endhighlight %}

***

####Sorting algorithms, related articles:

- [Simple O(n^2) sorting algorithms in JavaScript](http://nick.balestra.ch/2015/quadratic-time-sorting-algorithms)
- [JavaScript swap in-place array elements](http://nick.balestra.ch/2015/swapping-array-elements-in-place/)
