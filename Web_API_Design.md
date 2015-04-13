# Web API Design - Shawn Wildermuth

This is how to design RESTful APIs, not on how to implement an API using a certain technology.


## Introduction

### What are Web APIs?

HTTP/RPC vs RESTful services - RPCs will have verbs in its API and may be stateful. RESTful uses http verbs and a stateless servers.

HATEOAS - hypermedia as the engine of application state - stateless, links inside payloads to lead to the next step in a given application given its context

Resource Based Architectures - Represents real world entities. Relations are typically nested and accessible through URIs. Often describes a hierarchy, but not relation.

REST - Representational State Transfer - Fielding's dissertation emphasizes separation of client and server, that the requests to servers are stateless, that requests are cacheable wherever possible, and that the interface is uniform across the API. Wildermuth says that to be to-the-tee RESTful does tend to be difficult, so we want to be pragmatic enough that we get the job done without getting caught in dogma.

Hypermedia - the idea of (hyper)links reaching across the internet, and the link describes the linked enough that describe how to process the data.

Should you use REST, and should you be strictly REST or RESTful? - Does using REST limit your application, or are you fitting a square peg into a round hole? Do what makes the most sense, but starting/using a lot of the ideas of REST does at least start solid enough.

Remember that there are a lot of better ideas and a lot of worse ideas, but there are very few black and white answers.

## Designing the API

### URIs

For URIs, use nouns, not verbs. This means `api/getCustomers` should really be `api/Customer` or `api/Customers`. Use identifiers to individual items, like `api/Customer/123`. Remember that your ID doesn't have to be internal, but should always refer to the same item and that ID really shouldn't change.

### HTTP Verbs
HTTP verbs (GET, PUT, POST, DELETE) should be leveraged for manipulation of data. Think GET = read, POST = insert, PUT = update, DELETE = delete. Return values for these calls should either be the value you queried for/ updated/ created, or status codes that indicate success or failure. You really should not be able to support things like `DELETE` to `api/customers` and have it blow away all users.

For PUTs or POSTs, your parameters should be in the body of your request. It's common to use in the header `Content-Type: application/json` and some JSON object in the body like `{ Address: "123 America Rd" }`. Check out using Fiddler to make requests through the 'Composer' tab.

### Status Codes
Try to simplify the amount of status codes you return. Http has a lot of them, but you don't need to use all of them.

**Minimum**

* 200 - OK = "It worked"
* 400 - Bad request = "User error"
* 500 - Internal error = "Somethings wrong on our end"

**Probably**

* 201 - Created
* 304 - Not modified
* 404 - Not found
* 401 - Unauthorized
* 403 - Forbidden

### Associations
Use URI based navigation, a la `api/Customer/132/Invoices`. Query parameters should be used when a simple association doesn't cover it, a la `api/Customer?state=GA&salesperson=144`

### Formatting
Use negotiation with Accept header values to determine the formatting to return, for example `application/json` or `text/xml`. Common Content Types also include JSONP = `application/javascript`, RSS = `application/xml+rss`, ATOM = `application/xml+atom`. Not a best practice, but depending on circumstance, you may just specify formating in the URI, a la `api/Customer/123?format=json&callback=foo`.

### Designing Results
Stick to simple objects, like JSON objects or simple XML. Be consistent in naming your members, shouldn't reveal implementation details, like through naming conventions. When returning collections, consider returning an object wrapping all the actual results, that way, you could additionally provide some helpful metadata, like maybe number of results.

### Entity Tags (ETags)
Can be leveraged for smart caching support, whether it is weak or strong. If using, clients should send etag back to see if a new version is available, and when they do, they should include an `If-None-Match` header with the same value of the etag. Returns 304 if it hasn't changed. For PUTs, use `If-Match` to see if you're working with the same version, if not same, returns 412.

### Paging
Lists should **always** support paging. Use query params to request paging information. Responses should include next/prev links, a la  

```
{
	"totalResults": 1598,
	"nextPage": "http://.../api/games/?page=3",
	"prevPage": "http://.../api/games/?page=1",
	"results": [...]
}
```

It's useful to be able to specify page size as a query parameter to get a certain number back, but always have some kind of upper limit of page size.

### Partial Items
Lets users request only this, this, and that. Can be useful, can use a query string like `api/games/123?fields=id,name,price,imageUrl`. There's a way to support partial item updating through the verb PATCH together with ETag through `If-Match`, but remember you need to support being able to handle this.

### Non-Resource APIs, like functions
This is a bit against REST, but remember, be pragmatic. Calls should be completely functional, should not cause side effects or statefulness. This can quickly become a Remote Procedure Call API if this keeps happening, which very well could be what you need, but don't call it REST. 


## Versioning your APIs

Once APIs are out there, you must treat it as set in stone, and any significant change must be done through versioning, otherwise you'll break your customers code. Unfortunately, there's no single best way to version, just choose one that will work for you and your customers.

**Options**

* URI Path: `api.tumblr.com/v2/user`
	+ Most common
	+ Enables dramatic API changes by super clear separation
	+ Requires a lot of client change as you change
* URI Parameter: `api.netflix.com/catalog/titles/series/123?v=1.5`
	+ Optional Parameter, default can be latest version of your API
	+ Enables API providers to make little under the hood changes without huge releases
	+ Possibility of breaking client code without them knowing. Puts the onus on them to specify a parameter in order to be safe
	+ Could lead to more management behind the scenes for each microversion
