# ASP.NET MVC 4 Fundamentals - Scott Allen

Check out the release notes for the framework. Embraces WebAPI, `Tasks` for async/await, jQuery Mobile, and Web Optimization for building and minifying

You can start a MVC 4 project by, in VS, going to 'Create new project' > MVC 4, but you can start with an empty solution also. The author starts with a project for all the domain objects as a Class Library. He creates a class representing the specific business entity.

**Protip**: In VS, you can click a whitespace in a definition, slide down across multiple defs, and type things that apply to all you selected.

You can do some powerful things using the `virtual` keyword. Learn to use it.

Each of the MVC 4 templates available have there own things that they bundle and include by default. The author creates a new Internet Application. If you want some of these updates or remove, you'll want to do it through NuGet, not by hand.

MVC is based off of Model, View, Controller architecture. The MVC runtime depends on **routing** to know what controller action to take on each web request, and the routing is set up at app start up. This is defined in `Global.asax` in `Application_Start()`, through `RouteConfig`, `FilterConfig`, and `BundleConfig`. Each of those live in `App_Start` in their own files.

By default, controllers are kept in the `Controllers` folder, but they don't need to be. In order to write a MVC controller, you need to be using `System.Web.Mvc`, your class name needs to have `Controller` as the last part of the name, and needs to inherit from `Controller`. Each method that you include in that class by default can map to a URL in the structure `{controller}/{action}`. You can really return anything from actions, but returning `ActionResults` from your call will return a `View()`. Without arguments, `View()` returns a default view, `{methodname}.cshtml`, that lives in Views > {controller}. You can return specific views by passing in a view name.

The view includes most of the HTML that gets returned to the browser, plus Razor expressions and view logic code. Views can also take advantage of layout template views (similar to master pages), defined in the `Shared` directory. Anything in `Shared` can be used by any controller. The default layout view in the sample project is `_Layout.cshtml`, and uses `@RenderSection()` and `@RenderBody()` for page specific replacement. In your individual views, you mark sections like `@section somename { <!-- html here --> }` and in `@RenderSection()`, you specify the `somename`.

To access a database, you'll want to use EntityFramework which is probably installed to the project by default. To access the db using EF, you'll create a class `SomethingDb` that inherits from `DbContext`, and implements an interface for your class datasource. In `SomethingDb`, you define public `DbSet<T>` for each (table) set you'll pull from. The author describes how to use AutomaticMigrations to make your classes map to table schema.

**rewatch database migrations to see the end**

For your controller to be easier to test, you should not instantiate private fields directly, they should be passed in through a constructor that takes an interface, and that is assigned to a private field. The controller needs a default constructor then, and we can handle that using tools called dependency injectors or inversion of control containers. The author mentions NinJect, but uses StructureMap. Through the installed tooling, you specify what concrete class to use for those constructors defined with interfaces.

Through VS, you can quickly scaffold views based on a controller and a model you want to put in it.