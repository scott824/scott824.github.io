---
layout: default
title: "Learning node chapter 2"
date: 2017-01-19 17:27:00 +0900
categories: learning-node
---

# Chapter2 : Node Building Blocks: Global Objects, Events, and Node's Asyncronous Nature

* * *

Node's Global Objects

> _buffer_, _global_, _process_, require, exports, module, and console


## The global and precess Objects

Two fundamental objects in Node are the ```global``` and ```process``` objects

*   ```global``` : similar to the global object in the browser, with some very major differences.
*   ```process``` : pure Node all the way


### The global Object

> In the browser, when you declare a vaiable at the top level, it's declared globally.
> When you declare a varibale in a module or application in Node, the variable isn't globally available.
> you can add variable in ```global``` Object to use variable globally.

### The process Object

1. provides information about the runtime environment
2. standard input/output (I/O) occurs though ```process```
3. you can gracefully terminate a Node application
4. you can even signal when the Node _event loop_ has finished a cycle

```
$ node -p "process.version"
$ node -p "process.env"
$ node -p "process.release"
```

- Standard streams
  - ```process.stdin```: a readable stream for ```stdin```
  - ```process.stdout```: a writable stream for ```stdout```
  - ```process.stderr```: a writable stream for ```stderr```

> The ```process``` I/O functions inherit from ```EventEmitter```

Demonstating standard I/O in Node, and exiting application

```js
process.stdin.setEncoding('utf8');

process.stdin.on('readable', function() {
  var input = process.stdin.read();

  if (input !== null) {
    // echo the text
    process.stdout.write(input);

    var command = input.trim();
    if (command == 'exit') {
      process.exit(0);
    }
  }
});
```


## Buffers, Typed Arrays, and Strings

study again after understand buffer's usage


### Buffer, JSON, StringDecoder, and UTF-8 Strings

### Buffer Manipulation


## Node's Callback and Asyncronous Event Handling

> Javascript is single-threaded, which makes it inherently synchronous.
> However, if you have functionality that needs to wait on something, 
> such as opening a file, a web response, or other activity of this nature,
> then blocking the application until the operation is finished would be 
> a major point of failure in a server-based application.
> The solution to prevent blocking is the event loop.

### The Event Queue (Loop)

There are two approaches to enable asyncronous functionality

