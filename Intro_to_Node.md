Getting Started with Node.js - Paul O'Fallon
============================================

Three big components

1. libuv - cross-platform evented I/O library
2. V8 - Google's JavaScript engine
3. Custom js and c++ for the node platform

Node versions can be managed with `nvm`. Git clone this, since there Node updates frequently. You can use `nvm` to install different versions of Node, or you can use a package-manager or binary installs.

Running `node` starts a REPL. You can use `console.log()` to print out. You can write JavaScript code in .js files then run `node hello.js` to run the script. You can use the `http` module to create a simple running web server with `http.createServer`.

The author uses Cloud9 as an online IDE for Node dev. If you are running a web server on Cloud9, be sure to set the listening port and IP to `process.env.PORT` and `process.env.IP`.

### Event Loop

Browsers have an event loop that runs constantly, listening for events to occur. Node also has an Event loop running on your server. Could be listening for http requests, timers, generated events, etc. Note that your events will probably interleave with each other. Http requests will not block and executing code will not block each other.

The non-blocking approach will often look different, where functions will be nested as params to be executed once its caller completes. Event Emitters will generate events for you that a callback function can be executed when it sees an event emitted. The callback in an async call is by Node convention always the last parameter. The first parameter to a callback should be also an error object, try to avoid using exceptions and try/catch blocks. Check for false error values for no errors.

Leverage anonymous functions as callbacks and use closures for making your life easier. Be aware though that you *can not guarantee when callbacks will execute or in what order*.

### The Christmas Tree Problem

Beware of nesting anonymous callbacks within anonymous callbacks within anonymous callbacks.... This creates code that looks like a Christmas tree and turns out to be messy and not elegant. Be smart and use event emitters, named functions, and others when you can.

## Modules, `require`, and NPM

External code files are loaded into your code with modules. Use something like `var foo = require('foo');` Convention: modules that export variables and/or functions are referenced with camelcased variables, modules that export objects should be assigned to vars with leading Uppercase.

You can require modules from built-in modules, your project files (each .js file is its own module and can be referenced with relative or absolute directories), or through NPM installs.

You export your variables, functions, etc, to external code by adding them onto `module.exports` and assigning them values.

NPM modules are installed to a project by invoking `npm install`, and this pulls copies of the required modules into your project. Some NPM utilities can be run from the command line, and should be installed with `npm install -g` for global. Check out [npmjs.org](npmjs.org). You can publish a module to NPM by including a `package.json` file with the required members, then create an NPM account, then run `npm publish .` from the project root. Double check your push by mkdir'ing a new dir, then run `npm install yourmodule` in there.


## Events and Streams

Node is really focused on being asynchronous, but that doesn't always mean use of callbacks. Use events and Node's `EventEmitter` class. EventEmitting can reduce used memory, enable acting on results as soon as they arrive, and deliver partial results. For example:

```
//Callback style
getThem(param, function(err, items) {
	//check for error
	//operate on array of items
});

//Event driven, using EventEmitters
var results = getThem(param);

results.on('item', function(i) {
	//do something with each one item
});

results.on('done', function() {
	//No more item handler
});

results.on('error', function(err) {
	//Error handling
});
```

Events can be emitted with arguments, like `emitter.emit(event, [args]);`

```
var EventEmitter = require('events').EventEmitter;

var getResource = function(c) {
	var e = new EventEmitter();
	process.nextTick(function() { //On the next cycle of the event loop(?)
		var count = 0;
		e.emit('start');
		var t = setInterval(function () {
			e.emit('data', ++count);
			if (count === c) {
				e.emit('end', count);
				clearInterval(t);
			}
		}, 10);
	});
	return(e);
};
```


Object can also extend EventEmitter, using `util.inherits(yourResource, EventEmitter)` where you'd have...

```
//Resource.js
var util= require("util");
var EventEmitter = require("events").EventEmitter;

function Resource (m) {
	var maxEvents = m;
	var self = this;
	
	process.nextTick(function () {
		var count = 0;
		self.emit('start');
		var t = setInterval(function () {
			self.emit("data", ++count);
			if (count === maxEvents) {
				self.emit("end", count);
				clearInterval(t);
			}
		}, 10);
	});
}

util.inherits(Resource, EventEmitter);

module.exports = Resource;

//app.js
var Resource = require("./resource");

var r = new Resource(7);

r.on("start", function () {
	console.log("I've started!");
});

r.on("data", function (d) {
	console.log("	I received data -> " + d);
});

r.on("end", function (t) {
	console.log("I'm done, with " + t + " dataevents.");
});
```

