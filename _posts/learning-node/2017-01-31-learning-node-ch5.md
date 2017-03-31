---
layout: default
title: "Learning node chapter 5"
date: 2017-01-31 15:55:00 +0900
categories: learning-node
---

# Chapter5 : Node and the Web

## The HTTP Module: Server and Client

Server that listens for a POST and processes posted data

```js
var http = require('http');
var querystring = require('querystring');

var server = http.createServer().listen(8124);

server.on('request', function(request, response) {
  
  if (request.method == 'POST') {
    var body = '';

    // append data chunk to body
    request.on('data', function(data) {
      body += data;
    });

    // data transmitted
    request.on('end', function() {
      var post = querystring.parse(body);
      console.log(post);
      response.writeHead(200, {'Content-Type': 'text/plain'});
      response.end('Hello World from Server\n');
    });
  }
});

console.log('server listening on 8124');
```

HTTP client POSTing data to a server

```js
var http = require('http');
var querystring = require('querystring');

var postData = querystring.stringify({
  'msg': 'Hello World from client!'
});

var options = {
  hostname: 'localhost',
  port: 8124,
  method: 'POST',
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded',
    'Content-Length': postData.length
  }
};

var request = http.request(options, function(response) {
  console.log('STATUS: ' + response.statusCode);
  console.log('HEADERS: ' + JSON.stringify(response.headers));

  // get data as chunks
  response.on('data', function(chunk) {
    console.log('BODY: ' + chunk);
  });

  // end response
  response.on('end', function() {
    console.log('No more data in response');
  });
});

request.on('error', function(err) {
  console.log('problem with request: ' + err.message);
});

request.write(postData);
request.end();
```


## What's involved in Creating a Static Web Server

