ServiceStack Plugin API provides a declarative way to enable modular functionality in ServiceStack:

## Plugin API

```csharp
public interface IPlugin
{
    void Register(IAppHost appHost);
}
```

#### Custom Logic before Plugins are loaded

If your plugin also implements `IPreInitPlugin` it will get run before any plugins are registered:

```csharp
public interface IPreInitPlugin
{
    void Configure(IAppHost appHost);
}
```

#### Custom Logic after Plugins are loaded

If your plugin implements `IPostInitPlugin` it will get run after all plugins are registered:

```csharp
public interface IPostInitPlugin
{
    void AfterPluginsLoaded(IAppHost appHost);
}
```

#### Disabling Plugins via Feature Enum Flags

All built-in Plugins are Registered and available via **base.Plugins** before your **Configure()** script is run so you have a chance to modify the behaviour or remove un-used plugins which is exactly what the short-hand:

```csharp
SetConfig(new HostConfig { 
    EnableFeatures = Feature.All.Remove(Feature.Csv)
});
```

Which under the covers just does:


```csharp
if ((Feature.Csv & config.EnableFeatures) != Feature.Csv)
    Plugins.RemoveAll(x => x is CsvFormat);
```

Which you now also have an opportunity to also do in your AppHost Configure() start-up script yourself - if you want to remove or customize any pre-loaded plugins.

#### Resolving Plugins

You can easily use LINQ to fetch any specific plugin:

```csharp
var htmlFormat = base.Plugins.First(x => x is HtmlFormat) as HtmlFormat;
```

Which is also what the `this.GetPlugin<T>()` convenience extension method does:

```csharp
var htmlFormat = base.GetPlugin<HtmlFormat>();
```

# List of Plugins added by Default

A list of all of the plugins available on ServiceStack and how to add them:

## Auto-registered plugins
These plugins below **are already added by default**, you can remove or customize them using the methods described above.

### Metadata Feature
Provides ServiceStack's [auto-generated metadata pages](https://github.com/ServiceStack/ServiceStack/wiki/Metadata-page).  

```csharp
var feature = Plugins.FirstOrDefault(x => x is MetadataFeature); 
Plugins.RemoveAll(x => x is MetadataFeature); 
```

### Predefined Routes
Provides ServiceStack's [pre-defined routes](https://github.com/ServiceStack/ServiceStack/wiki/Routing#pre-defined-routes) used in the built-in [C# Service Clients](https://github.com/ServiceStack/ServiceStack/wiki/C%23-client).  

```csharp
var feature = Plugins.FirstOrDefault(x => x is PredefinedRoutesFeature); 
Plugins.RemoveAll(x => x is PredefinedRoutesFeature); 
```

### Request Info
Provides ServiceStack's Request Info feature useful for debugging requests. Just add **?debug=requestinfo** in your `/pathinfo` and ServiceStack will return a dump of all the HTTP Request parameters to help with with debugging interoperability issues. The RequestInfoFeature is only enabled for **Debug** builds.

```csharp
var feature = Plugins.FirstOrDefault(x => x is RequestInfoFeature); 
Plugins.RemoveAll(x => x is RequestInfoFeature); 
```

### [CSV Format](https://github.com/ServiceStack/ServiceStack/wiki/ServiceStack-CSV-Format)
Providing ServiceStack's [CSV Format](https://github.com/ServiceStack/ServiceStack/wiki/ServiceStack-CSV-Format). 

```csharp
var feature = Plugins.FirstOrDefault(x => x is CsvFormat); 
Plugins.RemoveAll(x => x is CsvFormat); 
```

Note: By default the CSV Format tries serialize the Response object directly into CSV which is only ideal if your responses return `List<Poco>`. If however you mark your Response DTO with the **[Csv(CsvBehavior.FirstEnumerable)]** attribute the CSV Format instead will only serialize the first `IEnumerable<T>` it finds on your Response DTO e.g. if you had a `List<Poco> Results` property it will only serialize this list in the tabular CSV Format which is typically the behaviour you want.

