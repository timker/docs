---
slug: customize-http-responses
title: Customize HTTP Responses
---
ServiceStack provides multiple ways to customize your services HTTP response. Each option gives you complete control of the final HTTP Response that's returned by your service: 

  1. Decorating it inside a `HttpResult` object
  2. Throwing a `HttpError` 
  3. Returning a `HttpError`
  4. Using a Request or Response [Filter Attribute](https://github.com/ServiceStack/ServiceStack/wiki/Filter-attributes) like the built-in `[AddHeader]` (or your own) or using a [Global Request or Response Filter](https://github.com/ServiceStack/ServiceStack/wiki/Request-and-response-filters).
  5. Modifying output by accessing your services `base.Response` [IHttpResponse API](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/Web/IHttpResponse.cs)

Here are some code examples below using these different approaches:

```csharp
public class HelloService : Service
{ 
    public object Get(Hello request) 
    { 
        //1. Returning a custom Status and Description with Response DTO body:
        var responseDto = ...;
        return new HttpResult(responseDto, HttpStatusCode.Conflict) {
            StatusDescription = "Computer says no",
        };

        //2. Throw a HttpError:
        throw new HttpError(HttpStatusCode.Conflict, "Some Error Message");

        //3. Return a HttpError:
        return new HttpError(HttpStatusCode.Conflict, "Some Error Message");

        //4. Modify the Request's IHttpResponse
        base.Response.StatusCode = (int)HttpStatusCode.Redirect;
        base.Response.AddHeader("Location", "http://path/to/new/uri");
        base.Response.EndRequest(); //Short-circuits Request Pipeline
    }

    //5. Using a Request or Response Filter 
    [AddHeader(ContentType = "text/plain")]
    public string Get(Hello request)
    {
        return "Hello, {0}!".Fmt(request.Name);
    }
}
```

### Short-circuiting the Request Pipeline

At anytime you can short-circuit the Request Pipeline and end the response by calling `IResponse.EndRequest()`, e.g:

```csharp
res.EndRequest();
```

If you only have access to the `IRequest` you can access the `IResponse` via the Response property, e.g:

```csharp
req.Response.EndRequest();
```

### Using a [Request or Response Filter Attribute](https://github.com/ServiceStack/ServiceStack/wiki/Filter-attributes)

Example 4). uses the in-built [AddHeaderAttribute](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack/AddHeaderAttribute.cs) to modify the HTTP Response using a [Request Filter attribute](https://github.com/ServiceStack/ServiceStack/wiki/Filter-attributes). You can also modify all HTTP Service Responses by using a [Global Request or Response Filter](https://github.com/ServiceStack/ServiceStack/wiki/Request-and-response-filters): 

```csharp
public class AddHeaderAttribute : RequestFilterAttribute
{
    public string Name { get; set; }
    public string Value { get; set; }
    
    public AddHeaderAttribute() { }

    public AddHeaderAttribute(string name, string value)
    {
        Name = name;
        Value = value;
    }

    public override void Execute(IRequest req, IResponse res, object requestDto)
    {
        if (string.IsNullOrEmpty(Name) || string.IsNullOrEmpty(Value)) 
            return;

        if (Name.EqualsIgnoreCase(HttpHeaders.ContentType))
        {
            res.ContentType = Value;
        }
        else
        {
            res.AddHeader(Name, Value);
        }
    }
...
}
```

## Custom Serialized Responses

The new `IHttpResult.ResultScope` API provides an opportunity to execute serialization within a custom scope, e.g. this can
be used to customize the serialized response of adhoc services that's different from the default global configuration with:

```csharp
return new HttpResult(dto) {
    ResultScope = () => JsConfig.With(includeNullValues:true)
};
```

Which enables custom serialization behavior by performing the serialization within the custom scope, equivalent to:

```csharp
using (JsConfig.With(includeNullValues:true))
{
    var customSerializedResponse = Serialize(dto);
}
```

## Request and Response Converters

The [Encrypted Messaging Feature](https://github.com/ServiceStack/ServiceStack/wiki/Encrypted-Messaging) takes advantage of Request and Response Converters that let you change the Request DTO and Response DTO's that get used in ServiceStack's Request Pipeline where:

### Request Converters

Request Converters are executed directly after any [Custom Request Binders](https://github.com/ServiceStack/ServiceStack/wiki/Serialization-deserialization#create-a-custom-request-dto-binder):

```csharp
appHost.RequestConverters.Add((req, requestDto) => {
    //Return alternative Request DTO or null to retain existing DTO
});
```

### Response Converters

Response Converters are executed directly after the Service:

```csharp
appHost.ResponseConverters.Add((req, response) =>
    //Return alternative Response or null to retain existing Service response
});
```

### Using a Custom ServiceRunner

The ability to extend ServiceStack's service execution pipeline with Custom Hooks is an advanced customization feature that for most times is not needed as the preferred way to add composable functionality to your services is to use [Request / Response Filter attributes](https://github.com/ServiceStack/ServiceStack/wiki/Filter-attributes) or apply them globally with [Global Request/Response Filters](https://github.com/ServiceStack/ServiceStack/wiki/Request-and-response-filters).

To be able to add custom hooks without needing to subclass any service, we've introduced a [IServiceRunner](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/Web/IServiceRunner.cs) that decouples the execution of your service from the implementation of it.

To add your own Service Hooks you just need to override the default Service Runner in your AppHost from its default implementation:

```csharp
public virtual IServiceRunner<TRequest> CreateServiceRunner<TRequest>(
    ActionContext actionContext)
{
    //Cached per Service Action
    return new ServiceRunner<TRequest>(this, actionContext); 
}
```

With your own:

```csharp
public override IServiceRunner<TRequest> CreateServiceRunner<TRequest>(
    ActionContext actionContext)
{           
    //Cached per Service Action
    return new MyServiceRunner<TRequest>(this, actionContext); 
}
```

Where `MyServiceRunner<T>` is just a custom class implementing the custom hooks you're interested in, e.g:

```csharp
public class MyServiceRunner<T> : ServiceRunner<T> 
{
    public override void OnBeforeExecute(
        IRequest requestContext, TRequest request) 
    {
      // Called just before any Action is executed
    }

    public override object OnAfterExecute(
        IRequest requestContext, object response) 
    {
      // Called just after any Action is executed.
      // You can modify the response returned here as well
    }

    public override object HandleException(
        IRequest requestContext, TRequest request, Exception ex) 
    {
      // Called whenever an exception is thrown in your Services Action
    }
}
```
