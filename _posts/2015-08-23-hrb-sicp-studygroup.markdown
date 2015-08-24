---
title: "HRB SICP Study group"
date: 2015-08-23
description:  "A study group on 'Structure and Interpretation of Computer Programs'"
---

##The source of the exhilaration associated with computer programming is the continual unfolding within the mind and on the computer of mechanisms expressed as programs and the explosion of perception they generate. If art interprets our dreams, the computer executes them in the guise of programs! ~ *Alan J. Perlis, MIT/SICP Forword*.

The HRB SICP Study group stand for Hackreactor Remote Beta Structure and Interpretation of Computer Programs - Study group.

The group is intended to be a very long term study group. Together with other peers at [HackReactor](http://www.hackreactor.com/remote-beta) we plan on continuing to finish [SICP](http://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/) and push forward behind it after finishing HR's advanced software engineering immersive program.

![alt MIT/GNU Scheme Logo](/assets/images/hrb-sicp-logo.png)

To set up the MIT/GNU Scheme environment just follow the next steps.
You'll be up and running in seconds.

* * *

### Step 1: Download and install the Scheme binary
[Download](http://www.gnu.org/software/mit-scheme/) the best binary for your OS (for macOS X -> x86-64)

### Step 2: Set up Scheme to be used via cli

Similarly to the node REPL, you can also run the the MIT/GNU Scheme directly from the command line.

#### Symlink the binary in your /usr/bin:

{% highlight javascript linenos %}
sudo ln -s /Applications/MIT\:GNU\ Scheme.app/Contents/Resources/mit-scheme /usr/bin/scheme
{% endhighlight %}

#### Add the library path to your bash profile:

{% highlight javascript linenos %}
export MITSCHEME_LIBRARY_PATH="/Applications/MIT\:GNU\ Scheme.app/Contents/Resources"

// Optionally you can also add the
// following environmental variable:
export MIT_SCHEME_EXE="/usr/local/scheme"
{% endhighlight %}

Done! Just reload your terminal, and run

{% highlight javascript linenos %}
scheme
{% endhighlight %}

Now you should be ready start writing
your first awesome Scheme Scripts.

{% highlight javascript linenos %}
(display "Hello World")
{% endhighlight %}

***
##Other Resources

- [SICP Video lectures](http://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/video-lectures/)
- [SICP Material](https://mitpress.mit.edu/sicp/)
