# Introduction to ASP .NET Web Services - John Somez #
------------------------------------------------------


## Intro to services

Note that this is a legacy technology that can be surpased by newer technologies like WCF, WebAPI.

We use HTTP, XML, SOAP, and WSDLs for ASP .NET Web Services. HTTP is the transport means. XML defines the structure of the text sent across. SOAP is an XML protocol for defining the "language" of using the service. WSDL (web services description language) defines the interface for interacting with the service and is used for building objects for interacting with the service,

WSDL's have SoapIn and SoapOut messages, defines endpoints and methods of the service.

Web services are marked with an attribute `[WebService]` for classes or `[WebMethod]` for class methods. Service files are .asmx files for the "Code forward". These let you write .Net code to avoid building Soap packets, etc.

**Hello World for Web Services:**

* Create a new project in VS
* Select Web Service as the project type
* Run, make a browser request to the file


## Creating Simple Web Services

The key is to make the interface easy for clients to use.

For a set of web services, we can start with an Empty Web Application. An ASMX page looks kind of like a WebForms file, where you use the WebService directive in the Code Forward, and in the code behind, you define a namespace and classes that have `WebService` and `WebServiceBinding` attributes, and the class may inherit the `WebService` class. Add to the project a new item: Web Service.

WebMethods expose methods through the service by using the `[WebMethod]` attribute, and then write the method as you normally would. *Needs to be serializable, remember this if you are accepting or returning class types that are unique to your platform*. `[WebMethod]` can take params like

```
[WebMethod(Description = "Hello!")]   
public int Echo(string message)   
{   
	return message;   
}   
```

There is are examples of writing a service storing information in the `Session` object to persist information across a session between a client and the server, however, you probably never want to do this in prod. To do this, your web method needs to have the attribute `[WebMethod(EnableSession = true)]`.

Strongly consider always returning a value from your web methods so your client has a clue about what happened when they used your method. For example, return a User ID if a user calls a method `AddUser()`.

To get away from storing data in session, you should use a **Persistence Layer**. Always define an interface to your Persistence layer so you can swap in different means of persisting data, whether it's in memory, in cache, in S3, etc. *Note: You probably shouldn't just store things in memory for prod, but in memory can be good enough for dev.*

***Beware ye who change method signatures of deployed services without a new API version~***


## Consuming Web Services

Example of creating a Windows Forms client of the sample Protein Tracker web service.

Use "Add service reference" to consume the WSDL and generate the proxy objects in Visual Studio. When an API changes, you need to reconsume the WSDL to generate the new proxy objects. Use 'Update service reference' to do so in VS.

Once you use consume the WSDL, in your client code, create a new instance of the proxy class and then use the methods it exposes.

You create proxy classes for the web service by consuming the WSDL file from the service. Remember that the proxy classes make calls over the internet, so it may take a long time to complete, may not complete at all, etc. Make your calls asynchronous whenever possible. Add an `async` before your `public` or `private` in your method signature, and then use the Async versions of the methods your proxy exposes, along with using the `await` keyword before the actual method call.

```
var newTotal = await service.AddValueAsync(newItem.Value);
```
 
The old way of doing this: 'Configure Service Reference', under 'Allow generation of asynchronous operations', select 'Generate asynchronous operations' instead of 'Generate task-based operations'. Remove the `await`, then use the `Begin` versions of your methods to start the call, and use the `End` versions to handle when the method completes.

Exceptions that come across service calls will be wrapped in a `FaultException`, so your client code that calls the proxy methods should use a try/catch block catching FaultExceptions.

Custom SOAP headers for passing tokens or credentials example, but don't do this in prod. Make and use a public class that inherits from `SoapHeader`. After that, use a `[SoapHeader("instance_name")]` attribute on your method. Methods that use a `SoapHeader` need an instance of your class passed into each method call of your proxy class.

If for some reason you can't consume a WSDL, you can manually create a proxy class. Use `wsdl` on the Windows Developer Console with a url to the WSDL file.


##  Using Web Services with AJAX in Web Forms

Add a WebForms file in your Web Services project. You can add client frameworks like jQuery through NuGet, and it'll fetch your scripts, drop them in a Scripts folder, and add them to the project.

When doing AJAX in WebForms, drop a `asp:ScriptManager` control on your Code Forward. Add a `<Services>` tag inside the ScriptManager with a `<asp:ServiceReference Path="YourService.asmx />"` inside that. On your service class, add a `[ScriptService]` attribute to the class, then your Service can be used with your scripts. This generates JavaScript for you where you basically use server side class and methods in your JS. Use it like `var ret_value = Namespace.Class.Method();`. With these JS calls, you pass in parameters in matching order as the server side definition, but you can also add a last extra parameter that is a function that can use the return value from the call as a parameter. Like so:

```
//Serverside
[WebMethod]
public int AddToTotal(int addme)
{
	//...
	return total + me;
}

//Clientside
Namespace.Class.AddToTotal(addme, function (sum) {
	console.log("The new total is " + sum.ToString());
});
```

Remember the class specified in the .asmx file needs to match the .asmx Code Behind.

Suppose you want to add WebMethods directly to the WebForm itself without using a separate .asmx?

1. Add `EnablePageMethods="True"` attribute to your ScriptManager
2. Include your WebMethods into you WebForms Code Behind
3. Make each of your WebMethods `public static`
4. In your clientside code, replace `Namespace.Class.MyMethod()` with `PageMethods.MyMethod()`

Don't want to use ASP ScriptManagers and still want to make AJAX calls to your web methods? Good for you, here's how:

1. Remove your ScriptManager
2. Make an AJAX call to the WebMethod. You need to POST to the URL, which in this example is "ProteinTracker.aspx/AddUser" where `AddUser` is the web method.
3. The parameters need to be passed as `data` in a stringified JSON object
4. The content type needs to be `application/json; charset=utf-8` and datatype needs to be `json`
5. You should be able to add success and failure handlers as methods

In jQuery, this can look like:

```
$.ajax({
	type: 'POST',
	url: 'ProteinTracker.aspx/AddUser',
	data: JSON.stringify({ 'name': name, 'goal': goal}),
	contentType: 'application/json; charset=utf-8',
	dataType: 'json',
	success: PopulateSelectedUsers //Note that this is a function, not a function call
});
```


## Migrating to Newer Technologies

Why?

* Web Services is limited to just SOAP based services
* SOAP is pretty heavy and verbose, which has more impact on mobile clients.
* JSON and REST is simpler, lighter, and encourages the creation of documentation for humans.

Be aware though that there may be some advantages for using SOAP and building proxy classes off of WSDL. Choose the right tool for the right job.

### Newer Technologies

WCF - lets you define the protocol and data format. You could choose to use SOAP, or you can choose something else. Even lets you choose binary formats.

Web API - focuses on REST and JSON. Oriented towards MVC architecture, and is pretty light weight.

Non MS solutions - for example, ServiceStack

### WCF

Uses .svc files. The code from .asmx files can largely be copied into these files, but the output generated from these files won't be backwards compatible. Beware of any clients using your service, they will need to reconsume a WSDL.

Define an interface that basically replaces the .asmx class, and you can move all the attributes describing the class and its methods to the interface. Your class then just needs to implement the interface. You don't even need to re-apply the attributes to the class.

Add a new item, either WCF Service or AJAX-enabled WCF Service. Be aware that to use AJAX-enabled WCF, there's additional setup that you have to look up how to use.

wCF uses `[ServiceContract]` and `[OperationContract]` instead of `[WebService]` and `[WebMethod]`, but you can remove these, or all the code behind for that matter, and in the .svc code forward, change the service to the class you created back in Web Services. Now back in the class, add the `[ServiceContract]` and the `[OperationContract]` to the class and methods in addition to it having all the other ones for Web Services.

### Web API

Go ahead and start a new project, it's different enough from Web Services to warrant it. Create an MVC project, and select Web API template.

Using REST style architecture relies on having a more thorough understanding of HTTP, since you'll be using HTTP verbs as a part of your calls.

The controller created with the project will inherit from ApiController. To convert a Web Service to using an ApiController, you will basically have to re-implement the service since Web Services is SOAP-based.

In your project, right click Controllers and do "Add new controller", selecting "Empty API controller".  If you need to add a class that doesn't need to be a controller, it will probably go under "Models".

### Using Http verbs

Get - should be used for querying, asking for results, getting things. If your method returns a collection of items, use `IEnumerable<Whatver>` as your return type. If you use a method `Get()` on your control, a plain old GET request to your site.com/api/controller will return that. By default returns XML, but is easy to switch to JSON.

Post - should be used for creating new data. The parameter to the Post method in your ApiController will typically have a `[FromBody]` decoration ala `public int Post([FromBody]CreateUserRequest request)`

Check out "REST Console" plugin for Chrome to easily test RESTful APIs.

Put - can be used for adding new data or associating data to some bit of already existing data. You can do something like `Put(int id, [FromBody]int amount)` so requests to it would be PUT to a url api/user/12, and in the body, you'd have something like `{amount: 100}`.

Delete - when you want to delete some bit of data