* Content Negotiation in request header: `Content Type: application/vnd.github.1.param+json`
	+  Nice separation from more volatile parts of the request, and from API calls themselves 
	+  Adding custom headers can be harder on different platforms
	+  Beware of version churning
* Custom Request Header: `x-ms-version:2011-08-18`
	+ Common to use dates, but header value should *only* impact *your* API
	+ Not tied to resource versioning
	+ Adding custom headers can be harder on different platforms

***Whatever you choose, don't release without versioning***

### Resource versioning?
Not a bad idea, since returned data-structures can change too. Doing this with custom Content Types is easier, but adds complexity. Version in resource body dirties the return data.


## Securing Web APIs

Are you handling personalized or private data? Sending sensitive data? Sending credentials? Protecting against overuse of your servers? If yes, you need security. This includes relying on SSL, but also protecting the API in terms of Cross Origin Sec and Authorization/Authentication.

### Cross Domain Security

Is your clients accessing your API from a different domain? This is cross domain. If the service is only accessible internally, you're probably ok, but if it's public, you need something. There are two options:

JSONP - is actually JavaScript that includes a callback from your request. Use application/javascript as the Accept header. The return body is a JavaScript method, with say the results passed in as the parameter. Now calling that script is not Cross Domain.

CORS - Cross Origin Request Support(?) - requires some handshaking that needs to be implemented. The browser makes a cross origin request, it asks the server is CORS is allowed. The server responds with rules of what's allowed, the browser sends the request with a CORS header with an allowed request, which includes origin and request-method.

### Authentication - Who is calling the API?
Three different typical ways

1. Server-to-server Authentication w/ API Keys and Shared Secrets
2. User Proxy Auth w/ OAuth or similar standards
3. Direct User Authentication w/ Cookies or tokens

#### API keys and Signing
You sign up for an API and a Shared Secret through the API provider, then when you make requests to the API, you sign the request with the secret, send the request along with the signature to the API, and then the API provider can validate the request.

### Authenticating Users
If internal, then relying on website security is probably ok, if you're writing a client app, you need more. If you are exposing an API to other devs, you should use OAuth.

OAuth flow: Dev requests key, API supplies Key and Shared Secret, Dev requests Token, API validates and returns Token. Dev redirects to API's Auth URI, API displays authorization UI to user, user confirms. API redirects back to dev, dev requests access token via OAuth and Request token, API returns access token with timeout, dev uses API with Access Token until Timeout.

You should definitely leverage a library/service/ or 3rd party Identity (like Facebook) to do this for you.


## Hypermedia

Hyperlinks are what connects all the files of the WWW to each other. `<link>` tags also count in hypermedia, and you can describe using `rel` attributes what the linked data is. Remember the HATEOAS abbreviation.

Links returned should help the dev use the API. For example:

```
{
	"totalResults": 1588,
	"links": [
		{
			"href": "http://localhost:5555/api/v1/games?page=1",
			"rel":	"prevPage"
		},
		{
			"href":	"http://localhost:5555/api/v1/games?page=3",
			"rel":	"nextPage"
		}
	],
	"results": [...]
}
```

Consider self documenting links like:

```
{
    "links": [
        {
            "href": "http://.../api/v1/Games/2",
            "rel": "self"
        },
        {
            "href": "http://.../api/v1/Games/2/rating",
            "rel": "rating"
        }
    ],
    "id": 2,
    "name": "Bloodbourne",
    "description": "Hard"
}
```

### Profile Media Types

Profiles describe data returned, and is an alternative to customer MIME type in versioning. Include this in Accept header like `Accept: application/json;profile=http://awesomeapp.com/Games`

### Standards for Hypermedia
This are currently emerging standards. HAL, and Collection+JSON

HAL - hypertext application language - lightweight. Supports json and xml. Content types include `application/json+hal;profile=http://awesomeapp.com`. Returned objects look like

```
{
        "_links": {
            "self": { "href": "/games" },
            "next": { "href": "/games?page=2" },
            "find": {"href": "/games{?query}", "templated": true}
        },
        "totalCount": 100,
        "_embedded": {
            "games": [
                {
                    "_links": {
                        "self": {"href": "/games/123"}
                    },
                    "price": 30.00,
                    "currency": "USD",
                    "name": "Halo 3"
                }
            ]
        }
    }
```

Note that there are templateable links that can be filled.


Collection+JSON - standard for reading and writing collections. Can include UI in messages. MIME type = application/vnd.collection+json. Profile Media Type = application/vnd.collection+json;profile=http://foo.com/bar. Examle

```
{
        "collection": {
            "version": "1.0",
            "href": "http://example.org/friends",
            "links": [
                { "rel": "feed", "href": "http://example.org/friends/rss"}
            ],
            "items": [
                {
                    "href": 'http://example.org/friends/jdoe',
                    "data": [
                        { "name": "full-name", "value", "J. Doe", "prompt": "Full Name"},
                        { "name": "email", "value": "jdoe@example.org", "prompt": "Email"}
                    ],
                    "links": [
                        { "rel": "blog", "href": "http://example.org/blogs/jdoe", "prompt": "Blog"}
                    ]
                }
            ]
        }
    }
```

You can also support query and template collections. The author finds Collection+JSON to be to verbose though for otherwise lean REST applications.

Be aware that Hypermedia is still a real new idea, and the thinking about it is still changing. Remember you have a job to do and you should deliver a solid app first, bells and whistles second.

 