---
title: Building a simple server with node
date: 2015-07-26
description: Vanilla node, router and utils.
---

##[Node.js](http://nodejs.org/) is a platform built on Chrome's JavaScript runtime for easily building fast, scalable network applications. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient, perfect for data-intensive real-time applications that run across distributed devices.

In this post we'll go through creating and architecting a simple web server, its router and some nifty utils using node.

*Disclaimer: Haters gonna hate, lovers gonna love, but no express.js files were harmed in the making, so be warned, pure vanilla node code ahead.*

***

## About node

At it's core node is nothing more then a Javascript Interpreter, a cli (the Node REPL) and some helpers.
The Interpreter is built on the V8 engine, an open source JavaScript engine developed by Google for the Google Chrome web browser. V8 compiles JavaScript to native machine code. The node REPL environment (Read, Eval, Print, Loop) allows you to run Javascript inside the terminal. Node also come with a set of utility libraries for performing common development tasks, like [fs](https://nodejs.org/api/fs.html) to interact with the file system or [http](https://nodejs.org/api/http.html) to handle HTTP requests.
That's mostly about it as Node isn’t much more than that.

* * *

## The Architecture

### A basic server
The first thing that we need is to have a live server up and running listening for requests on a specific address:port. In other words here is where we'll expose our public API for the external world to communicate and interact with. For this we need to rely on the node http module, so we'll first require it and then create a server via its createServer() method. createServer accept a function as parameter and automatically add it to the 'request' event.

{% highlight javascript linenos %}
var http = require("http");
var router = require('./router');

var port = 3000;
var ip = '127.0.0.1';

var server = http.createServer(router);
server.listen(port, ip);
{% endhighlight %}

### A router module

We now have a new instance of http.Server listening for requests on 127.0.0.1:3000. Every time a 'request' event triggers, the router will be called and the request object passed to it. The router checks the url of the request against a list of registered routes (a.k.a our public API endpoints) and route the request to the relative endpoint handler, or, in case such a route weren't registered, send back a 404 response.

{% highlight javascript linenos %}
var url = require('url');

// The routes hash maps registered endpoint URLs
// with their relative handler modules.
var routes = {
  '/endpoint' : require('./endpointHandler')
};

// The router module check the url of the request
// against the routes hash and route the request to its
// relative handler. If the requested endpoint
// isn't a valid registered route a 404
// reponse will be sent back instead.
module.exports = function(request, response) {

  var routeHandler = routes[url.parse(request.url).pathname];

  if (routeHandler){
    routeHandler.requestHandler(request, response);
  } else {
    // TODO: respond with a 404
  }
};
{% endhighlight %}

For the sake of the post our server will just have a single valid endpoint '/ping' inside the routes hash. The handler for the endpoint will support GET requests only, responding back with a neat 'pong'.

Before being able to work on the ping endpoint handler we need to put together some helpers to make our life easier and dryer.

### Utils

At a very besic level we won't need many helpers, is enough to have:

**respond()** to build and send the http response.<br>
**fetchData()** to grab the data passed within each request (we won't take advantage of it throughout this post, but is definitely a must have).<br>
**actionDispatcher()** to check for requests methods supported inside each endpointHandler and dispatch the request accordingly.

Inside the utils file we can also define some headers to be used within our responses, allowing us to define CORS headers and any other header property we may need.

{% highlight javascript linenos %}
var headers = {
  "access-control-allow-origin": "*",
  "access-control-allow-methods": "GET, POST, PUT, DELETE, OPTIONS",
  "access-control-allow-headers": "content-type, accept",
  "access-control-max-age": 10, // Seconds.
  'Content-Type': "application/json"
};

exports.respond = function(response, data, statusCode){
  statusCode = statusCode || 200;
  response.writeHead(statusCode, headers);
  response.end(JSON.stringify(data));
};

exports.fetchData = function(request, callback){
  var data = "";
  request.on('data', function(chunk){
    data += chunk;
  });
  request.on('end', function(){
    callback(JSON.parse(data));
  });
};

exports.actionDispatcher = function(actionsHash){
  return function(request, response) {
    var action = actionsHash[request.method];
    if (action) {
      action(request, response);
    } else {
      exports.respond(response, '', 404);
    }
  }
};
{% endhighlight %}

### The endPoint handlers

We now have everything we need to easily create the endPointHandlers containing the logic and definition for each endpoint of our API. In our example we said to want to have a '/ping' end point that will respond only to GET requests with a simple pong response.

What we need is to require our utils, that will help us handle both the requests and responses as well as the actions hash. The actions Hash is where we will define the logic for each http method we may want to support for the given endpoint. Here the ping endpoint handler definition:

{% highlight javascript linenos %}
var utils = require('../utils');

var actions = {
   'GET': function(request, response){
     utils.respond(response, 'Pong');
   }
   // 'POST': function(request, response){},
   // 'PUT': function(request, response){},
   // 'DELETE': function(request, response){},
  // 'OPTIONS': function(request, response){}
};

exports.requestHandler = utils.actionDispatcher(actions);
{% endhighlight %}

### File Structure and running

Our server is now complete and should look something like this:

server.js<br>
router.js<br>
utils.js<br>
&nbsp;&nbsp;/endPointHandlers<br>
&nbsp;&nbsp;&nbsp;&nbsp;ping.js<br>

We should therefore being able to CD into the root folder, run and ping it:

{% highlight javascript linenos %}
nodemon server.js

// and make some http requests using curl:

curl -X GET 127.0.0.1:3000/ping
// -> 200 'pong'

curl -X GET 127.0.0.1:3000/pong
// -> 404 'oups'
{% endhighlight %}

* * *

## Final thoughts and resources.

Node is an awesome little beast that doesn't do much by itself but provide us with the right tools to build powerful backends. Much of the boiler plate I went through in the post can be avoided by relying on frameworks like [express.js](http://expressjs.com), that come packed with much moar awesomeness, give it a spin if you have't already.

[Download this blog post code](https://gist.github.com/nickbalestra/5c904e9cbe218ec6649c)