### Streams

Instances of, and extensions to, EventEmitters with a written "Interface". Think like file streams or i/o streams. These streams can be piped, like in bash pipes. Two common interfaces seem like ReadableStream and WritableStream. ReadableStream has a `pipe()` function, a `pause()` and `resume()`

The `request` function and module uses a ReadableStream. `process.stdout` is a WriteableStream. The `fs` module and object can create Streams to the local filesystem.


## The Local System

### The `process` object
stdin, stdout, stderr, env, argv, uptime(), process.memoryUsage(), abort(), chdir(), etc. Emits 'exit', 'uncaughtException', POSIX standard events, etc.

Need to run `process.stdin.resume()` to begin receiving info from stdin, it starts paused.

### The `fs` module
Writing and reading functions and streams for the file system. A lot have both synchronous and asynchronous implementations. POSIX functions like chown, rmdir, mkdir, symlink, etc. Also has a `watch()` function for watching for file or directory changes.

### Buffer objects
A Buffer handles raw binary allocation, and is needed in the case of dealing with networking and file system access. Buffers can be converted to/from different encodings. With Buffers to Strings, you can specify a portion of the buffer to be returned.

### The `os` module
Info about the currently running computer that the code is running on. Lets you pull up some interesting statistics.


## Interacting with the Web

### Making Web Requests
Use the `http` module. Can use the `http.request()` method to send requests. Add a param for the URL to hit, or an object that lets you specify host, port, headers, method, etc. Returns `http.ClientRequest` and feeds into the callback a `http.ClientResponse` object. Emits events for response coming back. `http.get()` is also available for simple GETs.

Be aware that `http` is a lower level object than `request`. For example, the `http` object may get a response that is a 304 redirect, the `http` object won't follow the redirect.

### Making a Web Server
A simple server may look like

```
var http = require("http");
var server = http.createServer(function(req, res) {
	//process request
});
server.listen(port, [host]);
```

Each request is provided via a callback like above, or as a `request` event on the server object. There is also a `https` object for SSL. http.createServer returns a `http.ServerRequest` stream and passes into its callback a `http.ServerResponse` stream.

A server that replies to everything could look like:

```
http.createServer(function(req, res) {
	res.writeHead(200, { "Content-Type" : "text/plain"});
	if (req.url === '/file.txt') {
		fs.createReadStream(__dirname + "/file.txt").pipe(res); //__dirname is a special var for current directory
	} else {
		res.end("Hello World");
	}
});
```

**[Socket.IO](socket.io)** - a library aiming to handle web socket connections between the client and the server. There is a 7 minute demo on how to use it. Be aware that many browsers may not support web sockets yet.

For developing Web Apps in Node, you'll want to levarage a framework like [Express](http://expressjs.com/). Check out the course on using Express.


## Testing and Debugging

### The `assert` module
You can assert like assert in C languages like

* test equality
* test exceptions thrown
* truthiness
* whether error was passed to a callback
* asserts can contain a message when there is a failure of the test

There is `assert.equal()`, uses `==`. `assert.strictEqual()`, uses `===`. `assert.deepEqual()` for deep value checking.

### Mocha and Should.js
Frameworks for testing. Mocha includes a lot of utilities for making debugging easier, organizes tests into suites, runs tests serially, which tests are slow, can re run tests on directory change, and more. Check it out

Should.js extends the `assert` module, and adds `should.have` or `should.equal` functionality to objects.

### Debugging with Cloud9
Node.js provides hooks for debugging, and Cloud9 leverages them inside its debugging console, and includes pausing execution, breakpoints, arbitrary code execution, value inspection, and more.

**Also**, there are integrations for Node into IIS available.


## Scaling Node Apps

### Creating child processes
Node doesn't do well with long running processes since it may slow down the event loop. One strategy for dealing with this could be making child processes to free up the event loop and make it run concurrently. Use the `child_process` module with the `spawn` method, and the Events that get emitted. There is also the `exec()` method for running shell commands, and the `execFile()` method. For spawning other Node processes, use `fork()` to launch other modules

### The `cluster` module
Experimental module for creating Worker classes and forking, along with forking events. For example, you can spawn multiple Workers to scale up along with demand. Needs Node 0.8