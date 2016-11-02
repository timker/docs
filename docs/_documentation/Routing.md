---
slug: routing
title: Routing
---

## Pre-defined Routes

Without any configuration required, ServiceStack already includes [pre-defined routes](/routing#pre-defined-routes) for all services in the format:

    /api?/[xml|json|html|jsv|csv]/[reply|oneway]/[servicename]

> servicename is the name of the Request DTO

e.g. the pre-defined url to call a JSON 'Hello' Service is:

    /json/reply/hello

#### [Auto Batched Requests](/auto-batched-requests)

    /json/reply/Hello[]

### SOAP Web Service urls

    /api?/[soap11|soap12]

## Custom Routes

In its most basic form, a Route is just any string literal attributed on your Request DTO:

```csharp
[Route("/hello")]
public class Hello { ... }
```

which matches: 

    /hello
    /hello?Name=XXX

### Variable place-holders

Routes can also have variable place-holders:

```csharp
[Route("/hello/{Name}")]
```

matches:

    /hello/foo

And will populate the public property **Name** on the Request DTO with **foo**.

*Note: The QueryString, FormData and HTTP Request Body isn't apart of the Route (i.e. only the /path/info is) but they can all be used in addition to every web service call to further populate the Request DTO.*

### Wildcard paths

Using a route with a wild card path like:

```csharp
[Route("/hello/{Name*}")]
```

matches any number of variable paths:

    /hello
    /hello/name
    /hello/my/name/is/ServiceStack    //Name = my/name/is/ServiceStack

[Another good use-case for when to use wildcard routes](http://stackoverflow.com/questions/7780157/multiple-optional-parameters-with-servicestack-net).

### Fallback Route

Use the `FallbackRoute` attribute to specify a fallback route starting from the root path, e.g:

```csharp
[FallbackRoute("/{Path}")]
public class Fallback
{
    public string Path { get; set; }
}
```

This will match any unmatched route from the root path (e.g. `/foo` but not `/foo/bar`) that's not handled by CatchAll Handler or matches a static file. You can also specify a wildcard path e.g. `[FallbackRoute("/{Path*}")]` which will handle every unmatched route (inc. `/foo/bar`). Only 1 fallback route is allowed.

The Fallback route is useful for HTML5 Single Page App websites handling server requests of HTML5 pushState pretty urls.

### Limiting to HTTP Verbs

If not specified Routes will match **All** HTTP Verbs. You can also limit Routes to individual Verbs, this lets you route the same path to different services, e.g:

```csharp
[Route("/contacts", "GET")]
[Route("/contacts/{Id}", "GET")]
public class GetContacts { ... }

[Route("/contacts", "POST PUT")]
[Route("/contacts/{Id}", "POST PUT")]
public class UpdateContact { ... }

[Route("/contacts/{Id}", "DELETE")]
public class DeleteContact { ... }
```

### Matching ignored paths

You can use the `{ignore}` variable placeholder to match a Route definition that doesn't map to a Request DTO property, e.g:

```csharp
[Route("/contacts/{Id}/{ignore}", "GET")]
public class GetContacts { ... }
```

Will match on `/contacts/1/john-doe` request and not require your Request DTO to have an **ignore** property

### Fluent API

You can also use a Fluent API to register ServiceStack Routes by adding them in your `AppHost.Configure()`:

```csharp
Routes
    .Add<Hello>("/hello")
    .Add<Hello>("/hello/{Name}");
```

and to match only **GET** request for `/Customers?Key=Value` and `/Customers/{Id}`:

```csharp
Routes
    .Add<GetContact>("/Contacts", "GET")
    .Add<GetContact>("/Contacts/{ContactId}", "GET");
```

## Content Negotiation

In addition to using the standard `Accept` HTTP Header to retrieve the response a different format, you can also request an alternative Content-Type by appending **?format=ext** to the query string, e.g:

  - [/rockstars?format=xml](http://razor.servicestack.net/rockstars?format=xml)
  - [/rockstars/1?format=json](http://razor.servicestack.net/rockstars/1?format=json)

Or by appending the format **.ext** to the end of the route, e.g:

  - [/rockstars.xml](http://razor.servicestack.net/rockstars.xml)
  - [/rockstars/1.json](http://razor.servicestack.net/rockstars/1.json)
  - [/rockstars.html?id=1](http://razor.servicestack.net/rockstars.html?id=1)

This is enabled on all custom routes and works for all built-in and user-registered formats. 
It can be disabled by setting `Config.AllowRouteContentTypeExtensions = false`.


## Auto Route Generation Strategies

Also related to this is registering Auto routes via the [Routes.AddFromAssembly](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack/ServiceRoutesExtensions.cs#L23) extension method, where this single call:

    Routes.AddFromAssembly(typeof(FooService).Assembly)

Goes through and scans all your services (in the Assemblies specified) and registers convention-based routes based on all the HTTP methods you have implemented. 

The default convention registers routes based on the Request DTO Name, whether it has an **Id** property and what actions were implemented. These conventions are configurable where you can now adjust/remove the existing rules or add your own to the **pre-defined** rules in `Config.RouteNamingConventions`:

```csharp
RouteNamingConventions = new List<RouteNamingConventionDelegate> {
    RouteNamingConvention.WithRequestDtoName,
    RouteNamingConvention.WithMatchingAttributes,     // defaults: PrimaryKeyAttrubute
    RouteNamingConvention.WithMatchingPropertyNames,  // defaults: Id, IDs
}
```

The existing rules can be further customized by modifying the related static properties, e.g:

```csharp
RouteNamingConvention.PropertyNamesToMatch.Add("UniqueId");
RouteNamingConvention.AttributeNamesToMatch.Add("DefaultIdAttribute");
```

Which will make these request DTOs:

```csharp
class MyRequest1
{
    public UniqueId { get; set;}
}

class MyRequest2
{
    [DefaultId]
    public CustomId { get; set;}
}
```

Generate the following routes:

    /myrequest1
    /myrequest1/{UniqueId}
    /myrequest2
    /myrequest2/{CustomId}

See the implementation of [RouteNamingConvention](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack/Host/RouteNamingConvention.cs) for examples of how to add your own auto-generated route conventions.

### Dynamically adding Route Attributes

Routes attributes can also be added dynamically but as Services are auto-registered before `AppHost.Configure()` runs, Route attributes need to be added before this happens like in the AppHost Constructor or before `new AppHost().Init()`, i.e:

```csharp
public class AppHost : AppHostBase {
    public AppHost() {
        typeof(MyRequest)
           .AddAttributes(new RouteAttribute("/myrequest"))
           .AddAttributes(new RouteAttribute("/myrequest/{UniqueId}"));
    }
}
```

### Customizing Defined Routes

You can customize existing routes by overriding `GetRouteAttributes()` in your AppHost, the example below adds a `/api` prefix to all existing routes:

```csharp
public class AppHost : AppHostBase
{
    //...
    public override RouteAttribute[] GetRouteAttributes(Type requestType)
    {
        var routes = base.GetRouteAttributes(requestType);
        routes.Each(x => x.Path = "/api" + x.Path);
        return routes;
    }
}
```

## Routing Resolution Order

This is described in more detail on the [New API Design wiki](/api-design) but the weighting used to select a route is based on:

  1. Any exact Literal Matches are used first
  2. Exact Verb match is preferred over All Verbs
  3. The more variables in your route the less weighting it has
  4. Routes with wildcard variables have the lowest precedence
  5. When Routes have the same weight, the order is determined by the position of the Action in the service or Order of Registration (FIFO)

### Route weighting example

The following HTTP Request:

    GET /content/v1/literal/slug

Will match the following Route definitions in order from highest precedence to lowest:

```csharp
[Route("/content/v1/literal/slug", "GET")]
[Route("/content/v1/literal/slug")]
[Route("/content/v1/literal/{ignore}", "GET")]
[Route("/content/{ignore}/literal/{ignore}", "GET")]
[Route("/content/{Version*}/literal/{Slug*}", "GET")]
[Route("/content/{Version*}/literal/{Slug*}")]
[Route("/content/{Slug*}", "GET")]
[Route("/content/{Slug*}")]
```

See the [RestPathTests.cs](https://github.com/ServiceStack/ServiceStack/blob/master/tests/ServiceStack.ServiceHost.Tests/RestPathTests.cs) and [Smart Routing](/api-design) section on the wiki for more examples.

### Reverse Routing

If you use `[Route]` metadata attributes (as opposed to the Fluent API) you will be able to generate strong-typed URI's using just the DTOs, letting you create urls outside of ServiceStack web framework as done with [.NET Service Clients](/csharp-client) using the `ToUrl(HttpMethod)` and `ToAbsoluteUri(HttpMethod)`, e.g:

```csharp
[Route("/reqstars/search", "GET")]
[Route("/reqstars/aged/{Age}")]
public class SearchReqstars : IReturn<ReqstarsResponse>
{
    public int? Age { get; set; }
}

var relativeUrl = new SearchReqstars { Age = 20 }.ToGetUrl();
var absoluteUrl = new SearchReqstars { Age = 20 }.ToAbsoluteUri();

relativeUrl.Print(); //=  /reqstars/aged/20
absoluteUrl.Print(); //=  http://www.myhost.com/reqstars/aged/20
```

The [Email Contacts demo](https://github.com/ServiceStack/EmailContacts/) shows an example of using the above Reverse Routing extension methods to [populate routes for HTML Forms and Links in Razor Views](https://github.com/ServiceStack/EmailContacts/#bootstrap-forms). 

#### Other Reverse Routing Extension methods

```csharp
new RequestDto().ToPostUrl();
new RequestDto().ToPutUrl();
new RequestDto().ToDeleteUrl();
new RequestDto().ToOneWayUrl();
new RequestDto().ToReplyUrl();
```

### Customize urls used with `IUrlFilter`

Request DTO's can customize urls used in Service Clients or any libraries using ServiceStack's typed 
[Reverse Routing](/routing#reverse-routing) by having 
Request DTO's implement 
[IUrlFilter](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/IUrlFilter.cs).

ServiceStack's [Stripe Gateway](https://github.com/ServiceStack/Stripe) takes advantage of ServiceStack's
typed Routing feature to implement its 
[Open-Ended, Declarative Message-based APIs](https://github.com/ServiceStack/Stripe#open-ended-declarative-message-based-apis)
with minimal effort.

In order to match Stripe's unconventional syntax for specifying arrays on the QueryString of their 3rd party
REST API we use `IUrlFilter` to customize the url that's used. E.g. we need to specify `include[]` in order
for the Stripe API to return any optional fields like **total_count**.

```csharp
[Route("/customers")]
public class GetStripeCustomers : IGet, IReturn<StripeCollection<Customer>>, IUrlFilter
{
    public GetStripeCustomers() 
    {
        Include = new[] { "total_count" };
    }

    [IgnoreDataMember]
    public string[] Include { get; set; }

    public string ToUrl(string absoluteUrl)
    {
        return Include == null ? absoluteUrl 
            : absoluteUrl.AddQueryParam("include[]", string.Join(",", Include));
    }
}
```

> `[IgnoreDataMember]` is used to hide the property being emitted using the default convention

Which when sending the Request DTO:

```csharp
var response = client.Get(new GetStripeCustomers());
```

Generates and sends the relative url:

    /customers?include[]=total_count

Which has the effect of populating the `TotalCount` property in the typed `StripeCollection<StripeCustomer>` response.

## Routing Metadata

Most of the metadata ServiceStack knows about your services are accessible internally via `HostContext.Config.Metadata` from within ServiceStack and externally via the `/operations/metadata` route. A link to the **Operations Metadata** page is displayed at the bottom of the /metadata when in ServiceStack is in `DebugMode`.

### Great Performance

Since [Routing in ASP.NET MVC can be slow](http://samsaffron.com/archive/2011/10/13/optimising-asp-net-mvc3-routing) when you have a large number of Routes, it's worthwhile pointing out ServiceStack's Routing implementation is implemented with hash lookups so doesn't suffer the linear performance regression issues you might have had with MVC. So you don't have to worry about degraded performance when registering a large number of Routes.

# Community Resources

  - [Use routes to customize service endpoints](http://dilanperera.wordpress.com/2014/02/23/servicestack-use-routes-to-customise-service-endpoints/)
    - [More on Routes](http://dilanperera.wordpress.com/2014/02/24/servicestack-more-on-routes/)
  - [Using PostSharp to Add ServiceStack Route Attributes](http://jokecamp.wordpress.com/2013/06/11/using-postsharp-to-add-servicestack-route-attributes/) by [@jokecamp](https://twitter.com/jokecamp)