Modern Structured Logging With Serilog and Seq - Jason Roberts
==============================================================

* Structured - Properties that are meaningful to the app and can be queried
* Serilog - can to plain text, can right structured log to NoSQL with named properties

### Why structured?

No need for regex or awk expressions. Files can be spread across locations and across servers. Structured logs can be quickly queried, like for a particular order, or user, or request time length.

### Serilog

Install through NuGet. Configure location(s) to write, configure rolling files. Can name properties (like property names in JSON) through a logging statement, and you can pass an object like a format string to be written to that property. Can log to MongoDB, Windows Events, SQL Server, ElasticSearch, Console, etc.

### Seq

An app for your app servers as a Windows Service. Provides a web interface for querying your structured logs.

## Serilog Fundamentals

### Getting started

Go to NuGet, install Serilog to your project. You'll want to configure an ILogger like

```
ILogger logger = new LoggerConfiguration()
						.WriteTo.Console()
						.CreateLogger();

logger.Information("Hello new logging!");
```
The property `WriteTo` tells Serilog where to write to and how.

You can cache the created logger into `Log.Logger`, so do `Lo.Logger = logger; Log.Logger.Information("Hello new logging!");`

### Log to a single text file

Use `WriteTo.File("Your_file.txt")` in your configuration. Serilog throws in a Date and Time for you on each line along with the debug level, in this case, `Information`. You can specify a custom template as a `const string` for your logger in case you don't like the default format. Pass this in as a second argument to your `File()` call.

Default file size will be up to 1GB. You can change this through `fileSizeLimitBytes` in the `File()` call. If you set this to `null`, your file will grow without restriction from Serilog (prolly shouldn't do that).

Instead of `File()`, check out `RollingFile()`. Default is a new file each day. Serilog will take care of creating the new files for you. You can configure how many log files you can keep through `retainedFileCount` (think of how many days you'd like to go back through).

### Logging Structured Data

In your string to log, you can do something like `"Added user {UserName}", freshUser` where this generates a structured log. Serilog can read IEnumerables and Dictionaries (with some caveats) and write the enumeration of it to your log. Else, it'll use your objects `toString()` unless you add a destructuring operator (`"Color is: {@Color}", favColor`). You can specify a Destructuring transformation in your configuration for a class.

### Provided Sinks

Sinks are the places Serilog can write to, like Mongo, SQL Server, Windows Event Log, etc. Go to NuGet and install a package for the sink you want to use. Now instead of `File()`, `RollingFile()`, etc, use the one provided by the installed sink. The author goes through an example using the RavenDB sink, and then querying the logging through Raven's web console.

## Beyond the Basics

* Message templating and formatting
* Logging levels
* Multiple logs and sinks
* Custom filtering of messages
* Adding additional properties to the structured data.
* Extras features, like for ASP .Net, web.configs, operation timing.