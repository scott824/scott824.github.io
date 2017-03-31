---
layout: default
title: "Learning node chapter 3"
date: 2017-01-26 15:13:00 +0900
categories: learning-node
---

# Chapter3 : Basics of Node Modules and Node Package Manager (npm)

* * *

## An Overview of the Node Module System

Node's module system is patterned after the _CommonJS module system_

* Support is included for a ```require``` function that takes the module identifier and returns the exported API.
* The module name is a string of characters, and may include forward slashes (for identification of paths).
* The module must explicitly export that which is to be exposed outside the module.
* Variables are private to the module.


### How Node Finds and Loads a Module

```js
var http = require('http');
var name = require('url').parse(req.url, true).query.name
var spawn = require('child_process').spawn
```

1. Node checks to see if the module has been _cached_. Rather than reload the module each time.
2. If the module isn't cached, Node then checks to see if it's native module(precompiled binaries).
3. If the module isn't both above, a new Module object is created for it, and the module's ```exports``` property is returned.

> Node only supports one module per file

you can delete the module from cache

```delete require('./circle.js');```

The module is reloaded the next time the application requires it

How Node looks for the modules?

1. A _node\_modules_ subdirectory local to the application (_/home/myname/projects/node\_modules_)
2. A _node\_modules_ subdirectory in the parent subdirectory to the current application (_/home/myname/node\_modules_)
3. Continuing up the parent subdirectories, looking for _node\_modules_, until top-level (root) is reached (_/node\_modules_)
4. Finally, looking for the module among those installed globally (discussed next)


### Sandboxing and the VM Module

read again


## An In-Depth Exploration of NPM

check postit

## Creating and Publishing Your Own Node Module

### Creating a Module

create node module (you have to use `exports` object)

```js
exports.concatArray = function(str, array) {
  return array.map(function(element) {
    return str + ' ' + element;
  });
};
```

we can use this module like this :

```js
var newArray = require('./concatArray.js');

console.log(newArray.concatArray('hello', ['test1', 'test2']));
```

### Packaging an Entire Directory

### Preparing Your Module for Publication

### Publishing the Module
  doesn't need yet

## Discovering Node Modules and Three Must-Have Modules

### Better Callback Management with Async

[Document of Async](http://caolan.github.io/async/).

example1 of waterfall

```js
var fs = require('fs'),
    async = require('async');

async.waterfall([
  function readData(callback) {
    fs.readFile('./data/data1.txt', 'utf8', function(err, data) {
      callback(err, data);
    });
  },
  function modify(text, callback) {
    var adjdata = text.replace(/somecompany\.com/g, 'burningbird.net');
    callback(null, adjdata);
  },
  function writeData(text, callback) {
    fs.writeFile('./data/data1.txt', text, function(err) {
      callback(err, text);
    });
  }
], function (err, result) {
  if (err) {
    console.error(err.message);
  } else {
    console.log(result);
  }
});
```

example2 of parallel

```js
var fs = require('fs');
var async = require('async');

async.parallel(

  // object which include three function
  function() {
    var obj = {};
    for(var i=1; i <= 3; i++) {
      obj['data' + i] = function() {
        var number = i;
        return function(callback) {
          fs.readFile('./data/fruit' + number + '.txt', 'utf8', function(err, data) {
            callback(err, data);
          });
        };
      }();
    }
    return obj;
  }(),

  // last callback
  function(err, result) {
    if (err) {
      console.error(err.message);
    } else {
      console.log(result);
    }
  }
);
```

### Command-Line Magic with Commander

[Commander github](https://github.com/tj/commander.js).

### The Ubiquitous Underscore

[Underscore official page](http://underscore.org/)

[lodash official page](https://lodash.com/docs)
