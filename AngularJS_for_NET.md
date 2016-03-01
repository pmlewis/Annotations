# AngularJS for .NET Developers #
Joe Eames And Jim Cooper

This course assumes you have a basic understanding of AngularJS, and enough Node knowledge to use Grunt. This also assumes you'll be using ASP .NET, as you are a .NET developer and you're writing web apps.

Angular was built with making client side functionality easier, and more testible. The people who worked on Karma have had a role in Angular.

Angular offers two-way binding, for keeping your view up to date with your model. Views in Angular are declarative.

Angular offers the next step up in clientside development where a significant amount of JS is being written to handle interactions to the point where it becomes unwieldy. After this point, your web app may end up becoming a miniSPA or a full SPA.

To get to the point of an Angular mindset, away from a jQuery mindset, the authors say to not start thinking with a full HTML page, and think of tacking on JS to it. Angular starts with describing your page in smaller, reusable components. Components may look like a tag that may not be valid HTML, but Angular will take care of that.

Really though, if you're starting with Angular, you're probably doing it work if you're going back to jQuery and working with the two hand-in-hand. If there's nothing you can do about it, fine, but leverage the framework you have.

## Organizing Your Code ##

The course describes file organization for Angular projects, where the most basic and least scalable is to have structure like

```
    App
        js
            controller1.js
            controller2.js
            service1.js
            service2.js
        css
        partials
            view1.html
            view2.html
        img
        lib
```
After that, you're probably going to create subfolders in js for each type, but this still gets too large too quick. Then, you're probably going to group your files by component or feature, then each of those contains the controllers, services, directives, etc for that component. This works for medium to large projects.

Organizing by feature, then subfeature with related files is what seems to work best for large organizations.

### Modules ###

Angular supports the idea of modules, bundles of functionality. Your app will ultimately be a module, probably composed of submodules. Modules can depend on other modules like in

```
// To declare modules and dependencies
angular.module('app', ['app2', 'app3']);
// app2 will be run before app3 runs for app
```

Think of modules as a composition of controllers, services, and directives. Angular uses the Injector to load your modules and Angular objects and hook them into your page structure. The Injector is a singleton.

If you're organizing your modules, the most basic way, and least scalable and reusable, is to have your app as one big module. A step up from there is two modules, your "main" module and your "common" module which contains all your reusable code. Then you have main module and multiple dependent modules.

### Naming strategies ###

You'll want to have a naming strategy so you can quickly find exactly what you're looking for, and to avoid Namespace collisions, say if two files are named the same thing. Angular will not puke if this happens, so be careful. For example, you'll collide if two files from two different modules share the same name. The author recommends using an abbreviation for the module or functionality as a prefix to your angular object.

For controllers, its not unusual to find them named with the suffix "Ctrl", so for a `Schedule` module, you might find a controller file named `ScheduleCtrl.js`. For services, the suffix "Svc". Filters may just be "Filter".

Directives can be a little different. Having a directive in HTML must use kabob case, so you could name your directive in kabob case.

Partials (your "view"), can be named like your module, or maybe append "Display".

Really though, the conventions you choice are less important than simply having one and sticking to it. Your conventions should serve your needs, quickly finding what you need and not having accidents, just have it be a little sane.

## Getting Ready for Prod - Using Grunt within Visual Studio ##

* Must have Node and Grunt installed on the building machine in order to use Grunt
* Need a package.json file for grunt. Dependencies for building with grunt are added to the `"devDependencies"` property of your JSON. Run `npm install` in your app to have all of these pulled in.
* Need to write a `gruntfile.js` to hold all of your grunt tasks
* Run a task by running `grunt $taskname`
* Can use grunt to minify JS or CSS, clean build directories, lint?, run tests, probably zip up files
* You can write a `default` task that will run when you simply execute `grunt` without params
* This can be executed as a part of an MSBuild file
* Right click the VS project > 'Properties' > 'Build Events'. Here you can add pre-build and post-build events. Need to add a batch file that executes grunt, since the build can't run shell commands directly.