1. assign a thread to each time-consuming process
2. adopt an event-driven architecture (the precess signals when it's finished by emitting an event)

Basic web server with additional event highlighting

```js
var http = require('http');

var server = http.createServer();

server.on('request', function(request, response) {
  console.log('request event');

  response.writeHead(200, {'Content-Type': 'text/plain'});
  response.end('Hello World\n');
});

server.on('connection', function() {
  console.log('connection event');
});

server.listen(8124, function() {
  console.log('listening event');
});

console.log('Server running on port 8124');
```

In ```request``` and ```connection``` cases, the events are subscribed to using the ```on()``` function,
which the HTTP server class inherits from ```EventEmitter``` class


### Creating an Asynchronous Callback function

The fundamental structure of the last callback functionality

```js
var fib = function(n) {
  if (n < 2) return n;
  return fib(n - 1) + fib(n - 2);
};

var Obj = function(){};

Obj.prototype.doSomething = function(arg1_) {
  var callback_ = arguments[arguments.length - 1];
  callback = (typeof(callback_) == 'function' ? callback_ : null);
  var arg1 = typeof arg1_ === 'number' ? arg1_ : null;

  if (!arg1) {
    return callback(new Error('first arg missing or not a number'));
  }

  process.nextTick(function() {
    //block on CPU
    var data = fib(arg1);
    callback(null, data);
  });
}

var test = new Obj();
var number = 50;

test.doSomething(number, function(err, value) {
  if (err) {
    console.error(err);
  } else {
    console.log('fibonaci value for %d is %d', number, value);
  }
});

console.log('called doSomething');
```

*   Ensure the last argument is a callback function.
*   Create a Node ```Error``` and return it as the first 
argument in the callback function if an error occurs.
*   If no error occurs, invoke the callback function, 
set the error argument to ```null```, and pass in any relavant data.
*   The callback function must be called within ```process.nextTick()``` 
to ensure the process doesn't block.


If you're interested in developing Addon extensions for Node, you'll need to become very familiar with libuv.
A good place to start is [An Introduction to libuv](https://nikhilm.github.io/uvbook/basics.html).

For more on the interesting, hidden world of multithreaded Node, I suggest the answers to the Stack Overflow question: When is thread pool used?
[Link](http://stackoverflow.com/questions/22644328/when-is-the-thread-pool-used)


### EventEmitter

> Scratch underneath the surface of many of the Node core objects, and you'll find ```EventEmitter```.
> Anytime you see an object ```emit``` an event, and an event handled with the function ```on```, you're
> seeing ```EventEmitter``` in action. The ```EventEmitter``` enables asyncronous event handling in Node.

Example 1

1. attach an event handler to an event
2. emit the actual event

```js
var events = require('events');
var em = new events.EventEmitter();

var counter = 0;

setInterval(function() {
  em.emit('timed', counter++);
}, 3000);

em.on('timed', function(data) {
  console.log('timed ' + data);
})
```

Example 2

```js
"use strict";

var util = require('util');
var eventEmitter = require('events').EventEmitter;
var fs = require('fs');

function InputChecker(name, file) {
  this.name = name;
  this.writeStream = fs.createWriteStream('./' + file + '.txt',
    {'flags': 'a',
     'encoding': 'utf8',
     'mode': 0o666});
};

// inherit the prototype mothods of another, a superconstructor
util.inherits(InputChecker, eventEmitter);

InputChecker.prototype.check = function check(input) {

  // trim extraneous white space
  let command = input.trim().substr(0, 3);

  // process command
  // if wr, write input to file
  if(command == 'wr:') {
    this.emit('write', input.substr(3, input.length));
  }

  // if en, end process
  else if(command == 'en:') {
    this.emit('end');
  }

  // just echo back to standard output
  else {
    this.emit('echo', input);
  }
};

// testing new object and event handling
let ic = new InputChecker('shelley', 'output');

ic.on('write', function(data) {
  this.writeStream.write(data, 'utf8');
});

ic.on('echo', function(data) {
  process.stdout.write(ic.name + ' wrote ' + data);
});

ic.on('end', function(data) {
  process.exit(0);
});

// capture input after setting encoding
process.stdin.setEncoding('utf8');
process.stdin.on('readable', function() {
  let input = process.stdin.read();
  if(input !== null) {
    ic.check(input);
  }
});
```

* ```on``` is same as ```addListener```
* ```ic.once(event, function)```
* ```ic.removeListener('echo', callback)```
* ```EventEmitter.listeners()```


### The Node Event Loop and Timers

Node's event loop is handled by a C++ library, libuv


example of setTimeout

```js
var timer1 = setTimeout(function(name) {
  console.log('Hello, ' + name);
}, 9000, 'Shelley');

console.log('waiting on timer...');

setTimeout(function(timer) {
  clearTimeout(timer);
  console.log('cleared timer');
}, 3000, timer1);
```

example of setInterval

```js
var interval = setInterval(function(name) {
  console.log('Hello, ' + name);
}, 1000, 'Shelley');

setTimeout(function(interval) {
  clearInterval(interval);
  console.log('cleared timer');
}, 6000, interval);

console.log('waiting on first interval...');
```

> if you call ```unref()``` on a timer, and it's the only event in the event queue, the timer
> is cancelled and the program is allowed to terminate. If you call ```ref()``` on the same
> timer object, this keeps the program going until the timer has processed.

example of unref()

```js
var interval = setInterval(function(name) {
  console.log('Hello, ' + name);
}, 1000, 'Shelley');

var timer = setTimeout(function(interval) {
  clearInterval(interval);
  console.log('cleared timer');
}, 6000, interval);

timer.unref();

console.log('waiting on first interval...');
```

* ```setImmadiate()``` and ```clearImmediate()``` has precedence over those created by
```setTimeout()``` and ```setInterval()```. However, it doesn't have precedence over I/O
events.

* ```process.nextTick()``` callback function is invoked once the current event loop is finished.


## Nested Callback and Exception Handling

example of a sequential synchronous application

```js
var fs = require('fs');

try {
  var data = fs.readFileSync('./apples.txt', 'utf8');
  console.log(data);
  var adjData = data.replace(/[A|a]pple/g, 'orange');

  fs.writeFileSync('./oranges.txt', adjData);
} catch(err) {
  console.error(err);
}
```

converted into asynchronous nesed callbacks

```js
var fs = require('fs');

fs.readFile('./apple.txt', 'utf8', function(err, data) {
  if(err) {
    console.error(err.stack);
  } else {
    var adjData = data.replace(/apple/g, 'orange');
    fs.writeFile('./orages.txt', adjData, function(err) {
      if(err) {
        console.error(err);
      }
    });
  }
});
```

example of difficult callbacks

```js
var fs = require('fs');
var writeStream = fs.createWriteStream('./log.txt', {
  flags: 'a',
  encoding: 'utf8',
  mode: 0666
});

writeStream.on('open', function() {
  var counter = 0;

  // get list of files
  fs.readdir('./data/', function(err, files) {
    // for each files
    if(err) {
      console.error(err.message);
    } else {
      files.forEach(function(name) {
        fs.stat('./data/' + name, function(err, stats) {
          if(err) return err;
          if(!stats.isFile()) {
            counter++;
            return;
          }
          // modify contents
          fs.readFile('./data/' + name, 'utf8', function(err, data) {
            if(err) {
              console.error(err.message);
            } else {
              var adjData = data.replace(/somecompany\.com/g, 'burningbird.net');

              // write to file
              fs.writeFile('./data/' + name, adjData, function(err) {
                if(err) {
                  console.error(err.message);
                } else {
                  // log write
                  writeStream.write('changed ' + name + '\n', function(err) {
                    if(err) {
                      console.error(err.message);
                    } else {
                      console.log('finished ' + name);
                      counter++;
                      if(counter >= files.length) {
                        console.log('all done');
                      }
                    }
                  });
                }
              });
            }
          });
        });
      });
    }
  });
});

writeStream.on('error', function(err) {
  console.error("ERROR: " + err);
});
```

1. Start the directory lookup.
2. Filter out subdirectories.
3. Read each file's contents.
4. Modify the contents.
5. Wrtie back to the original file.

We'll use the **Async** module to tackle this _callback spaghetti_
And we'll look to see if ES6 promises can help us.