AngularJS for .NET Developers

Using Grunt within Visual Studio

* Must have Node and Grunt installed on the building machine in order to use Grunt
* Need a package.json file for grunt. Dependencies for building with grunt are added to the `"devDependencies"` property of your JSON. Run `npm install` in your app to have all of these pulled in.
* Need to write a `gruntfile.js` to hold all of your grunt tasks
* Run a task by running `grunt $taskname`
* Can use grunt to minify JS or CSS, clean build directories, lint?, run tests, probably zip up files
* You can write a `default` task that will run when you simply execute `grunt` without params
* This can be executed as a part of an MSBuild file
* Right click the VS project > 'Properties' > 'Build Events'. Here you can add pre-build and post-build events. Need to add a batch file that executes grunt, since the build can't run shell commands directly.