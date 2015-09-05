---
title: "Pick your parsing"
date: 2015-09-05
description:  "Why I went with acorn ever esprima"
---


##[Acorn](https://github.com/marijnh/acorn) is a tiny, fast JavaScript parser, written completely in JavaScript, it produces a well-documented, widely used [AST format](https://developer.mozilla.org/en-US/docs/SpiderMonkey/Parser_API) as esprima does. Acorn is really fast. Just like Esprima. Acorn is tiny. About half as big as Esprima, in lines of code.

Still, there's no good reason to pick Acorn over Esprima. So why did I pick it?

* * *

## Speed 
Speed sometime can be a relative thing, let say you are aiming to specifically parse comments: in that case being able to invoke functions on each comment as its found (both single line or block) may definitely increase the speed of the overal process, as you don't need to wait for the Abstract Syntax Treet to be genereated in orded to massage those comments data. And this is what acorn offer over esprima.

The acorn onComment option can be passed in the form of a function, so that whenever a comment is encountered the function will be called with the following parameters:

- block: true if the comment is a block comment, false if it is a line comment.
- text: The content of the comment.
- start: Character offset of the start of the comment.
- end: Character offset of the end of the comment.
- If the locations options is on, the {line, column} locations of the comment’s start and end are passed as two additional parameters.

Plus, if you want to fall back to how esprima works with comments, pass an array for this option instead, amd each found comment is pushed to it as object in Esprima format:

{% highlight javascript linenos %}
{
  "type": "Line" | "Block",
  "value": "comment text",
  "start": Number,
  "end": Number,
  // If `locations` option is on:
  "loc": {
    "start": {line: Number, column: Number}
    "end": {line: Number, column: Number}
  },
  // If `ranges` option is on:
  "range": [Number, Number]
}
{% endhighlight %}


***
## Final thoughts

Speed is important, but sometime the specific context you are working in can have a big impact on speed thanks to some little extra features, as it is in the case of acorn, if you need to parse comments. Secondary, documentation may play a big part of the decision, and in this direction both have a very similar API, and both documentations are very well written.

- [Acorn](http://marijnhaverbeke.nl/blog/acorn.html)
- [Esprima](http://esprima.org/)
