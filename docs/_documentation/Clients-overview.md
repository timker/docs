---
slug: clients-overview
---
As ServiceStack is just a pure HTTP web service it can be accessed with any HTTP-capable client. 

Although in addition to this the [ServiceStack.Client](http://nuget.org/packages/ServiceStack.Client) library and NuGet package also include multiple service clients on [Xamarin iOS, Android, Windows Store, Silverlight 5, WPF, etc](https://github.com/ServiceStackApps/HelloMobile) which are optimized for consuming ServiceStack Services including built-in [Error handling](https://github.com/ServiceStack/ServiceStack/wiki/Error-Handling), [Predefined Routes](https://github.com/ServiceStack/ServiceStack/wiki/Routing), [Auto Batched Requests](https://github.com/ServiceStack/ServiceStack/wiki/Auto-Batched-Requests), etc. 

### ServiceStack Clients

  * [[C# client]]
  * [TypeScript client](https://github.com/ServiceStack/ServiceStack/wiki/TypeScript-Add-ServiceStack-Reference)
  * [Java/Kotlin Client](https://github.com/ServiceStack/ServiceStack/wiki/Java-Add-ServiceStack-Reference)
  * [Swift Client](https://github.com/ServiceStack/ServiceStack/wiki/Swift-Add-ServiceStack-Reference)
  * [Silverlight client](https://github.com/ServiceStack/ServiceStack/wiki/SilverlightServiceClient)
  * [[JavaScript client]]
  * [[Dart Client]]
  * [MQ Clients](https://github.com/ServiceStack/ServiceStack/wiki/Messaging-and-redis)

### Add ServiceStack Reference

You can also use [ServiceStackVS](https://github.com/ServiceStack/ServiceStack/wiki/Creating-your-first-project#step-1-download-and-install-servicestackvs)'s **Add ServiceStack Reference** feature for adding generated Native Types to client VS.NET C#, F# and VB.NET projects. This gives developers a consistent way of creating and updating your DTOs regardless of your language of choice.

### Supported Languages


* [C# Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/CSharp-Add-ServiceStack-Reference)
* [TypeScript Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/TypeScript-Add-ServiceStack-Reference)
* [Swift Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/Swift-Add-ServiceStack-Reference)
* [Java Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/Java-Add-ServiceStack-Reference)
* [Kotlin Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/Kotlin-Add-ServiceStack-Reference)
* [F# Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/FSharp-Add-ServiceStack-Reference)
* [VB.NET Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/VB.Net-Add-ServiceStack-Reference)

***

### .NET Clients

There are multiple C# service clients included, each optimized for their respective formats:

![ServiceStack HTTP Client Architecture](http://mono.servicestack.net/files/servicestack-httpclients.png) 

- [JSON Client](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Client/JsonServiceClient.cs)
- [XML Client](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Client/XmlServiceClient.cs)
- [JSV Client](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Client/JsvServiceClient.cs)
- [SOAP 1.1/1.2 Clients](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Client/Soap12ServiceClient.cs)
- [ProtoBuf Client](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.ProtoBuf/ProtoBufServiceClient.cs) available in the [ServiceStack.Plugins.ProtoBuf](http://nuget.org/packages/ServiceStack.Plugins.ProtoBuf) NuGet package.

All clients share the same [IServiceClient](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/IServiceClient.cs) and [IServiceClientAsync](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/IServiceClientAsync.cs) so they're easily swappable at runtime, and is what allows the same Unit test to be re-used as within an [Xml, JSON, JSV, SOAP Integration test](https://github.com/ServiceStack/ServiceStack/blob/master/tests/ServiceStack.WebHost.IntegrationTests/Tests/WebServicesTests.cs). The JSON, XML and JSV clients also share [IRestClient](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/IRestClient.cs) and [IRestClientAsync](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/IRestClientAsync.cs)

## What's the best way to expose our services to clients today?

### Native language client libraries

A productive option for clients (and the recommended approach by ServiceStack) would be to provide a native client library for each of the popular languages you wish to support. This is the approach of companies who really, really want to help you use their services like Amazon, Facebook and Windows Azure. This is an especially good idea if you want to support static languages (i.e. C# and Java) where having typed client libraries saves end-users from reverse engineering the types and API calls. It also saves them having to look up documentation since a lot of it can be inferred from the type info. ServiceStack's and Amazons convention of having `ServiceName` and `ServiceNameResponse` for each service also saves users from continually checking documentation to work out what the response of each service will be.

### Packaging client libraries

In terms of packaging your client libraries, sticking a link to a zip file on your Websites APIs documentation page would be the easiest approach. If the zip file was a link to a master archive of a Github repository, that would be better as you'll be able to accept bug fixes and usability tips from the community. Finally we believe the best way to make your client libraries available would be to host them in the target languages native package manager - letting end-users issue 1-command to automatically add it to their project, and another to easily update it when your service has changed.

### Using NuGet

For .NET this means adding it to NuGet, and if you use ServiceStack your package would just need to contain your types with a reference to [ServiceStack.Client](http://nuget.org/packages/ServiceStack.Client). One of the benefits of using ServiceStack is that all your types are already created since it's what you used to define your web services with!


# Community Resources

  - [Servicestack and PHP](http://www.majorsilence.com/servicestack_and_php) by [@majorsilence](https://github.com/majorsilence)