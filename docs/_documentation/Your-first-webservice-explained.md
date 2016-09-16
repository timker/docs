---
slug: your-first-webservice-explained
---
Let's look a bit deeper into the Hello World service you created:

As you have seen, the convention for response DTO is `RequestDTO` and  `RequestDTOResponse`. **Note, request and response DTO should be in the same namespace if you want ServiceStack to recognize the DTO pair**.

To support automatic exception handling, you also need to add a `ResponseStatus` property to the response DTO:

```csharp
//Request DTO
public class Hello
{
    public string Name { get; set; }
}

//Response DTO
//Follows naming convention
public class HelloResponse
{
    public ResponseStatus ResponseStatus { get; set; } //Automatic exception handling
    
    public string Result { get; set; }
}
```

A service class is marked as such by its inheritance of the empty `IService` interface. You will generally want to do so by having such a class inherit from the more convenient `Service` base class which provides easy access to the most common functionality. 

```csharp
public class HelloService : Service
{
    public object Any(Hello request)
    {
        return new HelloResponse { Result = "Hello, " + request.Name };
    }
}
```

The above service can be called with **Any** HTTP Verb (e.g. GET, POST,..) and from any endpoint or format (e.g. JSON, XML, etc). You can also choose to handle a specific Verb by changing the method name to suit, e.g. here's how to change it so you only handle HTTP **GET** requests:

```csharp
public class HelloService : Service
{
    public object Get(Hello  request)
    {
        return new HelloResponse { Result = "Hello, " + request.Name };
    }
}
```

***

To register your custom REST URLs, you can use the `Route` attribute on the request DTO:

```csharp
//Request DTO
[Route("/hello")]
[Route("/hello/{Name}")]
public class Hello
{
    public string Name { get; set; }
}
```

## Calling your web service

The above service can now be called with:

```csharp
var client = new JsonServiceClient(BaseUri);
HelloResponse response = client.Get<HelloResponse>("/hello/World!"); 
```

You can also make it even easier for your C# clients if you also provide the expected return type. E.g. if you add the `IReturn<T>` interface marker to your Request DTO like:

```csharp
public class Hello : IReturn<HelloResponse>
{
    public string Name { get; set; }
}
```

Your clients will now also be able to make the same call above but with a fully-typed C# API, i.e:

```csharp
HelloResponse response = client.Get(new Hello { Name = "World!" });
```

We highly recommend annotating your Request DTO's with the above `IReturn<T>` marker as it enables a generic typed API without clients having to know and specify the Response at each call-site, which would be invalidated and need to be manually updated if the Service Response Type changes.

More details on the Service Clients is available on the [C#/.NET Service Clients page](?id=csharp-client).

### Routing Tips

```csharp
[Route("/hello/{Name}")]
```

only matches:

    /hello/name

whereas:

```csharp
[Route("/hello/{Name*}")]
```

matches:

    /hello
    /hello/name
    /hello/my/name/is/ServiceStack 


More details about Routing is available on the [Routing page](?id=routing).

Next wikipage: [The IoC container](?id=the-ioc-container)
 