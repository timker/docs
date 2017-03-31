---
slug: release-notes-summary
title: Release Notes Summary
---

> [Release Notes History](/release-notes-history)

# [v4.5.8 Release Notes](/releases/v4.5.8)

We've got a another feature-packed release with a number of exciting new features  new support for the [Open API specification](https://github.com/OAI/OpenAPI-Specification) 
enabling new integration possibilities with 
[Azure API Management](https://azure.microsoft.com/en-us/services/api-management/)
and [Azure AutoRest](https://github.com/Azure/autorest), our real-time [Server Events](/server-events) 
solution extending to new Mobile, Desktop and Server platforms, more flexible Authentication options to 
dramatically simplify integration with Mobile, Desktop and Web clients, effortless JWT Refresh Tokens, 
enhanced support for TypeScript and Java client libraries, new compression options, upgrades to the latest 
Fluent Validation with support for Async validators, Async Request Filters, a number of new quality 
OSS ServiceStack projects developed by the community and lots more use-case driven features and 
refinements across the entire ServiceStack suite!

Please see the [v4.5.8 Release Notes](/releases/v4.5.8) for the full details, for a quick summary we'll highlight the major features below:

## [Open API](/openapi)

We've added support for the Open API Specification which opens up several new integration possibilities
including importing your Services into Azure API Management and generating native clients with 
Azure's AutoRest. The Open API plugin also embeds Swagger UI letting you quickly explore and interact with your services.

## [Server Events](/server-events)

We've made great strides in this release towards our goal of expanding the Server Event ecosystem in popular 
Mobile, Desktop and Server platforms with new first-class implementations for Android, Java and TypeScript which now includes:

 - [C# Server Events Client](/csharp-server-events-client)
    - Xamarin.iOS
    - Xamarin.Android
    - UWP
    - .NET Framework 4.5+
    - .NET Core (.NET Standard 1.3+)
- [TypeScript Server Events Client](/typescript-server-events-client)
    - Web
    - Node.js Server
    - React Native
        - iOS
        - Android
- [Java Server Events Client](/java-server-events-client)
    - Android
    - JVM 1.7+ (Java, Kotlin, Scala, etc)
        - Java Clients
        - Java Servers
- [JavaScript (jQuery plugin)](/javascript-server-events-client)
    - Web

## [Android Java Chat](https://github.com/ServiceStackApps/AndroidJavaChat)

To showcase real-world usage of `AndroidServerEventsClient` in an Android App we've ported 
[C# Xamarin Android Chat](https://github.com/ServiceStackApps/AndroidXamarinChat) into **Java 8** using 
Google's recommended [Android Studio Development Environment](https://developer.android.com/studio/index.html). 
In addition to retaining the same functionality as the original 
[C# Xamarin.Android Chat App](https://github.com/ServiceStackApps/AndroidXamarinChat), it also leverages the native 
Facebook, Twitter and Google SDK's to enable seamless and persistent authentication when signing in with Facebook, 
Twitter or Google User accounts.

[![](https://raw.githubusercontent.com/ServiceStack/docs/master/docs/images/java/java-android-chat-screenshot-auth.png)](https://github.com/ServiceStackApps/AndroidJavaChat/)

## [Web, Node.js and React Native ServerEvents Apps](https://github.com/ServiceStackApps/typescript-server-events)

The TypeScript [servicestack-client](https://github.com/ServiceStack/servicestack-client) npm package is a 
cross-platform library enabling a rich, productive end-to-end Typed development experience on Web, node.js Server 
projects, node.js test suites, React Native iOS and Android Mobile Apps - written in either TypeScript or 
plain JavaScript.

To help getting started using [servicestack-client](https://github.com/ServiceStack/servicestack-client) in each of 
JavaScript's most popular platforms we've developed a simple Server Events Web, Node.js and React Native Mobile iOS App
that can connect to any Server Events Server and listen to messages published on the subscribed channel. The App also 
maintains a live synchronized list of Users in the channel that's automatically updated whenever Users join or leave 
the channel:

[![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/typescript-serverevents/typescript-server-events-banner.png)](https://github.com/ServiceStackApps/typescript-server-events)

Each App uses the optimal starting configuration that enables a productive workflow on each platform and uses the minimum
runtime dependencies - essentially just [servicestack-client](https://github.com/ServiceStack/servicestack-client) 
and its es6-shim and W3C `EventSource` polyfills on Node.js and React Native where it's missing a native implementation.

## TypeScript servicestack-client improvements

New APIs have been added to TypeScript's [servicestack-client](https://github.com/ServiceStack/servicestack-client) 
to catch up with the additional flexibility and features available in [C#/.NET Service Clients](/csharp-client):

### Sending additional arguments with Typed API Requests

Many AutoQuery Services utilize [implicit conventions](/autoquery-rdbms#implicit-conventions) to query fields that 
aren't explicitly defined on AutoQuery Request DTOs, these can now be queried by specifying additional arguments with 
the typed Request DTO, e.g:

```ts
const request = new FindTechStacks();

client.get(request, { VendorName: "ServiceStack" })
    .then(r => { }) // typed to QueryResponse<TechnologyStack> 
```

### Calling APIs with Custom URLs

In addition to Typed API Requests you can now also call Services using relative or absolute urls, e.g:

```ts
client.get<GetTechnologyResponse>("/technology/ServiceStack")

client.get<GetTechnologyResponse>("http://techstacks.io/technology/ServiceStack")

// GET http://techstacks.io/technology?Slug=ServiceStack
client.get<GetTechnologyResponse>("/technology", { Slug: "ServiceStack" }) 
```

## Relaxed TypeScript Definitions

The metadata requirements in TypeScript Ambient Interface definitions generated by Add ServiceStack Reference's `/types/typescript.d` have been relaxed so they can be used to Type Object Literals:

```ts
let request:QueryContracts = { accountId: 1234 };
```

But as metadata in 
Request DTO interface definitions are no longer inferrable, their return Type and route will need to be supplied 
on each call-site, e.g:

```ts
let request:FindTechnologies = { vendorName: "ServiceStack" };

client.get<QueryResponse<Technology>>("/technology/search", request)
    .then(r => { }); //typed response
```

## Authentication via OAuth AccessTokens 

To improve OAuth Sign In integration from native Mobile or Desktop Apps we've added support for direct 
Authentication via AccessTokens which can dramatically simplify the Development and User Experience by 
being able to leverage the Native Facebook, Twitter and Google Client SDK's to Sign In users locally
then reuse their local **AccessToken** to Authenticate with back-end ServiceStack Servers. 
This feature is what's used to enable [Integrated Facebook, Twitter and Google Logins](https://github.com/ServiceStackApps/AndroidJavaChat/#integrated-facebook-twitter-and-google-logins)
in Android Java Chat and be able to [Automatically Sign In users with saved AccessTokens](https://github.com/ServiceStackApps/AndroidJavaChat#automatically-sign-in-previously-signed-in-users).

This capability is now available on the popular OAuth Providers below:

- `FacebookAuthProvider` - Sign in with Facebook
- `TwitterAuthProvider` - Sign in with Twitter
- `GithubAuthProvider` - Sign in with Github
- `GoogleOAuth2Provider` - Sign in with Google

### Client Authentication with AccessToken

Clients can utilize this feature with the new `AccessToken` and `AccessTokenSecret` properties on the existing
`Authenticate` request DTO, sent with the **provider** that the AccessToken is for, e.g:

```csharp
var response = client.Post(new Authenticate {
    provider = "facebook",
    AccessToken = facebookAccessToken,
    RememberMe = true,
});
```

## JWT Refresh Tokens

Support for Refresh Tokens is now available in the [JWT Auth Provider](/jwt-authprovider) where new **JWT Tokens**
can now be generated from a longer-lived `RefreshToken` which can be used to fetch new JWT Tokens without
needing to re-authenticate with a separate Auth Provider.

### Accessing Refresh Tokens

Just like JWT Tokens, Refresh Tokens are populated on the `AuthenticateResponse` DTO after successfully 
authenticating via any registered Auth Provider, e.g:

```csharp
var response = client.Post(new Authenticate {
    provider = "credentials",
    UserName = userName,
    Password = password,
});

var jwtToken = response.BearerToken;
var refreshToken = response.RefreshToken;
```

### Lifetimes of tokens

The default expiry time of JWT and Refresh Tokens below can be overridden when registering the `JwtAuthProvider`:

```csharp
new JwtAuthProvider {
    ExpireTokensIn        = TimeSpan.FromDays(14),  // JWT Token Expiry
    ExpireRefreshTokensIn = TimeSpan.FromDays(365), // Refresh Token Expiry
}
```

### Using JWT and Refresh Tokens

In order to provide the simplest development experience possible you only need to specify the Refresh Token on the Service Client and it will take care of transparently fetching a new JWT Token behind-the-scenes. You don't even need to configure the client with a JWT Token as it will just fetch a new one on first use, e.g:

```csharp
var client = new JsonServiceClient(baseUrl) {
    RefreshToken = refreshToken,
};

var response = client.Send(new Secured());
```

## Fluent Validation Upgraded

Another major feature added in this release was contributed by the community with 
[Scott Mackay](https://github.com/wwwlicious) upgrading our internal Fluent Validation implementation 
to the latest version of [FluentValidation](https://github.com/JeremySkinner/FluentValidation). Special care
was taken to maintain backwards compatibility where ServiceStack enhancements were retrofitted on top 
of the new version and existing Error Codes were preserved to ensure minimal disruption with existing
code bases.

### Async Validators

One of the benefits from using the latest version of Fluent Validation is we now have support for Async Validators! 
Async validators can be registered using the `MustAsync` validator where you could simulate the following built-in 
**Not Empty** validation:

```csharp
public class MyRequestValidator : AbstractValidator<MyRequest>
{
    public MyRequestValidator()
    {
        RuleFor(x => x.Name).NotEmpty();
    }
}
```

And replace it with an Async version that uses the [Service Gateway](/service-gateway) to 
call a custom Async `GetStringLength` Service that returns the same `ErrorCode` and Error Message as the 
**Not Empty** validator:

```csharp
public class MyRequestValidator : AbstractValidator<MyRequest>
{
    public MyRequestValidator()
    {
        RuleFor(x => x.Name).MustAsync(async (s, token) => 
            (await Gateway.SendAsync(new GetStringLength { Value = s })).Result > 0)
        .WithMessage("'Name' should not be empty.")
        .WithErrorCode("NotEmpty");
    }
}
```

## Async Global Request Filters

To properly implement Async Validators we also needed Async Request Filters which were also added in this release 
where you can now register non-blocking Request Filters with:

```csharp
GlobalRequestFiltersAsync.Add(async (req,res,dto) => {
    var response = await client.Send(new CheckRateLimit { 
        Service = dto.GetType().Name,
        IpAddress = req.UserHostAddress,
     });
     if (response.RateLimitExceeded) 
     {
         res.StatusCode = 403;
         res.StatusDescription = "RateLimitExceeded";
         res.EndRequest();
     }
})
```

## Enhanced Compression Options

We've followed up [Request Compression](/releases/v4.5.6#clientserver-request-compression) added in the last release
with more compression features in this release including:

### `[CompressResponse]` Attribute

You can now selectively choose which Services should be compressed with the new `[CompressResponse]` attribute to 
compress responses for clients which support compression, which can be applied to most Response Types, e.g:

```csharp
[CompressResponse]
public class CompressedServices : Service
{
    public object Any(CompressDto request) => new CompressExamplesResponse(); 
    public object Any(CompressString request) => "foo"; 
    public object Any(CompressBytes request) => "foo".ToUtf8Bytes(); 
    public object Any(CompressStream request) => new MemoryStream("foo".ToUtf8Bytes()); 
    public object Any(CompressFile request) => new HttpResult(VirtualFileSources.GetFile("/foo"));

    public object Any(CompressAnyHttpResult request)
    {
        return new HttpResult(new CompressExamplesResponse());    // DTO
        return new HttpResult("foo", "text/plain");               // string
        return new HttpResult("foo".ToUtf8Bytes(), "text/plain"); // bytes
        //etc
    }
}
```

> Note using `[CompressResponse]` is unnecessary when returning [cached responses](/http-caching) as ServiceStack 
automatically caches and returns the most optimal Response - typically compressed bytes for clients that 
supports compression

### Static File Compression

ServiceStack can also be configured to compress static files with specific file extensions that are larger than 
specific size with the new opt-in Config options below:

```csharp
SetConfig(new HostConfig {
    CompressFilesWithExtensions = { "js", "css" },
    // (optional), only compress .js or .css files > 10k
    CompressFilesLargerThanBytes = 10 * 1024 
});
```

When more fine-grained logic is needed you can override `ShouldCompressFile()` in your AppHost to choose which
static files you want to compress on a per-file basis, e.g:

```csharp
public override bool ShouldCompressFile(IVirtualFile file)
{
    return base.ShouldCompressFile(file) || file.Name == "large.csv";
}
```

## Community

We've been lucky to receive a number of quality contributions from the Community in this release 
starting with the new [Serilog](https://serilog.net) Logging provider contributed by [Josh Engler](https://github.com/englerj):

### ServiceStack.Logging.Serilog

To Configure Serilog Logging, first download [ServiceStack.Logging.Serilog](https://www.nuget.org/packages/ServiceStack.Logging.Serilog) from NuGet:

    PM> Install-Package ServiceStack.Logging.Serilog

Then configure ServiceStack to use `SerilogFactory`:

```csharp
LogManager.LogFactory =  new SerilogFactory();
```

## Community Projects

There's also been a number of quality OSS Community projects that's surfaced recently that you may find useful to
enable in your ServiceStack projects:

### [ServiceStack.Webhooks](https://github.com/jezzsantos/ServiceStack.Webhooks)

If you need to implement a Webhooks solution for your Services you'll definitely want to check out 
[ServiceStack.Webhooks](https://github.com/jezzsantos/ServiceStack.Webhooks) by [@jezzsantos](https://github.com/jezzsantos). ServiceStack.Webhooks is a high-quality, actively developed, 
[well documented](https://github.com/jezzsantos/ServiceStack.Webhooks/wiki) solution for raising and managing 
application-level "events" raised by your services:

![](https://raw.githubusercontent.com/jezzsantos/ServiceStack.Webhooks/master/docs/images/Webhooks.Architecture.PNG)

### [ServiceStack.Authentication.Azure](https://github.com/ticky74/ServiceStack.Authentication.Azure)

[ServiceStack.Authentication.Azure](https://github.com/ticky74/ServiceStack.Authentication.Azure) is a fork of
[ServiceStack.Authentication.Aad](https://github.com/jfoshee/ServiceStack.Authentication.Aad) developed by 
[Ian Kulmatycki](https://github.com/ticky74) which provides an `AzureAuthenticationProvider` for easily 
authenticating users with Office365 and hybrid Azure Active Directories.

### [ServiceStack.Authentication.Marten](https://github.com/migajek/ServiceStack.Authentication.Marten)

[ServiceStack.Authentication.Marten](https://github.com/migajek/ServiceStack.Authentication.Marten) is a 
`UserAuthRepository` repository developed by [Michał Gajek](https://github.com/migajek) for persisting users in 
[Marten](http://jasperfx.github.io/marten/getting_started/) - a Document Database for PostgreSQL.

### MultiAppSettingsBuilder

Another feature contributed by [Josh Engler](https://github.com/englerj) is the `MultiAppSettingsBuilder` which adds
a fluent discoverable API for configuring ServiceStack's various [App Setting Providers](/appsettings), e.g:

```csharp
AppSettings = new MultiAppSettingsBuilder()
    .AddAppSettings()
    .AddDictionarySettings(new Dictionary<string,string> { "override" : "setting" })
    .AddEnvironmentalVariables()
    .AddTextFile("~/path/to/settings.txt".MapProjectPath())
    .Build();
```

### CacheClient with Prefix

The `CacheClientWithPrefix` class contributed by [@joelharkes](https://github.com/joelharkes) lets you decorate 
any `ICacheClient` to prefix all cache keys using the `.WithPrefix()` extension method. This could be used to 
easily enable multi-tenant usage of a single redis instance, e.g:

```csharp
container.Register(c => 
    c.Resolve<IRedisClientsManager>().GetCacheClient().WithPrefix("site1"));
```

### OrmLite Soft Deletes

Select Filters were added to OrmLite to let you specify a custom `SelectFilter` that lets you modify queries 
that use `SqlExpression<T>` before they're executed. This could be used to make working with "Soft Deletes" 
Tables easier where it can be made to apply a custom `x.IsDeleted != true` condition on every `SqlExpression`.

By either using a `SelectFilter` on concrete POCO Table Types, e.g:

```csharp
SqlExpression<Table1>.SelectFilter = q => q.Where(x => x.IsDeleted != true);
SqlExpression<Table2>.SelectFilter = q => q.Where(x => x.IsDeleted != true);
```

Or alternatively you can configure a global `SqlExpressionSelectFilter` with:

```csharp
OrmLiteConfig.SqlExpressionSelectFilter = q =>
{
    if (q.ModelDef.ModelType.HasInterface(typeof(ISoftDelete)))
    {
        q.Where<ISoftDelete>(x => x.IsDeleted != true);
    }
};
```

Both solutions above will transparently add the `x.IsDeleted != true` to all `SqlExpression<T>` based queries
so it only returns results which aren't `IsDeleted` from any of queries below:

```csharp
var results = db.Select(db.From<Table>());
var result = db.Single(db.From<Table>().Where(x => x.Name == "foo"));
var result = db.Single(x => x.Name == "foo");
```

## ServiceStack.Text 

You can easily escape HTML Entities characters (`<`, `>`, `&`, `=`, `'`) when serializing JSON strings with:

```csharp
JsConfig.EscapeHtmlChars = true;
```

This can also be requested by clients using [Customized JSON Responses](/customize-json-responses), e.g:

    /my-api?jsconfig=EscapeHtmlChars

## Rabbit MQ

A strong-named version of ServiceStack.RabbitMq is now available at:

    PM> Install-Package ServiceStack.RabbitMq.Signed

The new `CreateQueueFilter` and `CreateTopicFilter` filters added to the [Rabbit MQ Server](/rabbit-mq) will let 
you customize what options Rabbit MQ Queue's and topics are created with.

## Startup Errors

To better highlight the presence of Startup Errors we're now adding a red warning banner in `/metadata` 
pages when in [DebugMode](/debugging#debugmode), e.g:

![](/images/release-notes/startup-errors.png)

The number of Startup Errors is also added to the `X-Startup-Errors: n` Global HTTP Header so you'll be 
able to notice it when debugging HTTP Traffic.

### Forcing a Content Type

Whilst ServiceStack Services are typically available on any endpoint and format, there are times when you only 
want adhoc Services available in a particular format, for instance you may only want the View Models for your
dynamic Web Views available in HTML. This can now be easily enabled with the new `[HtmlOnly]` Request Filter 
Attribute, e.g:
    
```cshtml
[HtmlOnly]
public class HtmlServices : Service
{
    public object Any(MyRequest request) => new MyViewModel { .. };
}
```

This feature is also available for other built-in Content Types: `[JsonOnly]`, `[XmlOnly]`, `[JsvOnly]` and `[CsvOnly]`.

### AuthProvider available on IAuthSession

You can now determine what Auth Provider was used to populate a User's Session with the new 
`IAuthSession.AuthProvider` property. This could be used for instance to detect and ensure highly-sensitive 
services are only available to users that authenticated using a specific Auth Provider.

## All .NET Core Examples upgraded to .NET Core 1.1

All .NET Core Live examples including the **redis-geo** project used in 
[Deploy .NET Core with Docker to AWS ECS Guide](/deploy-netcore-docker-aws-ecs) 
were upgraded to .NET Core 1.1 and has been changed to use the `microsoft/dotnet:1.1-sdk-projectjson` 
Docker Image since `microsoft/dotnet:latest` was changed to only support projects with VS 2017 `.csproj` 
msbuild format.

This covers a high-level view of the major features added in this release, please see the 
[v4.5.8 Release Notes](/releases/v4.5.8) for the full details and other features added in this release.

# [v4.5.6 Release Notes](/releases/v4.5.6)

For the full details of this release please see the [full v4.5.6 release notes](http://docs.servicestack.net/releases/v4.5.6).

## New Angular2 Single Page App template!

We've added a new modern SPA VS.NET Template for Angular2 which is built the same npm-based TypeScript / JSPM / Gulp technology stack that's solidified in our other SPA templates with the main difference being that it's based on the *Material Design Lite* theme.

The Angular2 template also takes advantage of Angular2's modular architecture with a physical structure optimal for small-to-medium sized projects where its modular layout is compartmentalized into multiple independent sub modules which can easily scale to support large code bases. 

## Simpler and Optimized Single Page App Templates

We've also simplified all our existing npm-based SPA Templates to take advantage of the latest dependencies which can simplify our existing development workflow, some changes include:

### Upgraded to JSPM 0.17 beta

One of the benefits of using JSPM 0.17 beta is we're now using its built-in static builds with Rollup optimizations for production deployments which statically links your entire App's JavaScript into a single `app.js`, eliminating the need for `system.js` at runtime and removing the packaging overhead from using modules.

### Removed interim deps.tsx

The interim `deps.tsx` file used to minimize the number of requests required during development is no longer needed. We're now able to generate a cache of 3rd party npm dependencies using your App's main .js file and your dependencies listed in npm's `package.json`. 

### Simplified Typings

The typings dependency manager has been removed leaving one less `typings.json` that needs to be maintained. Templates now use TypeScript's new @types definitions directly from npm or when they exist, the definitions contained in each npm package that's referenced in *devDependencies*. 

### Upgraded to latest Bootstrap v4

The React and Aurelia SPA Templates have been upgraded to use the just released Bootstrap v4 alpha-6.

## Enhanced TypeScript Support

The SPA Templates also benefit from our enhanced TypeScript support with improvements to both the generated TypeScript DTOs and the `JsonServiceClient` which now includes TypeScript Definitions published with the npm package.

 - *Support for Basic Auth* - Basic Auth support is now implemented in `JsonServiceClient` and follows the same API made available in the C# Service Clients
 - *Raw Data Responses* - The `JsonServiceClient` also supports Raw Data responses like `string` and `byte[]`

## Swift 3

Swift Add ServiceStack Reference and the Swift `JsonServiceClient` has been upgraded to Swift 3 including
its embedded PromiseKit implementation. The earlier Swift 2 compiler bugs preventing AutoQuery Services from working have now been resolved. 

### [swiftref OSX command-line utility](https://github.com/ServiceStack/swiftref)

In response to XcodeGhost, Apple has killed support for plugins in Xcode 8 disabling all existing 3rd party 
plugins from working, and along with it our Integration with Xcode built into ServiceStack Xcode Plugin. 

To enable the best development experience we can without using an Xcode plugin we've developed the 
swiftref OSX command-line utility to provide a simple command-line UX to easily Add and Update Swift 
ServiceStack References.

Installation and Usage instructions for swiftref are available from: https://github.com/ServiceStack/swiftref

### All Swift Example Apps upgraded to Swift 3

Our existing Swift Example Apps have all been upgraded to use ServiceStack's new Swift 3 Support:

 - [TechStacks iOS App](https://github.com/ServiceStackApps/TechStacksApp) 
 - [TechStacks Desktop Cocoa OSX App](https://github.com/ServiceStackApps/TechStacksDesktopApp)
 - [AutoQuery Viewer iOS App](https://github.com/ServiceStackApps/AutoQueryViewer)
 - [101 Swift LINQ Samples](https://github.com/mythz/swift-linq-examples)

### [Swift Package Manager Apps](https://github.com/ServiceStack/SwiftClient)

In its quest to become a popular mainstream language, Swift now includes a built-in Package Manager 
to simplify the maintenance, distribution and building of Swift code. Swift Package Manager can be used to
build native statically-linked modules or Console Apps but currently has no support for iOS, watchOS, 
or tvOS platforms. 

Nevertheless it's simple console and text-based programming model provides a great way to quickly develop 
prototypes or Console-based Swift Apps like swiftref using your favorite text editor. To support this 
environment we've packaged ServiceStack's Swift Service clients into a *ServiceStackClient* package 
so it can be easily referenced in Swift PM projects.

A step-by-step guide showing how to build Swift PM Apps is available at: https://github.com/ServiceStackApps/swift-techstacks-console

### [Service Fabric Example](https://github.com/ServiceStackApps/HelloServiceFabric)

Another Example project developed during this release is [Hello ServiceFabric](https://github.com/ServiceStackApps/HelloServiceFabric) to show a Hello World example of running a ServiceStack Self Hosted Service inside Microsoft's Service Fabric platform.

### Performance Improvements

Our first support for .NET Core was primarily focused on compatibility and deep integration with .NET Core's 
Pipeline, Modules and Conventions. In this release we focused on performance and memory usage which we kicked 
off by developing a benchmarking solution for automatically spinning up Azure VM's that we use to run 
benchmarking and load tests against: https://github.com/NetCoreApps/Benchmarking

A simple JSON Service running our initial .NET Core support in v4.5.4 release yields *Requests/Sec Average*:

 - .NET Core 1.1 **26179**
 - mono 4.6.2 (nginx+hyperfasctcgi) **6428**

Running on Standard_F4s azure instance (4 Core, 8GB RAM) using the following 
[wrk benchmarking tool](https://github.com/wg/wrk) command:

    wrk -c 256 -t 8 -d 30 http://benchmarking_url

Using **8 threads**, keeping **256 concurrent HTTP Connections** open, Running for **30s**.

With the profiling and performance improvements added in this release, the same service now yields:

 - .NET Core 1.1 **37073**
 - mono 4.6.2 (nginx+hyperfasctcgi) **6840**

Which is over a **40% improvement** for this benchmark since last release. Running the same ServiceStack 
Service on the same VM also shows us that .NET Core is **5.4x faster** than Mono.

### Upgraded to .NET Core 1.1

We're closely following .NET Core's progress and continue to upgrade ServiceStack libraries and their test 
suites to run on the latest stable .NET Core 1.1 release.

### Client/Server Request Compression

You can now also elect to compress HTTP Requests in any C#/.NET Service Clients by specifying the Compression 
Type you wish to use, e.g:

```csharp
var client = new JsonServiceClient(baseUrl) {
    RequestCompressionType = CompressionTypes.GZip,
};
```

### FileSystem Mapping

Custom FileSystem mappings can now be easily registered under a specific alias by overriding your AppHost's 
`GetVirtualFileSources()` and registering a custom `FileSystemMapping`, e.g:

```csharp
public override List<IVirtualPathProvider> GetVirtualFileSources()
{
    var existingProviders = base.GetVirtualFileSources();
    existingProviders.Add(new FileSystemMapping(this, "img", "i:\\images"));
    existingProviders.Add(new FileSystemMapping(this, "docs", "d:\\documents"));
    return existingProviders;
}
```

This will let you access File System Resources under the custom `/img` and `/doc` routes, e.g:

 - http://host/img/the-image.jpg
 - http://host/docs/word.doc

## OrmLite

### SQL Server Features

Kevin Howard has continued enhancing the SQL Server Support in OrmLite with access to advanced SQL Server features including Memory-Optimized Tables where you can tell SQL Server to maintain specific tables in Memory using the `[SqlServerMemoryOptimized]` attribute, e.g:

```csharp
[SqlServerMemoryOptimized(SqlServerDurability.SchemaOnly)]
public class SqlServerMemoryOptimizedCacheEntry : ICacheEntry
{
    [PrimaryKey]
    [StringLength(StringLengthAttribute.MaxText)]
    [SqlServerBucketCount(10000000)]
    public string Id { get; set; }
    [StringLength(StringLengthAttribute.MaxText)]
    public string Data { get; set; }
    public DateTime CreatedDate { get; set; }
    public DateTime? ExpiryDate { get; set; }
    public DateTime ModifiedDate { get; set; }
}
```

The new Memory Optimized support can be used to improve the performance of SQL Server `OrmLiteCacheClient` by configuring it to use the above In Memory Table Schema instead, e.g:

```csharp
container.Register<ICacheClient>(c => 
    new OrmLiteCacheClient<SqlServerMemoryOptimizedCacheEntry>());
```

### PostgreSQL Data Types

To make it a little nicer to be define custom PostgreSQL columns, we've added `[PgSql*]` specific attributes which will let you use a typed `[PgSqlJson]` instead of previously needing to use `[CustomField("json")]`. 

The list of PostgreSQL Attributes include:

```csharp
public class MyPostgreSqlTable
{
    [PgSqlJson]
    public List<Poco> AsJson { get; set; }

    [PgSqlJsonB]
    public List<Poco> AsJsonB { get; set; }

    [PgSqlTextArray]
    public string[] AsTextArray { get; set; }

    [PgSqlIntArray]
    public int[] AsIntArray { get; set; }

    [PgSqlBigIntArray]
    public long[] AsLongArray { get; set; }
}
```

### .NET Core support for MySql

You can now use OrmLite with MySQL in .NET Core using the new [ServiceStack.OrmLite.MySql.Core](https://www.nuget.org/packages/ServiceStack.OrmLite.MySql.Core) NuGet package.

### Create Tables without Foreign Keys

You can temporarily disable and tell OrmLite to create tables without Foreign Keys by setting `OrmLiteConfig.SkipForeignKeys = true`.

### Custom SqlExpression Filter

The generated SQL from a Typed `SqlExpression` can now be customized using the new `.WithSqlFilter()`, e.g:

```csharp
var q = db.From<Table>()
    .Where(x => x.Age == 27)
    .WithSqlFilter(sql => sql + " option (recompile)");

var q = db.From<Table>()
    .Where(x => x.Age == 27)
    .WithSqlFilter(sql => sql + " WITH UPDLOCK");

var results = db.Select(q);
```

### Custom SQL Fragments

The new `Sql.Custom()` API lets you use raw SQL Fragments in Custom `.Select()` expressions, e.g:

```csharp
var q = db.From<Table>()
    .Select(x => new {
        FirstName = x.FirstName,
        LastName = x.LastName,
        Initials = Sql.Custom("CONCAT(LEFT(FirstName,1), LEFT(LastName,1))")
    });
```

### Cached API Key Sessions

You can reduce the number of I/O Requests and improve the performance of API Key Auth Provider Requests by specifying a `SessionCacheDuration` to temporarily store the Authenticated UserSession against the API Key which will reduce subsequent API Key requests down to 1 DB call to fetch and validate the API Key + 1 Cache Hit to restore the User's Session which if you're using the default in-memory Cache will mean it only requires 1 I/O call for the DB request. This can be enabled with:

```csharp
Plugins.Add(new AuthFeature(...,
    new IAuthProvider[] {
        new ApiKeyAuthProvider(AppSettings) {
            SessionCacheDuration = TimeSpan.FromMinutes(10),
        }
    }));
```

That covers the major features, for the full details please see the full release notes at: http://docs.servicestack.net/releases/v4.5.6

# [2016 Release Notes Summary](/releases/2016-summary)