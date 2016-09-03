---
#File header for Jekyll to pick up 
---
## HTTP Custom hooks

This list shows the order in which any user-defined custom hooks are executed:

  1. `HostContext.RawHttpHandlers` are executed before anything else, i.e. returning any ASP.NET `IHttpHandler` by-passes ServiceStack completely and processes your custom `IHttpHandler` instead.
  2. If the Request doesn't match any existing Routes it will search `IAppHost.CatchAllHandlers` for a match
  3. The `IAppHost.PreRequestFilters` gets executed before the Request DTO is deserialized
  4. Default Request DTO Binding or [Custom Request Binding][4] _(if registered)_
  5. Any [Request Converters](https://github.com/ServiceStack/ServiceStack/wiki/Customize-HTTP-Responses#request-converters) are executed
  5. [Request Filter Attributes][3] with **Priority < 0** gets executed
  6. Then any [Global Request Filters][1] get executed
  7. Followed by [Request Filter Attributes][3] with **Priority >= 0**
  8. Action Request Filters
  9. Then your **Service is executed** with the configured [IServiceRunner<T>](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/Web/IServiceRunner.cs) and its **OnBeforeExecute**, **OnAfterExecute** and **HandleException** custom hooks are fired
  10. Action Response Filters
  11. Any [Response Converters](https://github.com/ServiceStack/ServiceStack/wiki/Customize-HTTP-Responses#response-converters) are executed
  11. Followed by [Response Filter Attributes][3] with **Priority < 0** 
  12. Then [Global Response Filters][1] 
  13. Followed by [Response Filter Attributes][3] with **Priority >= 0** 
  14. Finally at the end of the Request `IAppHost.OnEndRequest` and any `IAppHost.OnEndRequestCallbacks` are fired

Any time you close the Response in any of your filters, i.e. `httpRes.EndRequest()` the processing of the response is short-circuited and no further processing is done on that request.

## MQ (non-HTTP) Custom hooks

  1. Any [Global Request Filters](https://github.com/ServiceStack/ServiceStack/wiki/Request-and-response-filters#message-queue-endpoints) get executed
  2. Followed by [Request Filter Attributes][3] with **Priority >= 0**
  3. Action Request Filters
  4. Then your **Service is executed** with the configured [IServiceRunner<T>](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/Web/IServiceRunner.cs) and its **OnBeforeExecute**, **OnAfterExecute** and **HandleException** custom hooks are fired
  5. Action Response Filters
  6. Then [Global Response Filters](https://github.com/ServiceStack/ServiceStack/wiki/Request-and-response-filters#message-queue-endpoints) 
  7. Finally at the end of the Request `IAppHost.OnEndRequest` is fired

## Implementation architecture diagram

The [Implementation architecture diagram][2] shows a visual cue of the internal order of operations that happens in ServiceStack:

![ServiceStack Overview](http://mono.servicestack.net/files/servicestack-overview-01.png)

After the IHttpHandler is returned, it gets executed with the current ASP.NET or HttpListener request wrapped in a common [IHttpRequest](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/ServiceHost/IHttpRequest.cs) instance. 

The implementation of [RestHandler](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack/WebHost.Endpoints/RestHandler.cs) shows what happens during a typical ServiceStack request:

![ServiceStack Request Pipeline](http://mono.servicestack.net/files/servicestack-overview-02.png)

  [1]: https://github.com/ServiceStack/ServiceStack/wiki/Request-and-response-filters
  [2]: https://github.com/ServiceStack/ServiceStack/wiki/Architecture-overview
  [3]: https://github.com/ServiceStack/ServiceStack/wiki/Filter-attributes
  [4]: https://github.com/ServiceStack/ServiceStack/wiki/Serialization-deserialization