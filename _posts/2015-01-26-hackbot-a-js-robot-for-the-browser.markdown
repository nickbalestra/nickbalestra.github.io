---
title: Hackbot - A js robot for the browser
date: 2015-01-26
description: A fully programmable bot, modeled after Hubot.js
---

##During my deep dive journey into JavaScript I've encountered the Hack Reactor chat-builder challenge, where you have to implement a very basic chat client. [Hack Reactor](http://www.hackreactor.com/) is a world leading CS degree for the 21st century based in San Francisco. The chat-builder challenge is part of their admission program. That’s where I got the inspiration for the name — Hackbot.

***

I loved coding through that exercise but soon, I found myself on a dead-end: I finished and refactored my solution and I still wanted more. What could I do next? I set myself to find learning mates to do pair programming with, using chat-builder itself, transforming a dead-end into a side project, curious to see where that will take me.
The plan: build a js robot that will listen and reply in the chat, capturing peers attention as they test their own solutions, pushing them to contact me on twitter or via email.

### Evolution
The first version, as a proof of concept, was pretty basic. It was mainly a fork of my chat-builder solution with some DOM manipulation stripped off and renamed methods. Everything was hardcoded into the robot single js file program. It was definitely hard to extend. Some early concepts are still presents in the latest version, like *normalize* now found in the adapter, or the *print* method, a reminiscence of the chat-builder UI. That’s also why Hackbot live in the browser, while Hubot live on the server.

Initially the *Bot*(later renamed *Robot*) object had a *listen()* method used to fetch messages from the parse API. *listen* was invoked passing *brain* as callback.

