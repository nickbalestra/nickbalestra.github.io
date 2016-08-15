---
title: Serving Static Files with Node.js
date: 2015-10-29
description: Vanilla node, static file serving and utilities.
---

In this post I'll go through the hassles of serving static files with node.js.

*Disclaimer: Haters gonna hate, lovers gonna love, but no express.js files were harmed in the making, so be warned, pure vanilla node code ahead.*

***

Let's imagine we have a basic node server up and running and that on a directory we host our application files that we want to serve to our users. Read the post on [Building a simple API server with node](http://nick.balestra.ch/2015/build-a-simple-api-server-with-node/) if you need some directions to be up and running quickly with a basic API server in node.js.

### Inside the request handler
The first thing that we need is making sure that our request handler is listening for the right requests. We are interested in two things specifically:

- Requests to root '/'
- Requests relatives to specific assets and static files.
<br><br>

First, let's create a hash of MIME types we want to support, it will act as our whitelist.

{% highlight javascript %}
var MIME = {
  html: 'text/html',
  css: 'text/css',
  js: 'text/javascript',
  gif: 'image/gif'
}
{% endhighlight %}

To serve the main application when a user visit the server URL  (i.e., '127.0.0.1:3000'), we need to be able to serve the index.html. Otherwise,in the case we get requests for any relative static file associated with the app, we need to be able to serve those.
Therefore our request handler module could look something like this:

{% highlight javascript %}

var url = require('url');
var utils = require('./utils');
var fs = require('fs');

var routes = {
  '/ping' : require('./endPoints/ping')
};

var MIME = {
  html: 'text/html',
  css: 'text/css',
  js: 'text/javascript',
  gif: 'image/gif'
}


module.exports = function(request, response) {
  var pathname = url.parse(request.url).pathname;
  var routeHandler = routes[pathname];
  var isFile = pathname.match(/^.*\.(html|css|js|gif)$/);
  var fileToServe = (isFile) ? isFile[0] : 'index.html';
  var contentType = (isFile) ? MIME[isFile[1]] : MIME.html;

  if (routeHandler){
    routeHandler.requestHandler(request, response);

  } else if (isFile || pathname === '/') {
    utils.sendFileContent(response, "client/" + fileToServe, contentType);

  } else {
    utils.respond(response, 'Oups', 404);
  }
};
{% endhighlight %}

### Send File Utility

Having some utilities we can always reuse will make our life a little bit easier. We can then abstract the work of having to parse each static file requested to being able to send it back with the response in a handy little function:



{% highlight javascript %}
var fs = require('fs');

exports.sendFileContent = function(response, fileName, contentType){
  fs.readFile(fileName, function(err, data){
    if(err){
      response.writeHead(404);
      response.write("Not Found!");
    }
    else{
      response.writeHead(200, {'Content-Type': contentType});
      response.write(data);
    }
    response.end();
  });
}
{% endhighlight %}

***

## Final thoughts and resources.

Node is an awesome little beast that doesn't do much by itself but provide us with the right tools to build powerful backends. Much of the boiler plate I went through in the post can be avoided by relying on frameworks like [express.js](http://expressjs.com).Obviously, express come packed with much moar awesomeness then that, give it a spin if you haven't already.

[Check some example code](https://github.com/nickbalestra/chatterbox-server/tree/master/server)
