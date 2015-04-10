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

When doing AJAX in WebForms, drop a `asp:ScriptManager` control on your Code Forward. Add a `<Services>` tag inside the ScriptManager with a `<asp:ServiceReference Path="YourService.asmx />"` inside that. On your service class, add a `[ScriptService]` attribute to the class, then your Service can be used with your scripts. This generates JavaScript for you where you basically use server side class and methods in your JS. Use it like `var ret_value = Namespace.Class.Method();`

Remember the class specified in the .asmx file needs to match the .asmx Code Behind.

