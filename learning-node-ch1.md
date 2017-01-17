---
layout: default
---

# Chapter1 : The Node Environment

* * *

Node can be installed on almost any machine and used for any purpose

```
for find: ImageMagick for batch photo editing
```


## A Basic Hello World Application

```js
// import module
var http = require('http');

http.createServer(function (request, response) {
  // callback function
  response.writeHead(200, {'Content-type': 'text/plain'});
  response.end('Hello World\n');
}).listen(8124);

console.log('Server running at http://127.0.0.1:8124/');
```

To run on terminal

```
**node hello.js**
```

To close the program
_Ctrl-C_


## Hello World, Tweaked

```js
var http = require('http');
var fs = require('fs');   // File System module

http.createServer(function (req, res) {
  var name = require('url').parse(req.url, true).query.name;

  if (name === undefined) name = 'world';

  if (name == 'burningbird') {
    var file = 'phoenix5a.png';
    fs.stat(file, function (err, stat) {
      if (err) {
        console.error(err);
        res.writeHead(200, {'Content-Type': 'text/plain'});
        res.end('Sorry, Burningbird isn\'t around right now \n');
      } else {
        var img = fs.readFileSync(file);
        res.contentType = 'image/png';
        res.contentLength = stat.size;
        res.end(img, 'binary');
      }
    });
  } else {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Hello ' + name + '\n');
  }
}).listen(8124);

console.log('Server running at port 8124\n');
```

## Node Command-Line Options

```
$ node --help
```

or

```
$ node -h
```


## Node Hosting Environment

finding host

```
https://github.com/nodejs/node-v0.x-archive/wiki/Node-Hosting
```

## Upgrading Node

Node version manager(nvm)

```
https://github.com/vreationix/nvm
```

## Advanced: Node C/C++ Add-ons

node-gyp