---
title: Intro.js onboarding API
date: 2015-01-25
description: My contribute for a better onboarding script
---


##Intro.js aim to deliver better introductions for websites and features with a step-by-step guide for your projects. I got the inspiration for a little pull request after having read a great post by Jan König titled “[Batman Onboarding](https://medium.com/@einkoenig/batman-onboarding-999d19f0cab9)”.

***
In that article, Jan writes about what’s wrong with onboarding wizards, outlining four main issues:

1. **Introducing all the features**
2. **Not based on user behavior**
3. **Too much friction**
4. **People want to be in control**
<br><br>
His third point, “Too much friction”, where he state how often onboarding wizards are not a part of the core user experience, but merely a layer on top of the product, made me think about those cases and how we could help them. It’s clear to me that top class teams and products, like the slack example he shows, know how to deal with those issues. But, what about all the smaller projects, apps and sites out there that don’t have all the resources or know-how needed to provide better onboarding experiences, and instead rely on off the shelf solutions like intro.js? Can something, that may not feel right, could partially address the other issues mentioned by Jan? Let see…

I’ll skip the “Introducing all the features” issue as with intro.js you have the power to decide what features to include in the intro from within your code. So, I’ll focus on the “Not based on user behavior” and “People want to be in control” issues, that are pretty tight together anyhow:

> The user has only one chance to be guided through the onboading flow: right after signup. So people who just want to play around and discover some features close the wizard immediately and don’t know how to get back when they really need to be guided through the product.

and

> Having no choice in the first steps they’re taking is obviously not a satisfying first-run experience.

**What if, through intro.js, an user could both retrieve the wizard and decide where to start from in that wizard? That’s exactly what my pull request for intro.js try to solve.**

***

###How it works
Batman will transform the intro.js steps defined in your markup into helpers (bringing them to the user in a “Pow / Ka-boom” style animation), that once clicked, will start the wizard from that specific point.
[Check out the demo to see it in action](https://medium.com/@einkoenig/batman-onboarding-999d19f0cab9).

####How to use it
We just need to call intro.js via its new batman API. This could be done via a click event: (Batman works as a toggle)

{% highlight javascript linenos %}
onclick="javascript:introJs().batman();"
{% endhighlight %}

####Under the hood
First an array containing the intro.js steps found on the page get created, together with an array of batman helpers (In case batman has been already fired).

{% highlight javascript linenos %}
var steps = document.querySelectorAll('*[data-intro]'),
    helpers = document.querySelectorAll('.batman-helper');
{% endhighlight %}

When invoked, the batman method will either add helpers on the page or remove them (in the case it has been already fired), following a toggle pattern.

Is important to know that for batman to work you must use the intro.js data-step attribute when defining the steps in the html.

By looping trough the *steps* array, batman will create for each item an html element (the helper) in the form of a *span*, appending it to the DOM. Every helper will contain an onclick attribute to fire intro.js. Similarly, if any helpers is found on the DOM, the method will remove it following the toggle pattern.

Is important to know that the batman method can accept any options that you will normally pass as argument to intro.js itself. It will use those arguments when firing intro.js later from within an helper, like:

{% highlight javascript linenos %}
onclick="javascript:introJs().batman({showBullets: false, ...});"
// check intro.js docs, for the full list of available options
{% endhighlight %}

Animations are handled purely via css.

{% highlight css linenos %}
.batman-helper {
  animation: POW .3s;
}
.ka-boom {
  animation: KA-BOOM .3s;
}
@-keyframes POW {
 0% {transform: scale(0.1, 0.1); opacity: 0.0;}
 50% {transform: scale(1.2, 1.2);opacity: .8;}
 100% {transform: scale(1, 1); opacity: 1.0;}
}
@-keyframes KA-BOOM {
  0% {transform: scale(1, 1); opacity: 1.0;}
  100% {transform: scale(1.3, 1.3); opacity: 0.0;}
}
{% endhighlight %}

***

###Final words and links
Take this as a proof of concept showing how some onboarding issues, as the ones outlined by Jan König in his post, could be addressed on a generic, off the shelf solution, like the popular intro.js.

Links:

- [intro.js batman API code](https://github.com/nickbalestra/intro.js/blob/feature/batman/intro.js#L271-L340)
- [intro.js batman CSS code](https://github.com/nickbalestra/intro.js/blob/feature/batman/introjs.css#L345-L380)
- [batman demo](http://nickbalestra.github.io/intro.js/example/batman/)


If you have any question, comment or feedback, please get in touch on [twitter](http://twitter.com/nickbalestra) or drop me a line at nick at balestra dot ch.