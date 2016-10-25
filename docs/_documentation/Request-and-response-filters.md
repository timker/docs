---
slug: request-and-response-filters
title: Request & Response filters
---

A recent addition to ServiceStack is the ability to register custom Request and Response filters. These should be registered in your `AppHost.Configure()` onload script: 

### Global Request Filters

The Request Filters are applied before the service gets called and accepts:
([IRequest](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/Web/IRequest.cs), [IResponse](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/Web/IResponse.cs), RequestDto) e.g:
    
```csharp
//Add a request filter to check if the user has a session initialized
this.GlobalRequestFilters.Add((req, res, requestDto) => {
    var sessionId = req.GetCookieValue("user-session");
    if (sessionId == null)
    {
        res.ReturnAuthRequired();
    }
});
```

### Global Response Filters

The Response Filters are applied after your service is called and accepts:
([IRequest](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/Web/IRequest.cs), [IResponse](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/Web/IResponse.cs), ResponseDto) e.g:

```csharp
//Add a response filter to add a 'Content-Disposition' header so browsers treat it as a native .csv file
this.GlobalResponseFilters.Add((req, res, responseDto) => {
    if (req.ResponseContentType == ContentType.Csv)
    {
        res.AddHeader(HttpHeaders.ContentDisposition,
        string.Format("attachment;filename={0}.csv", req.OperationName));
    }
});
```

Tip: If you're writing your own response to the response stream inside the response filter, add `res.EndRequest();` to signal to ServiceStack not to do anymore processing for this request.

### Typed Request Filters

A more typed API to register Global Request and Response filters per Request DTO Type are available under the `RegisterTyped*` API's in AppHost where you can register both typed Request and Response Filters for HTTP and MQ Services independently:

```csharp
void RegisterTypedRequestFilter<T>(Action<IRequest, IResponse, T> filterFn);
void RegisterTypedResponseFilter<T>(Action<IRequest, IResponse, T> filterFn);
void RegisterTypedMessageRequestFilter<T>(Action<IRequest, IResponse, T> filterFn);
void RegisterTypedMessageResponseFilter<T>(Action<IRequest, IResponse, T> filterFn);
```

Here's an example usage that enables more flexibility in multi-tenant solutions by attaching custom data on incoming requests, e.g:

```csharp
public override void Configure(Container container)
{
    RegisterTypedRequestFilter<Resource>((req, res, dto) =>
    {
        var route = req.GetRoute();
        if (route != null && route.Path == "/tenant/{TenantName}/resource")
        {
            dto.SubResourceName = "CustomResource";
        }
    });
}
```

### Apply custom behavior to multiple DTO's with Interfaces

Typed Filters can also be used to apply custom behavior on Request DTO's sharing a common interface, e.g:

```csharp
public override void Configure(Container container)
{
    RegisterTypedRequestFilter<IHasSharedProperty>((req, res, dtoInterface) => {
        dtoInterface.SharedProperty = "Is Shared";    
    });
}
```

### Message Queue Endpoints

Non-HTTP requests like [Redis MQ](?id=Messaging-and-Redis) are treated as _Internal Requests_ which only execute the alternate `GlobalMessageRequestFilters` and `GlobalMessageResponseFilters` and Action [Filter attributes](?id=filter-attributes). 