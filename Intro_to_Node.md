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

You can require modules from built-in modules, your project files (each .js file is its own module and can be referenced with relative or absolute directories), or through NPM.

You export your variables, functions, etc, to external code by adding them onto `module.exports` and assigning them values.

NPM modules are installed to a project by invoking `npm install`, and this pulls copies of the required modules into your project. Some NPM utilities can be run from the command line, and should be installed with `npm install -g` for global. 