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

## Using Angular with ASP .NET MVC ##

In the grand scheme of things, you could have your web page be entirely an html page with Angular object talking to web services, but perhaps you have an existing MVC project, or perhaps you want to have your web page work without JavaScript. Using Angular lets you have both.

Get started by creating your MVC project with a simple View that is returned. You can decorate your View with HTML and Angular markup, and if you drop in your script reference to Angular and have your `ng-app`, after MVC processes the View, Angular will process it when the HTML is returned to the client. You can consider including your script into your _Layout.cshtml

When you're considering getting data to your Angular app, there's two strategies: either having your app start up calling AJAX, or "bootstrapping" your page with data on page load, meaning that your page has embeded in it, rendered in it, the data it needs to start.

Check out `bootswatch.com` for some free Bootstrap templates you can work with. Also, if you add your client libraries through NuGet, you can click and drag the downloaded file into your View file, and Visual Studio will add a link or script reference to it.

If you really need to, you can leverage the `ng-init` attribute to bootstrap some data in the scope of your child elements. Another (not excellent) strategy is to set some global JavaScript, and then you *need to assign that global value to a property of `$scope` in your Angular controller you define for the sections of mark up. Remember, say, in the case of `ng-repeat`, your expression is an **Angular expression, not plain JavaScript**.

Really though, even if you have some bootstrapped data, it's better to still leverage an Angular service that returns the data, even if that's the only data it will ever get. For example, if you had a module named `mymodule`, your bootstrap data code could look like

```
<script type="text/javascript">
    mymodule.factory("bootstrappedData", function() {
        return {
            courses = @Html.Raw(Model.Course)
        };
    });
</script>
```

You could then pass that service into your controller when you register it.

The author shows a process for setting up your existing MVC app to leverage Angular for your views and creating a mini SPA out of it, letting you remove some existing controllers and views while being more sophisticated with client routing.

Client routing however may have implications on deep linking in your app. The author recommends adding a "catch all" route that can accept any url with any parameters, and then passing that to your mini SPA controller with your default action as `defaults`.


## Communicating with the Server ##

This section focuses on making requests to your MVC app from Angular using AJAX.

You'll want to create a service that gets passed to your controller, and that service will probably use the `$http` service to make AJAX calls to you MVC Action Methods. The author leverages `$q` for managing JavaScript Promises to your controller from your service that uses `$http`.

Remember that if you're sending Json from your MVC Action Method and you're using JSON.NET, don't forget to implement a means of serializing your classes into camelCase to fit in with JS conventions. The author shows means to only allow GET requests to your Action Method when returning Json.

For sending POST's to your server, create an Action method and decorate it with your `HttpPost` attribute. In your Angular view, if you are sending a "form", say with a button click, and you have a bunch of `<input>` tags decorated with `ng-model`, your button can use `ng-click` to call your Angular control's save method. Use `$http.post` to the route of your MVC action method, and pass you model data.

After you send data with your AJAX call, you'll want to show something after it sends, whether the POST was successful or not to the user. Chain your post call with `success`, `error`, and `then` callbacks. With Angular, you can use `ng-show` directives to show sections of html based on conditions, like if you set a `$scope.error` value to true or false.

You can really use either MVC or WebAPI as your endpoint when making Angular apps in .NET, as long as you are able to hit the endpoints and methods of your API controller and it returns some data you can make your app understand. If you are just using Web API, you can work with plain HTML files and no longer use MVC Views. If you do use HTML, you can leverage URL Rewriting to tell IIS to redirect to, say, /registration.html when the client sees /Registration.

The author shows how you can use angular-resources and the '$resource' service that can be cleaner when making URL requests to your server. '$resource' has promises built in and can include other useful data regarding your request and the resource returned. Having resources also handles your data binding, so it can handle page updating when the request completes. '$resource' is meant to hit RESTful endpoints, keep that in mind.

You can combine .NET model validation and html5 validation of form data if you wish and Angular can work with them. If you have a html form, and you want to use validation for each input, add the `novalidate` attribute to the form, then add any other validation attributes to your inputs. You can make use of a directive like `ng-pattern` that validates the input against some regex. `ng-show` can help with hiding and showing elements in case of, say, invalid values to the model. `ng-disabled` works similarly to `ng-show`, but selectively disables elements. Your controller's model has a property `.$invalid` that you can check and this should account your markup validation you put in.


## Using Angular with Legacy .NET ##

This will mostly be focused on getting Angular into WebForms. You can use Page Methods, though it's pretty constrainted and limited. You can add MVC to a WebForms project and use an MVC controller for your Ajax calls from Angular. You can also use RESTful WCF services, and Web Services too.

However your pages are set up, don't forget to add your script reference to Angular.js, whether it's through adding it to your Master page for your site/page, or adding it to your Head controller.

You'll probably want to keep your Angular app code separate from your framework code. The author stashes his in `app/` in his web root. `app` is fairly conventional for Angular. Your app files need to be referenced on your page also. You'll want to be conservative on where you place your `ng-app` attribute on your page, if you load it on most/all of your "webpages" and you're only using Angular on a few of them, try to load it only on those pages.

The author makes use of an asp:Content control at the end of the body of his Master page, then any Web Forms pages can load its required scripts and reference the Content control.

For bootstrapping your app, you can decide to just template your Server Side data into an inline script on your Code Forward if your data is really simple and you just need to get it done. The author uses an asp:Repeater to inline script an array of data between brackets in some JavaScript, or even use an asp:Literal to construct the JS serverside, then pass it into the Code Forward.

Page Methods are pretty simple to use, but are restricted to the Web Form you're on. You decorate your controller methods on an aspx file with `[WebMethod]`, then you can call it with some Ajax. On the client side, WebForms generates client code for using PageMethods, and each method gets namespaced to the `pageMethods` object on the page. You then need to include your runat server form and an asp:ScriptManager with `EnablePageMethods="True"`. You can then call your Page Methods from your Angular app, but always remember on your callback functions you supply to each client Web Method call to end it with `$scope.$apply()` to rebind the updated model to your page.

If you're are using a service to call a page method, and you don't have access to `$scope`, you can reinitialize Angular by passing the `$rootScope` service to your service and call `$rootScope.$apply()`, however this is kind of hacky.

To save things using Page Methods, you call your Page Method from the client passing the needed parameters, and save as usual from your Web Forms code.

## Getting Ready for Prod - Using Grunt within Visual Studio ##

* Must have Node and Grunt installed on the building machine in order to use Grunt
* Need a package.json file for grunt. Dependencies for building with grunt are added to the `"devDependencies"` property of your JSON. Run `npm install` in your app to have all of these pulled in.
* Need to write a `gruntfile.js` to hold all of your grunt tasks
* Run a task by running `grunt $taskname`
* Can use grunt to minify JS or CSS, clean build directories, lint?, run tests, probably zip up files
* You can write a `default` task that will run when you simply execute `grunt` without params
* This can be executed as a part of an MSBuild file
* Right click the VS project > 'Properties' > 'Build Events'. Here you can add pre-build and post-build events. Need to add a batch file that executes grunt, since the build can't run shell commands directly.
