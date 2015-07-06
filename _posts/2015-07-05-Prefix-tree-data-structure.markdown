---
title: Prefix Tree Data Structure
date: 2015-07-05
description: Storing sequences with a fast look-up
---

##A prefix tree, or [trie](https://en.wikipedia.org/wiki/Trie) is a data structure for storing strings or sequences in a way that allows for a fast look-up.

The idea is that in a prefix tree each successive letter of a string is stored as a separate node.

***

### traverse
To find out if the string *'hello'* is stored in our prefixTree we start at the root and look up the *'h'* node. Once we found the *'h'* node we iterate trough the list of its children in search for an *'e'* node, and so on.

### getSuggestions
Prefix trees are often used for suggestions as we can return the strings stored in the nodes ahead of us in the process of traversing the tree. For that we will need to flatten the results stored ahead the node on which we are currently located(while traversing the tree following the sequences of the letters in the string). How many suggestions we will find depend on many factors, but mainly is due to how many level deep we want to lookahead.

### insert
In a similar way as we do for traversing it, to add our *'hello'* string into our trie, we start by looking up the *'h'* node. In case there is no *'h'* we create it and we keep doing so for each letter of the string until we reach the last element of the sequence (*'o'* in our case) and its relative node, where we will finally store our string.

A prefix tree can be used for word auto-completetion, as a dictionary, but also as a storage alternative to hashmaps.

If you are curious and wanna give it a try, please check out my [prefix tree testing suite](https://github.com/nickbalestra/LearnPrefixTrees) on github .

The testing suite, powered by mocha, is available by simply running `SpecRunner.html`. This will check up on your PrefixTree implementation and run a series of tests on it to see if it is satisfactory. If you can make your prefix tree pass all of the tests than you should have a working prefix tree that can handle word-autocompletion for T9-style texting in javaScript!
Have fun getting that testing page full of green.
