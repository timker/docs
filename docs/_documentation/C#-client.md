---
slug: csharp-client
title: C#/.NET Service Clients
---

Using DTOs to define your web service interface makes it possible to provide strong-typed generic service clients without any code-gen or extra build-steps, leading to a productive end-to-end type-safe communication gateway from client to server.

**Note**: you have to install the NuGet package **ServiceStack.Client** in your client project, e.g. with the following command in the package manager console:

    PM> Install-Package ServiceStack.Client

Alternatively you can use the [HttpClient-based JsonHttpClient](?id=CSharp-client#jsonhttpclient) in:

    PM> Install-Package ServiceStack.HttpClient

These packages also contain PCL versions of the Service Clients available with support for [Xamarin.iOS, Xamarin.Android, Windows Store, WPF and Silverlight 5](https://github.com/ServiceStackApps/HelloMobile) platforms.

### [Cache Aware Service Clients](?id=Cache-Aware-Clients)

When [caching is enabled on Services](?id=HTTP-Caching), the Cache-aware Service Clients can dramatically improve performance by eliminating server requests entirely as well as reducing bandwidth for re-validated requests. They also offer an additional layer of resiliency as re-validated requests that result in Errors will transparently fallback to using pre-existing locally cached responses. For bandwidth-constrained environments like Mobile Apps they can dramatically improve the User Experience.

The Cache-Aware clients implement the full `IServiceClient` interface so they should be an easy drop-in enhancement for existing Apps:

```csharp
IServiceClient client = new JsonServiceClient(baseUrl).WithCache(); 

//equivalent to:
IServiceClient client = new CachedServiceClient(new JsonServiceClient(baseUrl));
```

Likewise for the HttpClient-based `JsonHttpClient`:

```csharp
IServiceClient client = new JsonHttpClient(baseUrl).WithCache(); 

//equivalent to:
IServiceClient client = new CachedHttpClient(new JsonHttpClient(baseUrl));
```

## REST API

All ServiceStack's C# clients share the same interfaces and are created by passing in the **Base URI** of your ServiceStack service in the clients constructor, e.g. if your ServiceStack instance was hosted on the root path `/` on the 8080 custom port:

```csharp
var client = new JsonServiceClient("http://host:8080/");
```

Or if hosted on the `/api` custom path:

```csharp
var client = new JsonServiceClient("http://host/api/");
```

In addition, the Service Clients provide HTTP verbs (Get, Post & PostFile, Put, Delete, Patch, etc) enabling a productive typed API for consuming ServiceStack Services with their best matching Custom Routes as seen in the examples below:

> See [IServiceClient](https://github.com/ServiceStack/ServiceStack/blob/master/docs/pages/IServiceClient.md) for the full API available

### Using the recommended [API Design](?id=api-design)

```csharp
HelloResponse response = client.Get(new Hello { Name = "World!" });
response.Result.Print();
```

**Async Example**

Using C# `await`:

```csharp
HelloResponse response = await client.GetAsync(
    new Hello { Name = "World!" });
```

Using Tasks:

```csharp
client.GetAsync(new Hello { Name = "World!" })
    .Success(r => r => r.Result.Print())
    .Error(ex => { throw ex; });
```

### Alternative API

```csharp
var response = client.Get<HelloResponse>("/hello/World!");
response.Result.Print();
```
**Async Example**

```csharp
var response = await client.GetAsync<HelloResponse>("/hello/World!");
```

## Service Client API

C#/.NET Clients can call the above Hello Service using any of the JSON, JSV, XML or SOAP Service Clients with the code below:

### Using the recommended [API Design](?id=api-design)

```csharp
var response = client.Send(new Hello { Name = "World!" });
response.Result.Print();
```

**Async Example**

```csharp
var response = await client.SendAsync(new Hello { Name = "World!" });
response.Result.Print();
```

### Alternative API

```csharp
var response = client.Send<HelloResponse>(new Hello { Name = "World!" });
response.Result.Print();
```

**Async Example**

```csharp
var response = await client.SendAsync<HelloResponse>(
    new Hello { Name = "World!" });
```

The service clients use the automatic [pre-defined routes](?id=Endpoints#) for each service.

<a name="native-responses"></a>

## Support for Native built-in Response Types

All of ServiceStack's generic Service Clients also allow you to fetch raw `string`, `byte[]` and `Stream` responses of any existing service, or when you need it, the underlying `HttpWebResponse` allowing fine-grained access to the HTTP Response. e.g With just the Service below:

```csharp
[Route("/poco/{Text}")]
public class Poco : IReturn<PocoResponse>
{
    public string Text { get; set; }
}

public class PocoResponse
{
    public string Result { get; set; }
}

public class NativeTypesExamples : Service
{
    public PocoResponse Any(Poco request)
    {
        base.Response.AddHeader("X-Response", request.Text);
        return new PocoResponse { 
            Result = "Hello, " + (request.Text ?? "World!") 
        };
    }
}
```

You can access it normally with the typed API:

```csharp
PocoResponse response = client.Get(new Poco { Text = "World" });
response.Result //Hello, World
```

Or as get the JSON as a raw string:

```csharp
string responseJson = client.Get<string>("/poco/World");
var dto = responseJson.FromJson<PocoResponse>();
dto.Result //Hello, World
```

Or as raw bytes:

```csharp
byte[] responseBytes = client.Get<byte[]>("/poco/World");
var dto = responseBytes.FromUtf8Bytes().FromJson<PocoResponse>();
dto.Result //Hello, World
```

Or as a Stream:

```csharp
using (Stream responseStream = client.Get<Stream>("/poco/World")) {
    var dto = responseStream.ReadFully()
        .FromUtf8Bytes()
        .FromJson<PocoResponse>();
    dto.Result //Hello, World
}
```

Or even access the populated `HttpWebResponse` object:

```csharp
HttpWebResponse webResponse = client.Get<HttpWebResponse>("/poco/World");

webResponse.Headers["X-Response"] //World
using (var stream = webResponse.GetResponseStream())
using (var sr = new StreamReader(stream)) {
    var dto = sr.ReadToEnd().FromJson<PocoResponse>();
    dto.Result //Hello, World
}
```

### Accessing raw service responses

ServiceStack isn't limited to just returning POCO's as you can effectively [return anything you want](?id=Service-return-types) even images 
[/helloimage/ServiceStack?Width=600&height=300&Foreground=Yellow](http://test.servicestack.net/image-draw/ServiceStack?Width=600&height=300&Foreground=Yellow). These native responses can also be mark on your Request DTO `IReturn<T>` interface marker to give you a terse end-to-end API for fetching raw responses, e.g:

```csharp
[Route("/headers/{Text}")]
public class Headers : IReturn<HttpWebResponse>
{
    public string Text { get; set; }
}

[Route("/strings/{Text}")]
public class Strings : IReturn<string>
{
    public string Text { get; set; }
}

[Route("/bytes/{Text}")]
public class Bytes : IReturn<byte[]>
{
    public string Text { get; set; }
}

[Route("/streams/{Text}")]
public class Streams : IReturn<Stream>
{
    public string Text { get; set; }
}

public class BuiltInTypesService : Service
{
    public void Any(Headers request)
    {
        base.Response.AddHeader("X-Response", request.Text);
    }

    public string Any(Strings request)
    {
        return "Hello, " + (request.Text ?? "World!");
    }

    public byte[] Any(Bytes request)
    {
        return new Guid(request.Text).ToByteArray();
    }

    public byte[] Any(Streams request)
    {
        return new Guid(request.Text).ToByteArray();
    }        
}
```

### Accessing client raw responses

Which let you access the results as you would a normal response:

```csharp
using (HttpWebResponse response = client.Get(new Headers { Text = "World" }))
{
    response.Headers["X-Response"] // "World"
}

string response = client.Get(new Strings { Text = "World" });
response // Hello, World

byte[] response = client.Get(new Bytes { 
    Text = Guid.NewGuid().ToString() 
});
var guid = new Guid(response);

using (Stream stream = client.Get(new Streams { Text = Guid.NewGuid().ToString() })) 
{
    var guid = new Guid(response.ReadFully());
}
```

All these APIs are also available asynchronously as well:

```csharp
using (HttpWebResponse response = await client.GetAsync(
    new Strings { Text = "Test" })) {
    response.Headers["X-Response"] // "World"
}

string response = await client.GetAsync(
    new Strings { Text = "World" });
response // Hello, World

byte[] response = await client.GetAsync(new Bytes { 
    Text = Guid.NewGuid().ToString() 
});
var guid = new Guid(response);

using (Stream stream = await client.GetAsync(new Streams { 
    Text = Guid.NewGuid().ToString() 
})) 
{
    var guid = new Guid(response.ReadFully());
}
```

> Note: You must explicitly dispose all APIs returning either `HttpWebResponse` or `Stream` as seen in the above examples.

They all behave the same as the sync versions except for `HttpWebResponse` which gets returned just after
the request is sent (asynchronously) and before any response is read so you can still access the HTTP Headers e.g:

```csharp
var client = new JsonServiceClient("http://localhost:2020/") {
    ResponseFilter = httpRes => {
        var header = httpRes.Headers["X-Response"];
    }
};
var response = await client.GetAsync(new Headers { Text = "World" });
```

Which makes a great starting point if you want to stream the responses back asynchronously as seen in this
[Reactive ServiceStack example](https://gist.github.com/bamboo/5078236) by [@rodrigobamboo](https://twitter.com/rodrigobamboo).

More examples can be found in the ServiceClients [Built-in native type response tests](https://github.com/ServiceStack/ServiceStack/blob/master/tests/ServiceStack.WebHost.Endpoints.Tests/ServiceClientsBuiltInResponseTests.cs)

## Sending Raw Data

.NET Service Clients can also send raw `string`, `byte[]` or `Stream` Request bodies in their custom Sync or Async API's, e.g:
 
```csharp
string json = "{\"Key\":1}";
client.Post<SendRawResponse>("/sendraw", json);

byte[] bytes = json.ToUtf8Bytes();
client.Put<SendRawResponse>("/sendraw", bytes);

Stream stream = new MemoryStream(bytes);
await client.PostAsync<SendRawResponse>("/sendraw", stream);
```

## Authentication

ServiceStack's [Auth Tests](https://github.com/ServiceStack/ServiceStack/blob/master/tests/ServiceStack.WebHost.Endpoints.Tests/AuthTests.cs#L108) shows different ways of authenticating when using the C# Service Clients. By default BasicAuth and DigestAuth is built into the clients, e.g:

```csharp
var client = new JsonServiceClient(baseUri) {
    UserName = UserName,
    Password = Password,
};

var request = new Secured { Name = "test" };
var response = client.Send<SecureResponse>(request);    
```

Behind the scenes ServiceStack will attempt to send the request normally but when the request is rejected and challenged by the Server the clients will automatically retry the same request but this time with the Basic/Digest Auth headers.

To skip the extra hop when you know you're accessing a secure service, you can tell the clients to always send the BasicAuth header with:

```csharp
client.AlwaysSendBasicAuthHeader = true;
```

The alternative way to Authenticate is to make an explicit call to the `Auth` service (this requires CredentialsAuthProvider enabled) e.g:

```csharp
AuthResponse authResponse = client.Post(new Auth {
    provider = CredentialsAuthProvider.Name,
    UserName = "user",
    Password = "p@55word",
    RememberMe = true,  //important tell client to retain permanent cookies
});

var request = new Secured { Name = "test" };
var response = client.Send<SecureResponse>(request);    
```

After a successful call to the `Auth` service the client is Authenticated and if **RememberMe** is set, the client will retain the Session Cookies added by the Server on subsequent requests which is what enables future requests from that client to be authenticated.

### Upload and Download Progress on Async API's

The Async API's support on progress updates with the `OnDownloadProgress` and `OnUploadProgress` callbacks which can be used to provide UX Progress updates, e.g:

```csharp
var client = new JsonServiceClient(ListeningOn);

//Available in ASP.NET/HttpListener when downloading responses with known lengths 
//E.g: Strings, Files, etc.
client.OnDownloadProgress = (done, total) =>
    "{0}/{1} bytes downloaded".Print(done, total);

var response = await client.GetAsync(new Request());
```

> Note: total = -1 when 'Transfer-Encoding: chunked'

Whilst the `OnUploadProgress` callback gets fired when uploading files, e.g:

```csharp
client.OnUploadProgress = (bytesWritten, total) => 
    "Written {0}/{1} bytes...".Print(bytesWritten, total);

client.PostFileWithRequest<UploadResponse>(url, 
    new FileInfo(path), new Upload { CreatedBy = "Me" });
```

### Custom Client Caching Strategy

The `ResultsFilter` and `ResultsFilterResponse` delegates on Service Clients can be used to enable a custom caching strategy. 

Here's a basic example implementing a cache for all **GET** Requests:

```csharp
var cache = new Dictionary<string, object>();

client.ResultsFilter = (type, method, uri, request) => {
    if (method != HttpMethods.Get) return null;
    object cachedResponse;
    cache.TryGetValue(uri, out cachedResponse);
    return cachedResponse;
};
client.ResultsFilterResponse = (webRes, response, method, uri, request) => {
    if (method != HttpMethods.Get) return;
    cache[uri] = response;
};

//Subsequent requests returns cached result
var response1 = client.Get(new GetCustomer { CustomerId = 5 });
var response2 = client.Get(new GetCustomer { CustomerId = 5 }); //cached response
```

The `ResultsFilter` delegate is executed with the context of the request before the request is made. Returning a value of type `TResponse` short-circuits the request and returns that response. Otherwise the request continues and its response passed into the `ResultsFilterResponse` delegate where it can be cached. 

### Implicitly populate SessionId and Version Number

Service Clients can be used to auto-populate Request DTO's implementing `IHasSessionId` or `IHasVersion` by assigning the `Version` and `SessionId` properties on the Service Client, e.g:

```csharp
client.Version = 1;
client.SessionId = authResponse.SessionId;
```

Which populates the SessionId and Version number on each Request DTO's that implementing the specific interfaces, e.g:

```csharp
public class Hello : IReturn<HelloResponse>, IHasSessionId, IHasVersion {
    public int Version { get; set; }
    public string SessionId { get; set; }
    public string Name { get; set; }
}

client.Get(new Hello { Name = "World" }); //Auto populates Version and SessionId
```

### HTTP Verb Interface Markers

You can decorate your Request DTO's using the `IGet`, `IPost`, `IPut`, `IDelete` and `IPatch` interface markers and the `Send` and  `SendAsync` API's will use it to automatically send the Request using the selected HTTP Method. E.g:

```csharp
public class HelloByGet : IGet, IReturn<HelloResponse>
{
    public string Name { get; set; }
}
public class HelloByPut : IPut, IReturn<HelloResponse> 
{
    public string Name { get; set; }
}

var response = client.Send(new HelloByGet { Name = "World" }); //GET

await client.SendAsync(new HelloByPut { Name = "World" }); //PUT
```

Interface markers is supported in all .NET Service Clients, they're also included in the generated 
[Add ServiceStack Reference](?id=Add-ServiceStack-Reference) DTO's so they're also available in the
[Java JsonServiceClient](?id=Java-Add-ServiceStack-Reference) and
[Swift JsonServiceClient](?id=Swift-Add-ServiceStack-Reference). It's also available in our 3rd Party [StripeGateway](https://github.com/ServiceStack/Stripe).

Whilst a simple feature, it enables treating your remote services as a message-based API 
[yielding its many inherent advantages](?id=Advantages-of-message-based-web-services#advantages-of-message-based-designs) 
where your Application API's need only pass Request DTO models around to be able to invoke remote Services, decoupling the Service Request from its implementation which can be now easily managed by a high-level adapter that takes care of proxying the Request to the underlying Service Client. The adapter could also add high-level functionality of it's own including auto retrying of failed requests, generic error handling, logging/telemetrics, event notification, throttling, offline queuing/syncing, etc.

### Multiple File Uploads

The `PostFilesWithRequest` APIs available in all .NET Service Clients allow you to easily upload multiple 
streams within a single HTTP request. It supports populating Request DTO with any combination of QueryString 
and POST'ed FormData in addition to multiple file upload data streams:

```csharp
using (var stream1 = uploadFile1.OpenRead())
using (var stream2 = uploadFile2.OpenRead())
{
    var client = new JsonServiceClient(baseUrl);
    var response = client.PostFilesWithRequest<MultipleFileUploadResponse>(
        "/multi-fileuploads?CustomerId=123",
        new MultipleFileUpload { CustomerName = "Foo,Bar" },
        new[] {
            new UploadFile("upload1.png", stream1),
            new UploadFile("upload2.png", stream2),
        });
}
```

Example using only a Typed Request DTO. The `JsonHttpClient` also includes async equivalents for each of the 
`PostFilesWithRequest` APIs:

```csharp
using (var stream1 = uploadFile1.OpenRead())
using (var stream2 = uploadFile2.OpenRead())
{
    var client = new JsonHttpClient(baseUrl);
    var response = await client.PostFilesWithRequestAsync<MultipleFileUploadResponse>(
        new MultipleFileUpload { CustomerId = 123, CustomerName = "Foo,Bar" },
        new[] {
            new UploadFile("upload1.png", stream1),
            new UploadFile("upload2.png", stream2),
        });
}
```

### ServiceClient URL Resolvers

The urls used in all .NET Service Clients are now customizable with the new `UrlResolver` and `TypedUrlResolver` 
delegates. 

E.g. you can use this feature to rewrite the URL used with the Request DTO Type Name used as the subdomain by:

```csharp
[Route("/test")] 
class Request {}

var client = JsonServiceClient("http://example.org/api") {
    TypedUrlResolver =  (meta, httpMethod, dto) => 
        meta.BaseUri.Replace("example.org", dto.GetType().Name + ".example.org")
            .CombineWith(dto.ToUrl(httpMethod, meta.Format)));
};

var res = client.Get(new Request());  //= http://Request.example.org/api/test
var res = client.Post(new Request()); //= http://Request.example.org/api/test
```

This feature is also implemented in `JsonHttpClient`, examples below shows rewriting APIs that use custom urls:

```csharp
var client = JsonHttpClient("http://example.org/api") {
    UrlResolver = (meta, httpMethod, url) => 
        meta.BaseUri.Replace("example.org", "111.111.111.111").CombineWith(url))
};

await client.DeleteAsync<MockResponse>("/dummy"); 
//=http://111.111.111.111/api/dummy

await client.PutAsync<MockResponse>("/dummy", new Request()); 
//=http://111.111.111.111/api/dummy
```

## [ServiceStack.Discovery.Consul](https://github.com/wwwlicious/servicestack-discovery-consul)

This feature makes it easier to support features like
[ServiceStack.Discovery.Consul](https://github.com/wwwlicious/servicestack-discovery-consul)
plugin which enables external RequestDTO endpoint discovery 
by integrating with [Consul.io](http://consul.io) to provide automatic service registration and health checking.


*** 

## Built-in Clients

All REST and ServiceClients share the same interfaces (`IServiceClient`, `IRestClient` and `IRestClientAsync`) so they can easily be replaced (for increased perf/debuggability/etc) with a single line of code.

### JsonHttpClient

The new `JsonHttpClient` is an alternative to the existing generic typed `JsonServiceClient` for consuming ServiceStack Services which instead of using **HttpWebRequest** is based on Microsoft's latest async [HttpClient](https://www.nuget.org/packages/Microsoft.Net.Http). 

JsonHttpClient implements the full [IServiceClient API](https://gist.github.com/mythz/4683438240820b522d39) making it an easy drop-in replacement for your existing JsonServiceClient where in most cases it can simply be renamed to JsonHttpClient, e.g:

```csharp
//IServiceClient client = new JsonServiceClient("http://techstacks.io");
IServiceClient client = new JsonHttpClient("http://techstacks.io");

var response = await client.GetAsync(new GetTechnology { Slug = "servicestack" })
```

#### Install

JsonHttpClient can be downloaded from NuGet at:

    > Install-Package ServiceStack.HttpClient

### [ModernHttpClient](https://github.com/paulcbetts/ModernHttpClient)

One of the primary benefits of being based on `HttpClient` is being able to make use of 
[ModernHttpClient](https://github.com/paulcbetts/ModernHttpClient) which provides a thin wrapper around iOS's native `NSURLSession` or `OkHttp` client on Android, offering improved stability for 3G mobile connectivity.

To enable, install [ModernHttpClient](https://www.nuget.org/packages/ModernHttpClient) then set the 
Global HttpMessageHandler Factory to configure all `JsonHttpClient` instances to use ModernHttpClient's `NativeMessageHandler`: 

```csharp
JsonHttpClient.GlobalHttpMessageHandlerFactory = () => new NativeMessageHandler()
```

Alternatively, you can configure a single client instance to use ModernHttpClient with:

```csharp
client.HttpMessageHandler = new NativeMessageHandler();
```

### Differences with JsonServiceClient

Whilst the goal is to retain the same behavior in both clients, there are some differences resulting from using HttpClient where the Global and Instance Request and Response Filters are instead passed HttpClients `HttpRequestMessage` and `HttpResponseMessage`. 

Also, all API's are **Async** under-the-hood where any Sync API's that doesn't return a `Task<T>` just blocks on the Async `Task.Result` response. As this can dead-lock in certain environments we recommend sticking with the Async API's unless safe to do otherwise. 

### HttpWebRequest Service Clients

Whilst the list below contain the built-in clients based on .NET's built-in `HttpWebRequest`:

- implements both `IRestClient` and `IServiceClient`:
    - [JsonServiceClient](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Client/JsonServiceClient.cs)
    (uses default endpoint with **JSON**) - recommended
    - [JsvServiceClient](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Client/JsvServiceClient.cs)
    (uses default endpoint with **JSV**)
    - [XmlServiceClient](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Client/XmlServiceClient.cs)
    (uses default endpoint with **XML**)
    - [CsvServiceClient](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Client/CsvServiceClient.cs)
    (uses default endpoint with **CSV**)
    - [MsgPackServiceClient](?id=MessagePack-Format)
    (uses default endpoint with **Message-Pack**)
    - [ProtoBufServiceClient](?id=Protobuf-format)
    (uses default endpoint with **Protocol Buffers**)
- implements `IServiceClient` only:
    - [Soap11ServiceClient](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Client/Soap11ServiceClient.cs) (uses **SOAP 11** endpoint)
    - [Soap12ServiceClient](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Client/Soap12ServiceClient.cs)
    (uses **SOAP 12** endpoint)

#### Install

The HttpWebRequest clients above are available in:

    > Install-Package ServiceStack.Client

# Community Resources

  - [Reactive ServiceStack](https://gist.github.com/bamboo/5078236) by [@rodrigobamboo](https://twitter.com/rodrigobamboo)