---
title: "Deep copying Objects and Arrays"
date: 2015-12-1
description: Dealing with infinite nested structures
---

Oftentimes, especially when dealing with pure functions, we might to deep copy objects and array. Although we may rely on JSON.stringify and JSON.parse methods for that, implementing a recursive routine that works for both objects and arrays, isn't something that hard, let see.

***

### deepCopy

The idea is simple: we iterate through each key/value pair of an object (in the case of arrays, keys will be represented by numeric strings), and we copy each pair over a new object and return it.

We have watch out for two things to :

#### 1) Nested objects / arrays

Here is where the recursive part of our function kicks in: in the case of a nested object we simply recursively invoke deepCopy on that value.

#### 2) Arrays with holes

Arrays could contain holes, meaning their length won't be affected, but an index/value could be missing, i.e., var array = ['a', ,'m']. In this case the we don't have any array[1], but the length of such array will still return 3. For that reason, we want to be sure that we iterate only through the referenced keys. So how could we see if array[1] === undefined because at the index 1 the 'undefined' value is stored vs the array missing that specific index key?

Object.keys to the rescue! As we can treat Arrays as Objects when accessing their key/values we could rely for both data structures to  Object.keys(array) to retrieve an array containing all its keys. Later we can simply use those keys to iterate through arrays and objects. Doing so will also avoid us relying on obj.hasOwnProperty(key) as we would have done if we used a for in loop construct instead).


Let's see some code:

{% highlight javascript linenos %}
function deepCopy(obj) {
  var copy = Array.isArray(obj) ? [] : {};

  var keys = Object.keys(obj);
  for (var i = 0; i < keys.length; i++) {
    var key = keys[i];
    if (obj[key] === undefined || obj[key] === null || typeof obj[key] !== 'object') {
      copy[key] = obj[key]; 
    } else {
      copy[key] = deepCopy(obj[key]);
    }
  }
  return copy;
}

// var obj = [{a:1, b: {b:2, c:3}, d: null, e: [1,2,3, [3, ,5]]}];
// JSON.stringify(deepCopy(obj1)) === JSON.stringify(obj1);
// -> true

{% endhighlight %}

