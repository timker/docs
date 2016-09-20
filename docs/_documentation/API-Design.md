---
slug: api-design
title: ServiceStack’s API design
---
We're excited to announce a ServiceStack New API design that's essentially better in every way than our original legacy v3 API design:

  - Promotes a more succinct, typed, end-to-end client API 
  - Works with all the existing JSON, XML and JSV Service Clients
  - Typed client APIs use user-defined [Cool Uris](http://www.w3.org/TR/cooluris/) without needing to build strings on the HTTP Client
  - Supports handling of any HTTP Verb 
  - Less Restrictive and Opinionated - Allows for 'Pure Responses' whilst retaining structured exceptions in the typed clients
  - Each service can now handle any number of different Request types and HTTP Verbs
  - Easier to use - Merges and simplifies existing IService and IRestService concepts and interfaces together
  - Introduces finer-grained Action Request and Response filters
  - Smarter Routing
  - Easier to add custom hooks that's more decoupled and testable
  - Works with ServiceStack's existing features, e.g. Content Negotiation, Metadata pages, Razor views, Auto HTML report, etc.

All this and the existing API wasn't actually bad :) It still promoted one of the DRY-est, typed APIs available in any .NET existing Web Services Framework, whilst continue to provide the most features out-of-the-box.

To give you a flavour of the new API design, you can now create a service with:

```csharp
[Route("/reqstars")]
public class AllReqstars : IReturn<List<Reqstar>> { }

public class ReqstarsService : Service
{
    public object Get(AllReqstars request) 
    {
        return Db.Select<Reqstar>();
    }
}
```

That your C# clients can call with just:

```csharp
List<Reqstar> response = client.Get(new AllReqstars());
```

This will make a **GET** call to the custom `/reqstars` url, making it the **minimum effort required in any Typed REST API in .NET!** 
When the client doesn't contain the `[Route]` definition it automatically falls back to using ServiceStack's [pre-defined routes](http://www.servicestack.net/ServiceStack.Hello/#predefinedroutes) - saving an extra LOC!

### Inspiration

We were heavily inspired by [Ivan Korneliuk's proposal](http://korneliuk.blogspot.com/2012/08/servicestack-reusing-dtos.html) on how he customised ServiceStack to provide an even more succinct client API. We've embraced his idea and baked it into the heart of the framework which is now shipping in the latest release of ServiceStack (v3.9.13+). The beauty of this proposal was that it already fitted perfectly with ServiceStack's existing message-based semantics which meant we were able to implement it in record time without any disruption or breaking changes to the existing code-base. The result is that you can now start creating new services along side your existing services and they'll both continue to work seamlessly side-by-side.

### Recommended for future Web Service Development

As the new API Design offers many benefits over the existing API, we're recommending its use for any new web service development. It will take us some time, but we intend to port all the old examples to adopt the new API ourselves. One reason to still prefer the older API is if you also wanted to [support SOAP clients and endpoints](?id=soap-support) which still requires the strict-ness enforced by the previous approach.

## ServiceStack's New API Design

We'll walk through a few examples here but for a more detailed look into the usages and capabilities of the new API design checkout its
[Comprehensive Test Suite](https://github.com/ServiceStack/ServiceStack/blob/master/tests/RazorRockstars.Console.Files/ReqStarsService.cs)

The new API design simplifies the existing Services with the single unified interface:

```csharp
public interface IService {}
```

That is now able to handle both RPC Service and Rest Service requests in a single class.
The interface is just used as a Marker interface that ServiceStack uses to find, register and auto-wire your existing services. A convenience concrete `Service` class is also included which contains easy access to ServiceStack's providers:

```csharp
public class Service : IService {
    IRequest Request { get; }                 //HTTP Request Wrapper
    IResponse Response { get; }               //HTTP Response Wrapper
    IServiceGateway Gateway { get; }          //Built-in Service Gateway
    IVirtualPathProvider VirtualFileSources   //Virtual FileSystem Sources
    IVirtualFiles VirtualFiles { get; }       //Writable Virtual FileSystem
    ICacheClient Cache { get; }               //Registered Caching Provider
    MemoryCacheClient LocalCache { get; }     //Local InMemory Caching Provider
    IDbConnection Db { get; }                 //Registered ADO.NET IDbConnection
    IRedisClient Redis { get; }               //Registered RedisClient 
    IMessageProducer MessageProducer { get; } //Message Producer for Registered MQ Server
    ISession SessionBag { get; }              //Dynamic Session Bag
    TUserSession SessionAs<TUserSession>();   //Resolve Typed UserSession
    T TryResolve<T>();                        //Resolve dependency at runtime
    T ResolveService<T>();                    //Resolve an auto-wired service
    void PublishMessage(T message);           //Publish messages to Registered MQ Server
    bool IsAuthenticated { get; }             //Is Authenticated Request
    void Dispose();                           //Override to implement custom Dispose
}
```

### Basic example - Handling Any HTTP Verb

Lets revisit the Simple example from earlier:

```csharp
[Route("/reqstars")]
public class AllReqstars : IReturn<List<Reqstar>> { }

public class ReqstarsService : Service
{
    public object Any(AllReqstars request) 
    {
        return Db.Select<Reqstar>();
    }
}
```

The new API maps HTTP Requests to your Services **Actions**. An Action is any method that:

  - Is public 
  - Only contains a single argument - the typed Request DTO
  - Has a Method name matching a HTTP Method or **Any** which is used as a fallback if it exists
  - Can specify either `T` or `object` Return type, both have same behavior 

The above example will handle any `AllReqstars` request made on any **HTTP Verb** or **endpoint** and will return the complete `List<Reqstar>` contained in your configured RDBMS. 

### Micro ORMs and ADO.NET's IDbConnection

Code-First Micro ORMS like [OrmLite](https://github.com/ServiceStack/ServiceStack.OrmLite) and 
[Dapper](http://code.google.com/p/dapper-dot-net/) provides a pleasant high-level experience whilst working directly against ADO.NET's low-level `IDbConnection`. They both support all major databases so you immediately have access to a flexible RDBMS option out-of-the-box. At the same time you're not limited to using the providers contained in the `Service` class and can continue to use your own register IOC dependencies (inc. an alternate IOC itself). 

### Micro ORM POCOs make good DTOs

The POCOs used in Micro ORMS are particularly well suited for re-using as DTOs since they don't contain any circular references that the Heavy ORMs have (e.g. EF). OrmLite goes 1-step further and borrows pages from NoSQL's playbook where any complex property e.g. `List<MyPoco>` is transparently blobbed in a schema-less text field, promoting the design of frictionless **Pure POCOS** that are uninhibited by RDBMS concerns. In many cases these POCO data models already make good DTOs and can be returned directly instead of mapping to domain-specific DTOs.

### Calling Services from a Typed C# Client

In Service development your services DTOs provides your technology agnostic **Service Layer** which you want to keep clean and as 'dependency-free' as possible for maximum accessibility and potential re-use. Our recommendation is to keep your service DTOs in a separate largely dep-free assembly. We intend to improve this story further with a commercial VS.NET extension to enable 'Add ServiceStack Reference' like behaviour providing a familiar productive development experience for existing VS.NET SOAP WebService developers.

One of ServiceStack's strengths is its ability to re-use your Server DTOs on the client enabling ServiceStack's productive end-to-end typed API. The exact DTOs aren't needed, only the shape of the DTOs is important, ServiceStack's message-based design means you can even use partial DTOs on the client containing just the fields they're interested in. 

But lets say you take the normal route of copying the DTOs (in either source of binary form) so you have something like this on the client:

```csharp
[Route("/reqstars")]
public class AllReqstars : IReturn<List<Reqstar>> { }
```

The code on the client now just becomes:

```csharp
var client = new JsonServiceClient(BaseUri);
List<Reqstar> response = client.Get(new AllReqstars());
```

Which makes a **GET** web request to the `/reqstars` route.
When a custom route is not present on the client it automatically falls back to using ServiceStack's [pre-defined routes](http://mono.servicestack.net/ServiceStack.Hello/#predefinedroutes).
	
Finally you can also use the previous more explicit client API (ideal for when you don't have the `IReturn<>` marker):

```csharp
var response = client.Get<List<Reqstar>>("/reqstars");
```

All these APIs **have async equivalents** which you can use instead, when you need to.

## Everything Just Works

The immediate benefit we were able to realise from designing the new API within ServiceStack's existing semantics was that everything else just works. You're able to re-use the same Routes, Filters, Views and Validators together and it will continue to work just as it did before. 

E.g. you can take advantage of [ServiceStack's recent Razor support](http://razor.servicestack.net/) and create a web page for this service by just adding a [AllReqstars.cshtml](https://github.com/ServiceStack/ServiceStack/blob/master/tests/RazorRockstars.Console.Files/Views/AllReqstars.cshtml) in your views folder. Thanks to the built-in Content Negotiation you can fetch the HTML contents calling the same url: 

```csharp
var html = "{0}/reqstars".Fmt(BaseUri).GetStringFromUrl(acceptContentType:"text/html");
```

This [feature is particularly nice](http://razor.servicestack.net/#unified-stack) as it lets you **re-use your existing services** to serve both Web and Native Mobile and Desktop clients.

### Action Filters

Service actions can also contain fine-grained application of Request and Response filters, e.g:

```csharp
public class ReqstarsService : Service
{
    [ClientCanSwapTemplates]
    public List<Reqstar> Any(AllReqstars request) 
    {
        return Db.Select<Reqstar>();
    }
}
```

This Request Filter allows the client to [change the selected Razor **View** and **Template**](http://razor.servicestack.net/#unified-stack) used at runtime. By default the view with the same name as the **Request** or **Response** DTO is used.

## Handling different HTTP Verbs

The new API design now lets you handle any HTTP Verb. This lets you respond with CORS headers to a HTTP **OPTIONS** request with just:

```csharp
public class ReqstarsService : Service
{
    [EnableCors]
    public void Options(Reqstar request) {}
}
```

Which if you now make an OPTIONS request to the above service, will emit the default `[EnableCors]` headers:

```csharp
var webReq = (HttpWebRequest)WebRequest.Create(Host + "/reqstars");
webReq.Method = "OPTIONS";
using (var webRes = webReq.GetResponse())
{
    webRes.Headers["Access-Control-Allow-Origin"]     // *
    webRes.Headers["Access-Control-Allow-Methods"]    // GET, POST, PUT, DELETE, OPTIONS
    webRes.Headers["Access-Control-Allow-Headers"]    // Content-Type
}
```

### PATCH request example

Handling a PATCH request is just as easy, e.g. here's an example of using PATCH to handle a partial update of a Resource:

```csharp
[Route("/reqstars/{Id}", "PATCH")]
public class UpdateReqstar : IReturn<Reqstar>
{
    public int Id { get; set; }
    public int Age { get; set; }
}

public Reqstar Patch(UpdateReqstar request)
{
    Db.Update<Reqstar>(request, x => x.Id == request.Id);
    return Db.Id<Reqstar>(request.Id);
}
```

And the client call is just as easy as you would expect:

```csharp
var response = client.Patch(new UpdateReqstar { Id = 1, Age = 18 });
```

Although sending different HTTP Verbs are unrestricted in native clients, they're unfortunately not allowed in some web browsers and proxies. So to simulate a PATCH from an AJAX request you need to set the **X-Http-Method-Override** HTTP Header.

## Structured Error Handling

In the previous API we had a restriction that if you wanted structured exceptions on the client you need to have a Response DTO with the same name as your Request DTO but with a 'Response' suffix. This restriction has now been lifted and we will now just use a generic `ErrorResponse` when a Response DTO can't be found. This is a transparent technical detail you don't need to know about.

So Error Handling is effectively the same as it was before, but now can work without needing a Response DTO, e.g:

```csharp
public List<Reqstar> Post(Reqstar request)
{
    if (!request.Age.HasValue)
        throw new ArgumentException("Age is required");

    Db.Insert(request.TranslateTo<Reqstar>());
    return Db.Select<Reqstar>();
}
```

Which will throw this Error if the client tried to create an empty Reqstar:

```csharp
try
{
    var response = client.Post(new Reqstar());
}
catch (WebServiceException webEx)
{
    webEx.StatusCode                    // 400
    webEx.StatusDescription             // ArgumentException
    webEx.ResponseStatus.ErrorCode      // ArgumentException
    webEx.ResponseStatus.Message        // Age is required
    webEx.ResponseDto is ErrorResponse  // true
}
```

When your Service does have a conventionally named Response DTO, thrown exceptions will continue to be injected into an instance of that as seen in the [invalid SearchReqstars request example](https://github.com/ServiceStack/ServiceStack/blob/master/tests/RazorRockstars.Console.Files/ReqStarsService.cs#L310).

You can use the Service Clients Exception handling for handling any HTTP error generated in or outside of your service, e.g. here's how to detect if a HTTP Method isn't implemented or disallowed:

```csharp
try
{
    var response = client.Send(new SearchReqstars());
}
catch (WebServiceException webEx)
{
    webEx.StatusCode                   // 405
    webEx.StatusDescription            // Method Not Allowed
}
```

In addition to standard C# exceptions your services can also return multiple, rich and detailed validation errors as enforced by [Fluent Validation's validators](?id=validation).

### Overriding the default Exception handling

Overriding the default exception handling in ServiceStack just got a lot easier as well, you now no longer need to provide your own base class to do this and can easily just override it in your AppHost with:

```csharp
void Configure(Container container) 
{
    this.ServiceExceptionHandlers.Add((req, reqDto, ex) => {
        return ...;
    });
}
```

## Smart Routing

For the most part you won't need to know about this as ServiceStack's routing works as you would expect. Although this should still serve as a good reference to describe the resolution order of ServiceStack's Routes:

  1. Any exact Literal Matches are used first
  2. Exact Verb match is preferred over All Verbs
  3. The more variables in your route the less weighting it has
  4. When Routes have the same weight, the order is determined by the position of the Action in the service or Order of Registration (FIFO)

These Rules only come into play when there are multiple routes that matches the pathInfo of an incoming request.

Lets see some examples of these rules in action using the routes defined in the [new API Design test suite](https://github.com/ServiceStack/ServiceStack/blob/master/tests/RazorRockstars.Console.Files/ReqStarsService.cs):

```csharp
[Route("/reqstars")]
public class Reqstar {}

[Route("/reqstars", "GET")]
public class AllReqstars {}

[Route("/reqstars/{Id}", "GET")]
public class GetReqstar {}

[Route("/reqstars/{Id}/{Field}")]
public class ViewReqstar {}

[Route("/reqstars/{Id}/delete")]
public class DeleteReqstar {}

[Route("/reqstars/{Id}", "PATCH")]
public class UpdateReqstar {}

[Route("/reqstars/reset")]
public class ResetReqstar {}

[Route("/reqstars/search")]
[Route("/reqstars/aged/{Age}")]
public class SearchReqstars {}
```

These are results for these HTTP Requests

	GET   /reqstars           =>	AllReqstars
	POST  /reqstars           =>	Reqstar
	GET   /reqstars/search    =>	SearchReqstars
	GET   /reqstars/reset     =>	ResetReqstar
	PATCH /reqstars/reset     =>	ResetReqstar
	PATCH /reqstars/1         =>	UpdateReqstar
	GET   /reqstars/1         =>	GetReqstar
	GET   /reqstars/1/delete  =>	DeleteReqstar
	GET   /reqstars/1/foo     =>	ViewReqstar

And if there were multiple of the exact same routes declared like:

```csharp
[Route("/req/{Id}", "GET")]
public class Req2 {}

[Route("/req/{Id}", "GET")]
public class Req1 {}

public class MyService : Service {
    public object Get(Req1 request) { ... }		
    public object Get(Req2 request) { ... }		
}
```

The Route on the Action that was declared first gets selected, i.e:

    GET /req/1              => Req1

## Advanced Usages

### Custom Hooks

The ability to extend ServiceStack's service execution pipeline with Custom Hooks is an advanced customisation feature that for most times is not needed as the preferred way to add composable functionality to your services is to use [Request / Response Filter attributes](?id=Filter-attributes) or apply them globally with [Global Request/Response Filters](?id=Request-and-response-filters).

Although this is another area we've improved on as you can now add your own custom hooks without needing to subclass any services. To do this we've introduced the concept of a [IServiceRunner](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/ServiceHost/IServiceRunner.cs) that decouples the execution of your service from the implementation of it.

To add your own Service Hooks you just need to override the default Service Runner in your AppHost from its default implementation:

```csharp
public virtual IServiceRunner<TRequest> CreateServiceRunner<TRequest>(ActionContext actionContext)
{           
    return new ServiceRunner<TRequest>(this, actionContext); //Cached per Service Action
}
```

With your own:

```csharp
public override IServiceRunner<TRequest> CreateServiceRunner<TRequest>(ActionContext actionContext)
{           
    return new MyServiceRunner<TRequest>(this, actionContext); //Cached per Service Action
}
```

Where `MyServiceRunner<T>` is just a custom class implementing the custom hooks you're interested in, e.g:

```csharp
public class MyServiceRunner<T> : ServiceRunner<T> {
    public override void OnBeforeExecute(IRequestContext requestContext, TRequest request) {
      // Called just before any Action is executed
    }

    public override object OnAfterExecute(IRequestContext requestContext, object response) {
      // Called just after any Action is executed, you can modify the response returned here as well
    }

    public override object HandleException(IRequestContext requestContext, TRequest request, Exception ex) {
      // Called whenever an exception is thrown in your Services Action
    }
}
```

## Limitations

This should probably be spelled out (even though wasn't possible with the previous API) but as the new API imposes less restriction we'll note it here: You're still not able to split the handling of a single Resource (i.e. Request DTO) over multiple service implementations. If you find you need to do this because your service is getting too big, consider using partial classes to spread the implementation over multiple files. Another option is encapsulating some of the re-usable functionality into Logic dependencies and inject them into your service.

## Other Notes

Although they're not needed or used anywhere [we've introduced new HTTP service interfaces](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/ServiceHost/IService.cs#L18) that enforce the correct signature required by the services, e.g:

```csharp
public class MyService : Service, IAny<AllReqstars>, IGet<SearchReqstars>, IPost<Reqstar>
{
    public object Any(AllReqstars request) { .. }
    public object Get(SearchReqstars request) { .. }
    public object Post(Reqstar request) { .. }
}
```

This has no effect to the runtime behaviour and your services will work the same way with or without the added interfaces.

## Refactoring existing services to use the new API Design

For the most part it should be fairly straight forward to port an existing service to the new API. To ease the process I'll walk through the changes we made to our most recent [razor.servicestack.net](http://razor.servicestack.net) demo. The goal of this port is to retain exactly the same API and behaviour so all existing urls continue to work just as they did before. 

The service that runs at the heart of the Razor Rockstars demo is the [/rockstars](http://razor.servicestack.net/rockstars) uber-service which originally with old API that looked like:

```csharp
[Route("/rockstars")]
[Route("/rockstars/aged/{Age}")]
[Route("/rockstars/delete/{Delete}")]
[Route("/rockstars/{Id}")]                      //All Routes on the single DTO
public class Rockstars                          //All Fields merged into a single DTO
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public int? Age { get; set; }
    public bool Alive { get; set; }
    public string Delete { get; set; }
}

[Csv(CsvBehavior.FirstEnumerable)]
public class RockstarsResponse
{
    public int Total { get; set; }
    public int? Aged { get; set; }
    public List<Rockstar> Results { get; set; }
}

[ClientCanSwapTemplates]
public class RockstarsService : RestServiceBase<Rockstars>
{
    //Base class didn't include a IDbConnectionFactory
    public IDbConnectionFactory DbFactory { get; set; }       

    //Single implementation handling all GET requests
    public override object OnGet(Rockstars request)           
    {
        using (var db = DbFactory.Open())
        {
            if (request.Delete == "reset")
            {
                db.DeleteAll<Rockstar>();
                db.InsertAll(Rockstar.SeedData);
            }
            else if (request.Delete.IsInt())
            {
                db.DeleteById<Rockstar>(request.Delete.ToInt());
            }

            return new RockstarsResponse {
                Aged = request.Age,
                Total = db.Scalar<int>("select count(*) from Rockstar"),
                Results = request.Id != default(int) ?
                    db.Select<Rockstar>(q => q.Id == request.Id)
                      : request.Age.HasValue ?
                    db.Select<Rockstar>(q => q.Age == request.Age.Value)
                      : db.Select<Rockstar>()
            };
        }
    }

    public override object OnPost(Rockstars request)
    {
        using (var db = DbFactory.Open())
        {
            db.Insert(request.TranslateTo<Rockstar>());
            return OnGet(new Rockstars());
        }
    }
}
```

I've added comments to all the attributes where the implementations differ with the new service.
The biggest change is seen with the flexibility of the new API where it now allows a single service to handle multiple Request DTOs and lets you provide different implementations for each. This allows us to split existing operations in more cohesive parts and instead of a single merged Request DTO, we have more fine-grained DTOs with just the fields required for each operation:

```csharp
[Route("/rockstars")]
[Route("/rockstars/aged/{Age}")]
public class Rockstars               //Include only fields used in the GET/Search action
{
    public int? Age { get; set; }
    public int Id { get; set; }
}

[Route("/rockstars/delete/{Id}")]
public class DeleteRockstar         //Specific Action to delete a Rockstar. Only Id field needed
{
    public int Id { get; set; }
}

[Route("/rockstars/delete/reset")]  //The route for this can now be /rockstars/reset
public class ResetRockstars { }     //No fields required for this Action

[Csv(CsvBehavior.FirstEnumerable)]
public class RockstarsResponse
{
    public int Total { get; set; }
    public int? Aged { get; set; }
    public List<Rockstar> Results { get; set; }
}

[ClientCanSwapTemplates]
[DefaultView("Rockstars")]                //Default View for each Action
public class RockstarsService : Service
{
    public object Get(Rockstars request)  //Only concerned with GET/Search actions
    {
        return new RockstarsResponse {
            Aged = request.Age,
            Total = Db.Scalar<int>("select count(*) from Rockstar"),
            Results = request.Id != default(int) 
                ? Db.Select<Rockstar>(q => q.Id == request.Id)
                : request.Age.HasValue 
                    ? Db.Select<Rockstar>(q => q.Age == request.Age.Value)
                    : Db.Select<Rockstar>()
        };
    }

    public object Any(DeleteRockstar request) //Handles any HTTP Verb
    {
        Db.DeleteById<Rockstar>(request.Id);
        return Get(new Rockstars());
    }

    public object Post(Rockstar request)
    {
        Db.Insert(request);
        return Get(new Rockstars());
    }

    public object Any(ResetRockstars request) //Handles any HTTP Verb
    {
        Db.DeleteAll<Rockstar>();
        Db.InsertAll(AppHost.SeedData);
        return Get(new Rockstars());
    }
}
```

The major omission that's no longer required in the Port is the using statement around DB access since it's now available by default in the `Service` base class. If you have your own dependencies that a lot of your services use, it's a good idea to include them in your own custom base class as it reduces the amount of boilerplate needed.

The new introduction in this port is the `[DefaultView("Rockstars")]` Request Filter Attribute. This is required because we want the **text/html** format to use the **Rockstars.cshtml** Razor view to render the HTML page. In the old version when we only had 1 Request DTO named **Rockstars** this was able to inferred by the Razor View Engine. This is no longer the case now that we have multiple actions with different Request DTO's. Although if the Razor View was instead called **RockstarsResponse.cshtml** we wouldn't have needed the attribute since in all cases a populated **RockstarsResponse** DTO is returned. As we want this to be an exact port, rather than changing the name of the View we decided to specify the default view to be used in each action instead.

#### Filter Attributes can be Applied to Actions as well
Other interesting points in this port is now that the new API allows Action Filter attributes we could've instead placed the Attribute on each action, instead of on the service where it applies to all Actions. Also it's worth pointing out that the `[DefaultView]` is a Request Filter Attribute so is executed before the Action therefore doesn't clobber the values set by `[ClientCanSwapTemplates]` since it's a Response Filter which is executed after the Action.

### Other New API examples

Examples of more converted services are in the New API [[Release Notes]].