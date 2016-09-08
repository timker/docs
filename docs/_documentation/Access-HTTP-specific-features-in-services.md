---
# 
---
ServiceStack is based on [http handlers](http://msdn.microsoft.com/en-us/library/system.web.ihttphandler.aspx), but ServiceStack provides a clean, dependency-free [IService<T>](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/ServiceHost/IService.cs) to implement your Web Services logic in. The philosophy behind this approach is that the less dependencies you have on your environment and its request context, the more testable and re-usable your services become. 

## Accessing the [IHttpRequest](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/ServiceHost/IHttpRequest.cs) and [IHttpResponse](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/ServiceHost/IHttpResponse.cs) in filters and Services

#### Request Filters
The Request Filters are applied before the service gets called and accepts: (IHttpRequest, IHttpResponse, RequestDto) e.g:

```csharp
//Add a request filter to check if the user has a session initialized
this.RequestFilters.Add((httpReq, httpResponse, requestDto) =>
{
    httpReq.Headers["HttpHeader"];
    httpReq.QueryString["queryParam"];
    httpReq.Form["htmlFormParam"];
    httpReq.GetParam("aParamInAnyOfTheAbove");

    httpReq.Cookies["requestCookie"];
    httpReq.AbsoluteUri;
    httpReq.Items["requestData"] = "Share data between Filters and Services";

     //Access underlying Request in ASP.NET hosts
    var aspNetRequest = httpResponse.OriginalRequest as HttpRequestBase;
     //Access underlying Request in HttpListener hosts
    var listenerRequest = httpResponse.OriginalRequest as HttpListenerRequest;
});
```

#### Services
When inheriting from Service you can access them via `base.Request` and `base.Response`:

```csharp
public class MyService : Service
{
    public object Any(Request request)
    {
        var value = base.Request.GetParam("aParamInAnyHeadersFormOrQueryString");
        base.Response.AddHeader("X-CustomHeader", "Modify HTTP Response in Service");
    }
}
```

#### Accessing the Request and Response from the RequestContext

The HttpRequest and HttpResponse is also accessible from a RequestContext with:

```csharp
var httpReq = requestContext.Get<IHttpRequest>();
var httpRes = requestContext.Get<IHttpResponse>();
```

#### Response Filters

The Response Filters are applied after your service is called and accepts: (IHttpRequest, IHttpResponse, ResponseDto) e.g Add a response filter to add a 'Content-Disposition' header so browsers treat it as a native .csv file:

```csharp
this.ResponseFilters.Add((req, res, responseDto) => 
{
    if (req.ResponseContentType == ContentType.Csv)
    {
        res.AddHeader(HttpHeaders.ContentDisposition,
            "attachment;filename={0}.csv".Fmt(req.OperationName));
    }

    //Access underlying Response in ASP.NET hosts
    var aspNetResponse = httpResponse.OriginalResponse as HttpResponseBase;
    //Access underlying Response in HttpListener hosts
    var listenerResponse = httpResponse.OriginalResponse as HttpListenerResponse;
});
```

### Communicating throughout the Request Pipeline

The recommended way to pass additional metadata about the request is to use the `IHttpRequest.Items` collection. E.g. you can change what Razor View template the response DTO gets rendered in with: 

```csharp
httpReq.Items["Template"] = "_CustomLayout";

...

var preferredLayout = httpReq.Items["Template"];
```

## Advantages for having dependency-free services

If you don't need to access the HTTP specific features your services can be called by any non-HTTP endpoint,  like from a [message queue](https://github.com/ServiceStack/ServiceStack/wiki/Messaging-and-Redis).

### Injecting the IRequestContext into your Service

Although working in a clean-room can be ideal from re-usability and testability point of view, you stand the chance of missing out a lot of the features present in HTTP.

Just like using built-in Funq IOC container, the way to tell ServiceStack to inject the request context is by implementing the [IRequiresRequestContext](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/ServiceHost/IRequiresRequestContext.cs) interface which will get the [IRequestContext](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/ServiceHost/IRequestContext.cs) injected before each request.

```csharp
public interface IRequestContext : IDisposable
{
    T Get<T>() where T : class;
    string IpAddress { get; }
    string GetHeader(string headerName);
    IDictionary<string, Cookie> Cookies { get; }
    EndpointAttributes EndpointAttributes { get; }
    IRequestAttributes RequestAttributes { get; }
    string ContentType { get; }
    string ResponseContentType { get; }
    string CompressionType { get; }
    string AbsoluteUri { get; }
    string PathInfo { get; }
    IFile[] Files { get; }
}
```

> **Note:** ServiceStack's `Service` base class already implements `IRequiresRequestContext` which allows you to access the `IRequestContext` with `base.RequestContext` and the HTTP Request and Response with `base.Request` and `base.Response`.

> **Note:** To return a customized HTTP Response, e.g. set Response Cookies or Headers, return the [HttpResult](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Common/Web/HttpResult.cs) object.