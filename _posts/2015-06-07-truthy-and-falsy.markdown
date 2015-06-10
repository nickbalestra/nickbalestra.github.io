---
title: Truthy and falsy
date: 2015-06-07
description: When non-boolean values are coerced to booleans
---

##JavaScript defines a list of specific values that are considered "falsy" because when coerced to a boolean, they become false.

* * *

### Falsy

```javascript
Boolean("")  // Empty string
Boolean(0)
Boolean(-0)
Boolean(NaN)
Boolean(null)
Boolean(undefined)
Boolean(false)
```

Every other value will become true when coerced to a boolean, and therofore can be assumed to be truthy.

### Truthy

```javascript
Boolean("hello")
Boolean(true)
Boolean(17)
Boolean({ a: "hello" })  // Any object
Boolean({}) // Empty objects as well
Boolean([ 17, "hello", true ]) // Any array
Boolean([]) // Empty arrays as well
Boolean( function foo(){...} ) // Functions
```

***

It's important to remember that a non-boolean value only follows this "truthy"/"falsy" coercion if it's actually coerced to a boolean.