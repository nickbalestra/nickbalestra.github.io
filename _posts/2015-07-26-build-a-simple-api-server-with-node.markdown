---
title: Building a simple API server with node
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
That's mostly about it as Node isn’t really much more than that.

* * *

## The Architecture

### A basic server
The first thing that we need is to have a live server up and running listening for requests on a specific address:port. In other words we just need an address to expose our public API for the external world to communicate and interact with. For this we'll need to rely on the node http module. After loading it we can then create a server via its createServer() method. createServer() accept a function as parameter and automatically add it to the 'request' event (registering it as a listener). Yes, right.. enough said, here is the code:

{% highlight javascript linenos %}
var http = require("http");
var router = require('./router');

var port = 3000;
var ip = '127.0.0.1';

var server = http.createServer(router);
server.listen(port, ip);
{% endhighlight %}

### A router module

So, we now have a new instance of http.Server listening for requests on 127.0.0.1:3000. Every time a 'request' event triggers, the router will be called and the request object passed to it. The router should checks the url of the request against a list of registered routes (a.k.a our public API endpoints) and route the request to the relative endpoint handler, or, in case such a route wasn't registered, send back a 404 response. Here the code for the router and it routes hash:

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

For the sake of the post our server will just have a single valid endpoint '/ping' registered inside the routes hash. The handler for the endpoint will support GET requests only, responding back with a simple and neat 'pong'.

Before being able to work on the ping endpoint handler we need to put together some helpers to make our life easier and dryer.

### Utils

At a very besic level we don't need much, is enough to have:

**respond()** to build and send the http response.<br>
**fetchData()** to grab the data passed within each request (we won't take advantage of it throughout this post, but is definitely a must have).<br>
**actionDispatcher()** to check for requests methods supported inside each endpointHandler and dispatch the request accordingly.

Inside the utils file we can also define some headers to be used within our responses, allowing us to define CORS headers and any other header property we may need. As we are at it, and we are building an API server let's add json as the content type of our headers:

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

Now we should have everything we need to easily create the endPointHandlers with the logic and definition for each endpoint of our API. For our example server we said that we want to have a unique endpoint ('/ping') that will respond only to GET requests.

Let's require our freshly backed utils, to support us handling both the requests and responses as well as the actions hash like a boss. The actions hash is where we will define the logic for each http method we may want to support/allow for the given endpoint. Here the final ping endpoint handler definition:

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

Our server is now complete and it's directory structure should look something like this:

server.js<br>
router.js<br>
utils.js<br>
&nbsp;&nbsp;/endPointHandlers<br>
&nbsp;&nbsp;&nbsp;&nbsp;ping.js<br>

Boom!, we should now be able to CD into the root folder, run it and ping it via curl:

{% highlight javascript linenos %}
nodemon server.js

// make some http requests using curl:

curl -X GET 127.0.0.1:3000/ping
// -> 200 'pong'

curl -X GET 127.0.0.1:3000/pong
// -> 404 'oups'

curl -X POST 127.0.0.1:3000/ping
// -> 404 'oups'
{% endhighlight %}

* * *

## Final thoughts and resources.

Node is an awesome little beast that doesn't do much by itself but provide us with the right tools to build powerful backends. Much of the boiler plate I went through in the post can be avoided by relying on frameworks like [express.js](http://expressjs.com).Obviously, express come packed with much moar awesomeness then that, give it a spin if you have't already.

[Download this blog post code](https://gist.github.com/nickbalestra/5c904e9cbe218ec6649c)