As you can immagine, the main logic was located in there, and it didn’t take long for *brain()* to start growing. That’s when I started to look into the most famous and respected open source chat bot out there: [Hubot](https://hubot.github.com/).
***
### Architecture
Piece by piece, I started refactoring my robot, implementing the very same modular architecture of Hubot into Hackbot. This was the best way to learn how Hubot was working while, at the same time, learn javaScript. For this reason, even if Hackbot differ on many aspects from Hubot, it is similar enough to be used to learn about Hubot itself, how it works and how to program it. If this is what you are looking for, then you will enjoy the rest of the post

The main elements of the Hackbot (and Hubot) architecture are:
*Robot, Adapter, Listener, Message & Response* and *scripts*.
<br><br>
####Robot
#####What is it and why we need it
Robots receive messages from a chat source, and dispatch them to matching listeners. Robot is the central part of our application. Among other things it has a name (default to ‘Hackbot’), a brain as storage system, an adapter to interact with a specific chat source (in our case the chat-builder API), a collection of listeners to analyse incoming messages and few other utilities.

#####How to create a robot

{% highlight javascript linenos %}
robot = new Robot();  // Create a Robot object
robot.load();         // load scripts defined in script.js
robot.run();          // start the robot adapter
{% endhighlight %}

> Hackbot says: Bite my shiny metal ass!<br>
> Loading scripts at ../hackbot/scripts/<br>
> Starting adapter<br>
> running...


#####Under the hood
The created robot object will looks like:

{% highlight javascript linenos %}
{
    name: "Hackbot",
    listeners: Array,
    adapter: Adapter Object,
    brain: Brain Object
}
{% endhighlight %}

Through its prototype chain a robot has access to the following methods:


- [robot.hear()](http://nickbalestra.github.io/hackbot/robot.html#section-3)
- [robot.respond()](http://nickbalestra.github.io/hackbot/robot.html#section-4)
- [robot.receive()](http://nickbalestra.github.io/hackbot/robot.html#section-5)
- [robot.load()](http://nickbalestra.github.io/hackbot/robot.html#section-6)
- [robot.help()](http://nickbalestra.github.io/hackbot/robot.html#section-7)
- [robot.send()](http://nickbalestra.github.io/hackbot/robot.html#section-8)
- [robot.print()](http://nickbalestra.github.io/hackbot/robot.html#section-9)
- [robot.reply()](http://nickbalestra.github.io/hackbot/robot.html#section-10)
- [robot.run()](http://nickbalestra.github.io/hackbot/robot.html#section-11)
- [robot.shutdown()](http://nickbalestra.github.io/hackbot/robot.html#section-12)

<br>
####Adapter
#####What is it and why we need it
An adapter is a specific interface to a chat source for robots. In the case of Hackbot its adapter is made to work with chat-builder as chat source. Shortly, adapters can be seen as some kind of middleware API, sitting between the robot and the chat source.

#####Under the hood
The adapter through its prototype chain has access to the following methods (you will notice that few of them have similar names to the methods we listed above for the robot itself. This is because in some cases the robot will just delegate the action to its adapter):

- [adapter.fetch()](http://nickbalestra.github.io/hackbot/adapter.html#section-3)
- [adapter.run()](http://nickbalestra.github.io/hackbot/adapter.html#section-4)
- [adapter.close()](http://nickbalestra.github.io/hackbot/adapter.html#section-5)
- [adapter.receive()](http://nickbalestra.github.io/hackbot/adapter.html#section-6)
- [adapter.send()](http://nickbalestra.github.io/hackbot/adapter.html#section-7)
- [adapter.reply()](http://nickbalestra.github.io/hackbot/adapter.html#section-8)
- [adapter.emote()](http://nickbalestra.github.io/hackbot/adapter.html#section-9)
- [adapter.print()](http://nickbalestra.github.io/hackbot/adapter.html#section-10)
- [adapter.clean()](http://nickbalestra.github.io/hackbot/adapter.html#section-11)
- [adapter.format()](http://nickbalestra.github.io/hackbot/adapter.html#section-12)
- [adapter.normalize()](http://nickbalestra.github.io/hackbot/adapter.html#section-13)
<br><br><br>


####Listener

#####What is it and why we need it
Listeners receive messages from the chat source and decide if they want to act on them. Among other things, they have the following properties: a regular expression that determines if the listener should trigger the callback, the callback itself (a function to be triggered if the incoming message matches) and through its prototype chain it has access to a call method that determines if the listener likes the content of the message. If so, a Response built from the given Message is passed to the Listener’s callback.
<br><br>

#####Under the hood
Shortly, when robots receive a message (via an adapter), will invoke the listener via its call method passing the message as argument:

{% highlight javascript linenos %}
listener.call(message)
{% endhighlight %}


Inside that call function, if the listener’s regex match the message text, the listener’s callback function will be invoked passing a new response as argument:

{% highlight javascript linenos %}
if (match) {
    callback(new Response(robot, message, match));
... }
{% endhighlight %}
See [annotated documentation of the listener](http://nickbalestra.github.io/hackbot/listener.html) for more details

<br>
####Message & Response
#####What are them and why we need them
Messages and responses are data structures used to carry data around the application. Messages get created inside the adapter with data receive from the chat source. Responses get created inside listeners when Messages trigger a listener’s callback, as we just saw earlier.

#####Under the hood
A message object, has the following properties:

{% highlight javascript linenos %}
{
    user: User Object,
    text: 'Hello world',
    id: '0000000000001',
    createdAt: date,
    done: false || true
}
{% endhighlight %}

Through its prototype chain a message has access to a match method to determines if the message matches a given regex, and is used by the listener’s matcher method when invoked trough its call method as we saw above. The Response object wrap the message and contain some useful properties, it look something like:

{% highlight javascript linenos %}
{
    message: Message Object,
    match: Array (containing the regex matchings if any),
    envelope: Object (containing user and message)
}
{% endhighlight %}

Through the prototype chain, the response object has access to the following methods: send, emote, reply, random and finish.
Check the full annotated documentation of Message and Response for further details.
<br><br>

####Scripts
#####What are them and why we need them
Scripts refer to the system that allow to extend Hackbot. In other words they are little plugins that extend the functionality of the bot. Scripts are made with the goal of adding one or more listeners to our robot. For that reason they will need to define what the robot will listen for (using a regular expression), and what will the robot do if it hears the right words (using a callback function).

#####How to create listeners within scripts
There are two methods available within a robot for that: hear() and respond(). Both accept a regex and a callback function as parameters. They differ in how the robot will listen to messages. With hear(), robot listen to the whole conversation, checking all words, while with respond(), robots will listen to words only when somebody calls it. For example:

> **User**: ‘Oh, nice to be in here, hello world’<br>
> **User**: ‘@Hackbot: hey bot, hello ’<br>
> **User**: ‘Hackbot please say hello to me’

robot.hear(/hello/, callback) will trigger its callback in all the three examples above, while robot.respond(/hello/, callback) only in the latest two.

Similarly, if in the callback we want our Robot to say something in the chat, we have three options: using response.send(), response.reply() and response.emote(). They will generate the following type of messages in the chat:

> **Hackbot**: ‘Thank you all ’ // send()<br>
> **Hackbot**: ‘@User: Thank you Sir’ // reply()<br>
> **Hackbot**: ‘* is Happy *’ // emote()

The latest useful methods to be aware when scripting Hackbot are response.finish() and response.random().

Response.finish() will mark the message as done (by setting the value of his property done to true). The result of this is that the message won’t be passed to any other listener for checking after that moment.

Response.random() is a simple method that return a random value out of an array. Can be useful to create random replies or sentences. For an example, see how its being used in the [thank you script](http://nickbalestra.github.io/hackbot/hackbot-thankyou.html).

#####How to add help commands to a script
To add some help to a script that can be called by typing the help command in the chat, use the help method:

{% highlight javascript linenos %}
robot.help("Hello", "Make Hackbot saying Hello");
{% endhighlight %}
If in the chat somebody type help, robot will respond as follow:

> **Hackbot**: < Hello > — Make Hackbot saying Hello

See the [annotated documentation of the help script](http://nickbalestra.github.io/hackbot/hackbot-help.html) for more details. In Hackbot the help system is implemented as a script, so you can disable help by not loading that script.
***
###Hackbot vs Hubot
Although Hackbot mimic very closely Hubot’s architecture, the two are different in some aspects.

####Vanilla javascript vs Coffeescript
Hubot is entirely written in [coffeescript](http://coffeescript.org/), a little language that compiles into JavaScript, while Hackbot is written in plain javaScript. To help navigate from one to the other, use [js2coffee](http://js2coffee.org/).

####Browser vs Server
Hubot run on the server thanks to node.js while Hackbot run on the browser. It may be argued that having a chat bot running on the browser ain’t that useful, and I could generally agree with that, although there can be cases where having a robot born and die within your browser could be a good thing. For sure, in a learning environment depending solely on the browser reduces the elements needed to be understood in order to have everything up and running, making our life a little bit easier.
***
###Dependencies
####Require.js
Running in the browser is perhaps the single biggest cause of changes inside Hackbot compared to Hubot. First, to be able to implement a similar modular architecture, based around the module pattern, I had to use a library for that. Hackbot is therefore built using require.js, and therefore depend on it.(Require.js is a javaScript file and module loader. It is optimised for in-browser use).

####Chatbuilder.js
Of course, being built around chatbuilder, Hacknot require chatbuilder.js. Although this could be relegated to a dependency for the adapter only, the chatbuilder.js has been added as a global dependency to be inline with the scope of the exercise. As a nice side effect, there is no need to add jQuery as it comes packed within chatbuilde.js itself.

####Underscorish.js
In order to take advantage of some functional utilities similar to what the underscore.js library offer, I’ve created a very little library called underscorish. At the moment underscorish come packed with very limited methods:

- [_.each()](http://nickbalestra.github.io/hackbot/underscorish.html#section-2)
- [_.map()](http://nickbalestra.github.io/hackbot/underscorish.html#section-3)
- [_.filter()](http://nickbalestra.github.io/hackbot/underscorish.html#section-4)
- [_.reduce()](http://nickbalestra.github.io/hackbot/underscorish.html#section-5)
- [_.every()](http://nickbalestra.github.io/hackbot/underscorish.html#section-6)

If you want to take advantage of the underscorish.js library when creating your own scripts, just define the relative dependency following the require.js syntax as follow:

{% highlight javascript linenos %}
define([‘underscorish’], function (_) {
    ...your script code here...
}
{% endhighlight %}

For more details, please check the [underscorish annotated documentation](http://nickbalestra.github.io/hackbot/underscorish.html).

***

###Challenges
####Brain
Hackbot runs in the browser, so gaining writing access to local file system in order to have a persistent storage was anything but trivial. After digging possible solutions it seemed that using the html5 localStorage API was the best things to do. Hackbot’s brain therefore differ from the Hubot brain implementation as it use the localStorage API. Few extra methods are then available to deal with it:

- [.backup()](http://nickbalestra.github.io/hackbot/brain.html#section-9)
- [.hasBackup()](http://nickbalestra.github.io/hackbot/brain.html#section-10)
- [.clearBackup()](http://nickbalestra.github.io/hackbot/brain.html#section-11)

See [annotated documentation of the brain](http://nickbalestra.github.io/hackbot/brain.html) for more details

####Scripts
Similar to brain, I had to find a way for Hackbot to know what scripts to load. In order to achieve this, two things has been slightly changed. First the load() method for the robot:

{% highlight javascript linenos %}
robot.load(path)
{% endhighlight %}

We saw this earlier when I showed how to create a robot. Path is a string (i.e: ‘/newScriptFolder’). If nothing is passed to the method, the default will be ‘../hackbot/scripts/’ as found in the github repository structure. It’s important to know that in the defined directory a file named scripts.js must be present. In that file, all scripts that Hackbot should load must be listed and the corresponding files must exist in the same directory. Then it’s just a matter of calling the load() method:

{% highlight javascript linenos %}
define({
    'aName' : 'scriptFilename'
});
// This will load scriptFilename.js
{% endhighlight %}

See [annotated documentation of the scripts](http://nickbalestra.github.io/hackbot/scripts.html) for more details

####Adapter
For the adapter few things were different compared to other Hubot adapters. First of all the chat-builder API automatically generate dummy messages for the sake of the challenge. That’s why the Hackbot adapter come with a private helper [clean()](http://nickbalestra.github.io/hackbot/adapter.html#section-11) in order to remove dummy content from the fetched data. Another important private helper added to the Hackbot adapter is [normalize()](http://nickbalestra.github.io/hackbot/adapter.html#section-13). As the adapter pull data from the chat-builder API at regular intervals, the normalize() helper implements some kind of temporary cache to be sure that the same message is not sent twice to the robot and its listeners.

See [annotated documentation of the adapter](http://nickbalestra.github.io/hackbot/adapter.html) for more details.

***

###Scripts
To test and use Hackbot I’ve developed few scripts, some have been ported from Hubot while others have been developed from scratch. Treat them as examples to learn and understand how to program Hackbot. Below is an high level overview of some of them.

####Hackbot-log.js
This little script log every user message on screen, using the print method.

####Hackbot-users.js
This script take advantage of the robot brain storing all the users messages. This allow for retrieveing both the user list and the message history for a single user. To use it type:

> **< Hackbot chat users >** — list all users who chatted in here<br>
> **< Hackbot hackbot chat history [name] >** — show all messages from [name] — case sensitive<br>
> **< Hackbot chat history me >** — show all my messages

####Hackbot-ambush.js
This script can be used to leave a message to somebody that is not online now and deliver it once the received is back (when he/she writes something in the chat). This script have been ported from a Hubot script. To use it type:

> **< Hackbot ambush [name] : message >** — leave a message for [name]

####Hackbot-hackreactor.js
This script uses 3rd part APIs. Those APIs have been created using [kimonolabs](https://www.kimonolabs.com/) and they fetch the following data from hackreactor.com:

- Next program dates.
- Names, github, linkedin, personal sites, and twitter handles of the previous alumni.

To use it type:

> **< Hackbot hr program >** — show next courses as published on hackreactor.com/program<br>
> **< Hackbot hr students (name | twitter | github | linkedin | site) >** — take and list relative students data from hackreactor.com/students

For more details check the full documentation:

- [hackbot-help.js](http://nickbalestra.github.io/hackbot/hackbot-help.html)
- [hackbot-about.js](http://nickbalestra.github.io/hackbot/hackbot-about.html)
- [hackbot-ping.js](http://nickbalestra.github.io/hackbot/hackbot-ping.html)
- [hackbot-hello.js](http://nickbalestra.github.io/hackbot/hackbot-hello.html)
- [hackbot-thankyou.js](http://nickbalestra.github.io/hackbot/hackbot-thankyou.html)
- [hackbot-log.js](http://nickbalestra.github.io/hackbot/hackbot-log.html)
- [hackbot-users.js](http://nickbalestra.github.io/hackbot/hackbot-users.html)
- [hackbot-ambush.js](http://nickbalestra.github.io/hackbot/hackbot-ambush.html)
- [hackbot-hackreactor.js](http://nickbalestra.github.io/hackbot/hackbot-hackreactor.html)
- [hackbot-hackreactorrocks.js](http://nickbalestra.github.io/hackbot/hackbot-hackreactorrocks.html)

***

###Conclusion and Future
Building Hackbot is been an extremely funny, challenging and very rewarding exercise for me. Although I didn’t keep it alive enough in order to find pair programming mates (I had two responses, one via twitter and one via email), I believe that hackbot could be used and be beneficials to others as it end up being a nice open ended exercise that you can always grow with: scripts can do anything you can immagine so sky is the limit. I therefore see it as a perfect companion to the chat-builder challange for anyone learning javaScript or preparing for the HR admission.

###Documentation and code
Hackbot uses inline comments formatted accordingly to [docco.js](http://jashkenas.github.io/docco/).
Please check the whole [Hackbot documentation](http://nickbalestra.github.io/hackbot/) to learn more.
Feel free to [fork Hackbot and send a pull-request on Github](https://github.com/nickbalestra/hackbot).


Thanks for reading, if you have any questions, please get in touch on [twitter](http://twitter.com/nickbalestra) or drop me a line at nick at balestra dot ch.