### [Html Format](https://github.com/ServiceStack/ServiceStack/wiki/HTML5ReportFormat)
Providing ServiceStack's [Html Format](https://github.com/ServiceStack/ServiceStack/wiki/HTML5ReportFormat). 

```csharp
var feature = Plugins.FirstOrDefault(x => x is HtmlFormat); 
Plugins.RemoveAll(x => x is HtmlFormat); 
```

### [Razor Markdown Format](https://github.com/ServiceStack/ServiceStack/wiki/Markdown-Razor)
This provides ServiceStack's [Razor Markdown Format](https://github.com/ServiceStack/ServiceStack/wiki/Markdown-Razor) and also enables ServiceStack to serve static **.md** or **.markdown** files in either plain text, rendered as HTML (partial), or rendered in HTML inside a static **_Layout.shtml** HTML template. 

```csharp
var feature = Plugins.FirstOrDefault(x => x is MarkdownFormat); 
Plugins.RemoveAll(x => x is MarkdownFormat); 
```

The entire [docs.servicestack.net](http://docs.servicestack.net/docs) website is rendered using static Markdown. More information of Razor Markdown features can be found in:

  - [Introduction to Markdown Razor](https://github.com/ServiceStack/ServiceStack/wiki/Markdown-Razor)
  - [Markdown Razor Features](https://github.com/ServiceStack/ServiceStack/wiki/Markdown-Razor)
  - [How Docs is built with Markdown](http://docs.servicestack.net/markdown/about)

## Available Plugins

The rest of ServiceStack's plugins are not enabled by default by can easily be added on adhoc basis, as and when needed.

### [[Auto Query]]

AutoQuery enables instant querying support on RDBMS tables behind clean self-describing APIs by enhancing the ideal API the developer would naturally write and completing their implementation for them! This is essentially the philosophy behind AutoQuery which utilizes conventions to automate creation of intent-based self-descriptive APIs that are able to specify configurable conventions and leverage extensibility options to maximize the utility of AutoQuery services.

```csharp
Plugins.Add(new AutoQueryFeature { MaxLimit = 100 });
```

> Requires ServiceStack.Server

## [Server Events](https://github.com/ServiceStackApps/Chat#server-sent-events)

Server Events enables server push notifications to create real-time responsive web apps with its support for [Server Sent Events](http://www.html5rocks.com/en/tutorials/eventsource/basics/). It offers a number of different API's for sending notifications to select users at different levels of granularity, letting you interact and modify live-running web apps. 

```csharp
Plugins.Add(new ServerEventsFeature());
```

## [[Postman]]

The [Postman Rest Client](http://www.getpostman.com/) is a very popular and easy to use HTTP Request composer that makes it easy to call web services, similar to [Fiddler's Composer](https://www.blackbaud.com/files/support/guides/infinitydevguide/Subsystems/inwebapi-developer-help/Content/InfinityWebAPI/coUsingFiddlerCreateHTTPRequest.htm). It also provides as an alternative for auto-generating API documentation to [ServiceStack's Swagger support](https://github.com/ServiceStack/ServiceStack/wiki/Swagger-API) that makes it easier to call existing services but does require users to install the [Postman Rest Client](http://www.getpostman.com/).

```csharp
Plugins.Add(new PostmanFeature());
Plugins.Add(new CorsFeature());
```

### [Swagger support](https://github.com/ServiceStack/ServiceStack/wiki/Swagger-API)

Swagger support an optional add-on available in the [ServiceStack.Api.Swagger](https://nuget.org/packages/ServiceStack.Api.Swagger/) NuGet package.

After installing the NuGet package enable the Swagger with:

```csharp
Plugins.Add(new SwaggerFeature());
```

Now you can enjoy your shiny new Swagger UI at: `http://yoursite/swagger-ui/index.html`

#### Annotating your services

You can further document your services in the Swagger UI with the new `[Api]` and `[ApiMember]` annotation attributes, e,g: Here's an example of a fully documented service:

```csharp
[Api("Service Description")]
[Route("/swagger/{Name}", "GET", Summary = @"GET Summary", Notes = "GET Notes")]
[Route("/swagger/{Name}", "POST", Summary = @"POST Summary", Notes = "POST Notes")]
public class MyRequestDto
{
    [ApiMember(Name="Name", Description = "Name Description", 
               ParameterType = "path", DataType = "string", IsRequired = true)]
    public string Name { get; set; }
}
```

### [Razor Format](http://razor.servicestack.net/)
Provides ServiceStack's primary HTML story with support for the MVC Razor view engine.

```csharp
Plugins.Add(new RazorFormat()); 
```

It's an optional .NET 4.0 plugin that is available in the [ServiceStack.Razor](https://nuget.org/packages/ServiceStack.Razor) NuGet package.

### [Validation](https://github.com/ServiceStack/ServiceStack/wiki/Validation)
Enable the validation feature if you want to ensure all of ServiceStack's Fluent validators for Request DTOs `IValidator<TRequestDto>` are automatically validated on every request. 

```csharp
Plugins.Add(new ValidationFeature());
```

More information on ServiceStack's built-in Fluent Validation support is described on the [[Validation]] page.

### [Authentication](https://github.com/ServiceStack/ServiceStack/wiki/Authentication-and-authorization)
The Authentication Feature enables the [Authentication and Authorization](https://github.com/ServiceStack/ServiceStack/wiki/Authentication-and-authorization) support in ServiceStack. It makes available the AuthService at the default route at `/auth/{provider}`, registers **AssignRoles** and **UnAssignRoles** services (at `/assignroles` and `/unassignroles` default routes) and auto-enables Session support if it's not added already.

An example AuthFeature registration (taken from the [SocialBootstrapApi](https://github.com/ServiceStack/SocialBootstrapApi/blob/master/src/SocialBootstrapApi/App_Start/AppHost.cs#L161) project):

```csharp
Plugins.Add(new AuthFeature(
    () => new CustomUserSession(), //Use your own typed Custom UserSession type
    new IAuthProvider[] {
        new CredentialsAuthProvider(),         //HTML Form post of UserName/Password
        new TwitterAuthProvider(appSettings),  //Sign-in with Twitter
        new FacebookAuthProvider(appSettings), //Sign-in with Facebook
        new BasicAuthProvider(),               //Sign-in with Basic Auth
    }));
```

This registers and provides your ServiceStack host a myriad of different Authentication options as described above.

### [Session support](https://github.com/ServiceStack/ServiceStack/wiki/Sessions)

If you're **not** using the AuthFeature above and you still want Session support you need to enable it explicitly with:

```csharp
Plugins.Add(new SessionFeature());
```

This will add a [Request Filter](https://github.com/ServiceStack/ServiceStack/wiki/Request-and-response-filters) to instruct any HTTP client calling a ServiceStack web service to create a Temporary (ss-id) and Permanent (ss-pid) cookie if not already done so.

### Registration

Related to Authentication is Registration which enables the Registration Service at the default route `/register` which lets new Users to be registered and validated with the Credentials and Basic AuthProviders.

```csharp
Plugins.Add(new RegistrationFeature());
```

See the [SocialBootstrapApi](https://github.com/ServiceStack/SocialBootstrapApi) project for a working example of Registration and Authentication.

### [MessagePack format](https://github.com/ServiceStack/ServiceStack/wiki/MessagePack-Format)
To add fast binary [MessagePack support](https://github.com/ServiceStack/ServiceStack/wiki/MessagePack-Format) to ServiceStack install the **[ServiceStack.Plugins.MsgPack](https://nuget.org/packages/ServiceStack.Plugins.MsgPack)** NuGet package and register the plugin with:

```csharp
Plugins.Add(new MsgPackFormat());
```

### [ProtoBuf format](https://github.com/ServiceStack/ServiceStack/wiki/Protobuf-format)
To enable [ProtoBuf support](https://github.com/ServiceStack/ServiceStack/wiki/Protobuf-format) install the **[ServiceStack.Plugins.ProtoBuf](https://nuget.org/packages/ServiceStack.Plugins.ProtoBuf)** NuGet package and register the plugin with:

```csharp
Plugins.Add(new ProtoBufFormat());
```

### [Request Logger](https://github.com/ServiceStack/ServiceStack/wiki/Request-logger)

Add an In-Memory `IRequestLogger` and service with the default route at `/requestlogs` which maintains a live log of the most recent requests (and their responses). Supports multiple config options incl. Rolling-size capacity, error and session tracking, hidden request bodies for sensitive services, etc.

```csharp
Plugins.Add(new RequestLogsFeature());
```

The `IRequestLogger` is a great way to introspect and analyze your service requests in real-time. Here's a screenshot from the [http://bootstrapapi.servicestack.net](http://bootstrapapi.servicestack.net) website:

![Live Screenshot](http://mono.servicestack.net/img/request-logs-01.png)

It supports multiple queryString filters and switches so you filter out related requests for better analysis and debuggability:

![Request Logs Usage](http://mono.servicestack.net/img/request-logs-02.png)

The [RequestLogsService](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack/Admin/RequestLogsService.cs) is just a simple C# service under-the-hood but is a good example of how a little bit of code can provide a lot of value in ServiceStack's by leveraging its generic, built-in features.

## [[Encrypted Messaging]]

The Encrypted Messaging feature enables a secure channel for all Services to offer protection to clients who can now easily send and receive encrypted messages over unsecured HTTP by registering the EncryptedMessagesFeature plugin:

```csharp
Plugins.Add(new EncryptedMessagesFeature {
    PrivateKeyXml = ServerRsaPrivateKeyXml
});
```

Where `PrivateKeyXml` is the Servers RSA Private Key Serialized as XML. See the [Encrypted Messaging docs](https://github.com/ServiceStack/ServiceStack/wiki/Encrypted-Messaging) for more info.

## [[Cancellable Requests]]

The Cancellable Requests Feature makes it easy to design long-running Services that are cancellable with an external Web Service Request. To enable this feature, register the CancellableRequestsFeature plugin:

```csharp
Plugins.Add(new CancellableRequestsFeature());
```

### Web Sudo

A common UX in some websites is to add an extra layer of protection for **super protected** functionality by getting users to re-confirm their password verifying it's still them using the website, common in places like confirming a financial transaction. 

**WebSudo** is a new feature similar in spirit requiring users to re-authenticate when accessing Services annotated with the `[WebSudoRequired]` attribute. To make use of WebSudo, first register the plugin:

```csharp
Plugins.Add(new WebSudoFeature());
```

You can then apply WebSudo behavior to existing services by annotating them with `[WebSudoRequired]`:

```csharp
[WebSudoRequired]
public class RequiresWebSudoService : Service
{
    public object Any(RequiresWebSudo request)
    {
        return request;
    }
}
```

Once enabled this will throw a **402 Web Sudo Required** HTTP Error the first time the service is called:

```csharp
var requiresWebSudo = new RequiresWebSudo { Name = "test" };
try
{
    client.Send<RequiresWebSudoResponse>(requiresWebSudo); //throws
}
catch (WebServiceException)
{
    client.Send(authRequest); //re-authenticate
    var response = client.Send(requiresWebSudo); //success!
}
```

Re-authenticating afterwards will allow access to the WebSudo service.

# Community Resources

  - [Extending ServiceStack](http://morganskinner.blogspot.com/2013/04/extending-servicestack.html) by [morganskinner](http://morganskinner.blogspot.com/)