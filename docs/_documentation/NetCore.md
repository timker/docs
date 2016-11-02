---
title: .NET Core Overview
slug: netcore
---

![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/release-notes/netcore-banner.png?t)

We've been able to bring over ServiceStack's near entire feature-set over to .NET Core which is all 
maintained within a single code-base offering high source-code compatibility to maximize existing 
knowledge and code-reuse and reducing portability efforts, all without breaking changes to existing 
.NET 4.5 Customers.

### .NET Core - the future of .NET on Linux

Whilst the development and tooling experience is still in a transitionary period we believe .NET Core puts 
.NET Web and Server App development on the cusp of an exciting future - the kind .NET hasn't seen before. 
The existing Windows hosting and VS.NET restraints have been freed, now anyone can develop using .NET's
productive expertly-designed and statically-typed mainstream C#/F# languages in their preferred editor and 
host it on the most popular server Operating Systems, in either an all-Linux, all-Windows or mixed ecosystem. 
Not only does this flexibility increase the value of existing .NET investments but it also makes .NET appeal 
to the wider and highly productive developer ecosystem who've previously disregarded .NET as an option. 

.NET Core offers significant performance and stability improvements over Mono that's derived from a shared 
cross-platform code-base and supported by a well-resourced, active and responsive team. 
If you're currently running **ServiceStack on Mono**, we strongly recommend **upgrading to .NET Core** to 
take advantage of its superior performance, stability and its top-to-bottom supported Technology Stack.

and with that let's jump into seeing some ServiceStack Live Demos running on .NET Core in Linux...

### ServiceStack .NET Core Apps running in Docker

Hosting .NET Core Apps immediately exposes us to the benefits of .NET Core. We've ported the Live Demos using 
the most productive IDE and tooling combination we've found for us - which is still VS.NET with ReSharper. 
But for deployments and hosting we now have an array of options at our disposal, including joining the 
thriving state-of-the-art ecosystem around building Linux Docker images and deploying them to the cloud. 

For .NET Core Live Demos we've settled on the popular power combo of:

 - Using [travis-ci.org](https://travis-ci.org/) (free for OSS projects) for running CI scripts to rebuild Docker App Images on every check-in
 - Using [AWS EC2 Container Service](https://aws.amazon.com/ecs/) for managing Docker images and instance deployments
 - Using [nginx-proxy](https://github.com/jwilder/nginx-proxy) setting up an nginx reverse proxy and automatically bind virtual hosts to Docker Instances

You can checkout our [Deploy .NET Core with Docker to AWS ECS Guide](/deploy-netcore-docker-aws-ecs) 
for the details on how we've deployed the .NET Core Live Demos, but ultimately packaging .NET Core Apps 
inside Docker images enables a higher-level of abstraction letting you define your entire App Server 
Instance with a repeatable recipe that lets you treat and deploy instances like opaque self-contained units.

With our [.NET 4.5 Windows Live Demos](https://github.com/ServiceStackApps/LiveDemos) we're effectively 
mutating a static Windows Server VM that required pre-configuring with IIS Virtual Hosts. Any infrastructure 
Servers each Live Demo needs, are set up out-of-band and to minimize the System administration burden, 
all Demos share the same Redis server instance.

#### Repeatable, Isolated, no-touch automated Deployments

But for our .NET Core Docker deployments we have proper isolation and repeatable no-touch deployments 
where any infrastructure services each App needs are declared in configuration and deployed in a separate 
Docker container along side each App to an ECS cluster - decoupling your deployments from static EC2 instances.
This lets you treat your server infrastructure and deployment automation story like code, where it's 
checked-in with your Repo and run with your CI who packages it in a Docker Container, publishes it as an 
opaque Image and deploys it to the [AWS EC2 Container Service](https://aws.amazon.com/ecs/).

#### Linux Cost Savings

In addition to the thriving ecosystem and superior automation, another benefit of hosting .NET Core Apps 
on Linux is the considerable cost savings of hosting on a Linux infrastructure. Docker instances enable 
isolation with considerably more efficiency than VM's allowing you to pack them with greater density. 
For .NET Core Live Demos the single T2 medium instance (**$25 /month**) is hosting 15 Docker Images whilst
running at **~50% Memory Utilization** and **<1% CPU Utilization** in its current idle state.

## .NET Core Live Demos

To showcase ServiceStack features running on .NET Core we've forked several of our existing 
[Live Demos](https://github.com/ServiceStackApps/LiveDemos) and ported them to .NET Core and 
listed them side-by-side with their original ASP.NET 4.5 code-bases so they can be easily compared.

The Live Demos cover a broad spectrum of ServiceStack features including:

<table>
<tr>
  <td colspan="2">
    <b>Redis GEO</b> - simple example showing how to make use of Redis 3.2.0 new GEO features
  </td>
</tr>
<tr>
  <td colspan="2">
    <b>Features</b> - 
    <a href="https://github.com/ServiceStack.Redis">Redis Client</a>,
    <a href="https://github.com/ServiceStackApps/redis-geo#servicestackredis-geo-apis">ServiceStack.Redis GEO APIs</a>
  </td>
</tr>
<tr>
    <td>
        <a href="http://redisgeo.netcore.io">
          <img src="https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/redis-geo/redisgeo-screenshot.png" width="350"/>
        </a>
    </td>
    <td>
        <table>
            <caption>.NET Core</caption>
            <tr>
              <td>Live Demo:</td>
              <td><a href="http://redisgeo.netcore.io">redisgeo.netcore.io</a></td>
            </tr>
            <tr>
              <td>Github:</td>
              <td><a href="https://github.com/NetCoreApps/redis-geo">github.com/NetCoreApps/redis-geo</a></td>
            </tr>
        </table>
        <table>
            <caption>ASP.NET 4.5</caption>
            <tr>
              <td>Live Demo:</td>
              <td><a href="http://redisgeo.servicestack.net">redisgeo.servicestack.net</a></td>
            </tr>
            <tr>
              <td>Github:</td>
              <td><a href="https://github.com/ServiceStackApps/redis-geo">github.com/ServiceStackApps/redis-geo</a></td>
            </tr>
        </table>  
    </td>
</tr>
<tr>
  <td colspan="2">
    <b>Chat</b> - Single Page Chat App with real-time Server Events + Simple OAuth in <190 lines of JavaScript
  </td>
</tr>
<tr>
  <td colspan="2">
    <b>Features</b> - 
    Real-time <a href="http://docs.servicestack.net/server-events.html">Server Events</a>
    and <a href="http://docs.servicestack.net/javascript-server-events-client.html">JavaScript Client</a>,
    Twitter, Facebook and Github <a href="http://docs.servicestack.net/authentication-and-authorization.html">OAuth Providers</a>
  </td>
</tr>
<tr>
    <td>
        <a href="http://chat.netcore.io">
          <img src="https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/chat.png" width="350"/>
        </a>
    </td>
    <td>
        <table>
            <caption>.NET Core</caption>
            <tr>
              <td>Live Demo:</td>
              <td><a href="http://chat.netcore.io">chat.netcore.io</a></td>
            </tr>
            <tr>
              <td>Github:</td>
              <td><a href="https://github.com/NetCoreApps/Chat">github.com/NetCoreApps/Chat</a></td>
            </tr>
        </table>        
        <table>
            <caption>ASP.NET 4.5</caption>
            <tr>
              <td>Live Demo:</td>
              <td><a href="http://chat.servicestack.net">chat.servicestack.net</a></td>
            </tr>
            <tr>
              <td>Github:</td>
              <td><a href="https://github.com/ServiceStackApps/Chat">github.com/ServiceStackApps/Chat</a></td>
            </tr>
        </table>
    </td>
</tr>
<tr>
  <td colspan="2">
    <b>SimpleAuth.Mvc</b> - Simple demo showcasing integration and authentication with ServiceStack &amp; .NET Core MVC
  </td>
</tr>
<tr>
  <td colspan="2">
    <b>Features</b> - 
    Simple <a href="http://docs.servicestack.net/authentication-and-authorization.html">Auth and OAuth Providers</a>,
    <a href="http://docs.servicestack.net/authentication-and-authorization.html#requiredrole-and-requiredpermission-attributes">Role-based Security</a>,
    <a href="http://docs.servicestack.net/servicestack-integration.html">MVC Integration</a>,
    <a href="http://docs.servicestack.net/ss-utils-js.html">ss-utils.js</a>
  </td>
</tr>
<tr>
    <td>
        <a href="http://mvc.netcore.io">
          <img src="https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/mvc.png" width="350"/>
        </a>
    </td>
    <td>
        <table>
            <caption>.NET Core</caption>
            <tr>
              <td>Live Demo:</td>
              <td><a href="http://mvc.netcore.io">mvc.netcore.io</a></td>
            </tr>
            <tr>
              <td>Github:</td>
              <td><a href="https://github.com/NetCoreApps/SimpleAuth.Mvc">github.com/NetCoreApps/SimpleAuth.Mvc</a></td>
            </tr>
        </table>        
        <table>
            <caption>ASP.NET 4.5</caption>
            <tr>
              <td>Live Demo:</td>
              <td><a href="http://mvc.servicestack.net">mvc.servicestack.net</a></td>
            </tr>
            <tr>
              <td>Github:</td>
              <td><a href="https://github.com/ServiceStack/Test/tree/master/src/Mvc">github.com/ServiceStack/Test</a></td>
            </tr>
        </table>
    </td>
</tr>
<tr>
  <td colspan="2">
    <b>Stack Apis</b> - AngularJS Single Page App leveraging AutoQuery in <50 lines of JavaScript + 1 AutoQuery DTO
  </td>
</tr>
<tr>
  <td colspan="2">
    <b>Features</b> - 
    Fully-queryable <a href="http://docs.servicestack.net/autoquery-rdbms.html">RDBMS AutoQuery Services</a>,
    <a href="https://github.com/ServiceStack/ServiceStack.OrmLite">OrmLite.Sqlite</a>,
    <a href="http://stackapis.netcore.io/ss_admin/autoquery/">AutoQuery Viewer</a>
  </td>
</tr>
<tr>
    <td>
        <a href="http://stackapis.netcore.io">
          <img src="https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/stackapis.png" width="350"/>
        </a>
    </td>
    <td>
        <table>
            <caption>.NET Core</caption>
            <tr>
              <td>Live Demo:</td>
              <td><a href="http://stackapis.netcore.io">stackapis.netcore.io</a></td>
            </tr>
            <tr>
              <td>Github:</td>
              <td><a href="https://github.com/NetCoreApps/StackApis">github.com/NetCoreApps/StackApis</a></td>
            </tr>
        </table>        
        <table>
            <caption>ASP.NET 4.5</caption>
            <tr>
              <td>Live Demo:</td>
              <td><a href="http://stackapis.servicestack.net">stackapis.servicestack.net</a></td>
            </tr>
            <tr>
              <td>Github:</td>
              <td><a href="https://github.com/ServiceStackApps/StackApis">github.com/ServiceStackApps/StackApis</a></td>
            </tr>
        </table>
    </td>
</tr>
<tr>
  <td colspan="2">
    <b>Imgur</b> - Resize uploaded images in all iOS Resolutions in <30 lines of JavaScript + 1 ServiceStack ImageService
  </td>
</tr>
<tr>
  <td colspan="2">
    <b>Features</b> - 
    File Uploads, 
    <a href="http://docs.servicestack.net/http-utils.html">HTTP Utils</a>
  </td>
</tr>
<tr>
    <td>
        <a href="http://imgur.netcore.io">
          <img src="https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/imgur.png" width="350"/>
        </a>
    </td>
    <td>
        <table>
            <caption>.NET Core</caption>
            <tr>
              <td>Live Demo:</td>
              <td><a href="http://imgur.netcore.io">imgur.netcore.io</a></td>
            </tr>
            <tr>
              <td>Github:</td>
              <td><a href="https://github.com/NetCoreApps/Imgur">github.com/NetCoreApps/Imgur</a></td>
            </tr>
        </table>        
        <table>
            <caption>ASP.NET 4.5</caption>
            <tr>
              <td>Live Demo:</td>
              <td><a href="http://imgur.servicestack.net">imgur.servicestack.net</a></td>
            </tr>
            <tr>
              <td>Github:</td>
              <td><a href="https://github.com/ServiceStackApps/Imgur">github.com/ServiceStackApps/Imgur</a></td>
            </tr>
        </table>
    </td>
</tr>
<tr>
  <td colspan="2">
    <b>Todos</b> - Backbone.js TODO App powered by a C# Redis Client back-end
  </td>
</tr>
<tr>
  <td colspan="2">
    <b>Features</b> - 
    <a href="https://github.com/ServiceStack.Redis">Redis Client</a>
  </td>
</tr>
<tr>
    <td>
        <a href="http://todos.netcore.io">
          <img src="https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/todos.png" width="350"/>
        </a>
    </td>
    <td>
        <table>
            <caption>.NET Core</caption>
            <tr>
              <td>Live Demo:</td>
              <td><a href="http://todos.netcore.io">todos.netcore.io</a></td>
            </tr>
            <tr>
              <td>Github:</td>
              <td><a href="https://github.com/NetCoreApps/Todos">github.com/NetCoreApps/Todos</a></td>
            </tr>
        </table>        
        <table>
            <caption>ASP.NET 4.5</caption>
            <tr>
              <td>Live Demo:</td>
              <td><a href="http://todos.servicestack.net">todos.servicestack.net</a></td>
            </tr>
            <tr>
              <td>Github:</td>
              <td><a href="https://github.com/ServiceStackApps/Todos">github.com/ServiceStackApps/Todos</a></td>
            </tr>
        </table>
    </td>
</tr>
<tr>
  <td colspan="2">
    <b>Razor Rockstars</b> - showcasing ServiceStack's Smart Razor Views and Pages support
  </td>
</tr>
<tr>
  <td colspan="2">
    <b>Features</b> - 
    <a href="http://razor.netcore.io">Smart Razor Pages</a>,
    <a href="https://github.com/ServiceStack/ServiceStack.OrmLite">OrmLite.Sqlite</a>,
    Markdown
  </td>
</tr>
<tr>
    <td>
        <a href="http://razor.netcore.io">
          <img src="https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/razor.png" width="350"/>
        </a>
    </td>
    <td>
        <table>
            <caption>.NET Core</caption>
            <tr>
              <td>Live Demo:</td>
              <td><a href="http://razor.netcore.io">razor.netcore.io</a></td>
            </tr>
            <tr>
              <td>Github:</td>
              <td><a href="https://github.com/NetCoreApps/RazorRockstars">github.com/NetCoreApps/RazorRockstars</a></td>
            </tr>
        </table>        
        <table>
            <caption>ASP.NET 4.5</caption>
            <tr>
              <td>Live Demo:</td>
              <td><a href="http://razor.servicestack.net">razor.servicestack.net</a></td>
            </tr>
            <tr>
              <td>Github:</td>
              <td><a href="https://github.com/ServiceStackApps/RazorRockstars">github.com/ServiceStackApps/RazorRockstars</a></td>
            </tr>
        </table>
    </td>
</tr>
<tr>
  <td colspan="2">
    <b>REST Files</b> - GitHub-like browser with remote file management over REST APIs in 1 jQuery and 1 FileService.cs
  </td>
</tr>
<tr>
  <td colspan="2">
    <b>Features</b> - 
    REST File Management and <a href="http://docs.servicestack.net/virtual-file-system.html">Virtual File System</a>
  </td>
</tr>
<tr>
    <td>
        <a href="http://restfiles.netcore.io">
          <img src="https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/restfiles.png" width="350"/>
        </a>
    </td>
    <td>
        <table>
            <caption>.NET Core</caption>
            <tr>
              <td>Live Demo:</td>
              <td><a href="http://restfiles.netcore.io">restfiles.netcore.io</a></td>
            </tr>
            <tr>
              <td>Github:</td>
              <td><a href="https://github.com/NetCoreApps/RestFiles">github.com/NetCoreApps/RestFiles</a></td>
            </tr>
        </table>        
        <table>
            <caption>ASP.NET 4.5</caption>
            <tr>
              <td>Live Demo:</td>
              <td><a href="http://restfiles.servicestack.net">restfiles.servicestack.net</a></td>
            </tr>
            <tr>
              <td>Github:</td>
              <td><a href="https://github.com/ServiceStackApps/RestFiles">github.com/ServiceStackApps/RestFiles</a></td>
            </tr>
        </table>
    </td>
</tr>
<tr>
  <td colspan="2">
    <b>Northwind</b> - database viewer, effortless data-driven, AutoQuery and Cached services with ServiceStack + OrmLite
  </td>
</tr>
<tr>
  <td colspan="2">
    <b>Features</b> - 
    <a href="http://docs.servicestack.net/autoquery-rdbms.html">RDBMS AutoQuery Services</a>,
    <a href="http://stackapis.netcore.io/ss_admin/autoquery/">AutoQuery Viewer</a>,
    <a href="http://docs.servicestack.net/http-caching.html">HTTP Caching</a> and 
    <a href="http://docs.servicestack.net/cacheresponse-attribute.html">CacheResponse attributes</a>
  </td>
</tr>
<tr>
    <td>
        <a href="http://northwind.netcore.io">
          <img src="https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/northwind.png" width="350"/>
        </a>
    </td>
    <td>
        <table>
            <caption>.NET Core</caption>
            <tr>
              <td>Live Demo:</td>
              <td><a href="http://northwind.netcore.io">northwind.netcore.io</a></td>
            </tr>
            <tr>
              <td>Github:</td>
              <td><a href="https://github.com/NetCoreApps/Northwind">github.com/NetCoreApps/Northwind</a></td>
            </tr>
        </table>        
        <table>
            <caption>ASP.NET 4.5</caption>
            <tr>
              <td>Live Demo:</td>
              <td><a href="http://northwind.servicestack.net">northwind.servicestack.net</a></td>
            </tr>
            <tr>
              <td>Github:</td>
              <td><a href="https://github.com/ServiceStackApps/Northwind">github.com/ServiceStackApps/Northwind</a></td>
            </tr>
        </table>
    </td>
</tr>
<tr>
  <td colspan="2">
    <b>Redis StackOverflow</b> - StackOverflow-like ajax website powered by ServiceStack and Redis
  </td>
</tr>
<tr>
  <td colspan="2">
    <b>Features</b> - 
    <a href="https://github.com/ServiceStack.Redis">Redis Client</a>
  </td>
</tr>
<tr>
    <td>
        <a href="http://redisstackoverflow.netcore.io">
          <img src="https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/redisstackoverflow.png" width="350"/>
        </a>
    </td>
    <td>
        <table>
            <caption>.NET Core</caption>
            <tr>
              <td>Live Demo:</td>
              <td><a href="http://redisstackoverflow.netcore.io">redisstackoverflow.netcore.io</a></td>
            </tr>
            <tr>
              <td>Github:</td>
              <td><a href="https://github.com/NetCoreApps/RedisStackOverflow">github.com/NetCoreApps/RedisStackOverflow</a></td>
            </tr>
        </table>        
        <table>
            <caption>ASP.NET 4.5</caption>
            <tr>
              <td>Live Demo:</td>
              <td><a href="http://redisstackoverflow.servicestack.net">redisstackoverflow.servicestack.net</a></td>
            </tr>
            <tr>
              <td>Github:</td>
              <td><a href="https://github.com/ServiceStackApps/RedisStackOverflow">github.com/ServiceStackApps/RedisStackOverflow</a></td>
            </tr>
        </table>
    </td>
</tr>
</table>

### Exceptional Code reuse

We're especially pleased with how little effort was required to port existing ServiceStack code-bases where 
essentially **all Service implementations were able to be reused as-is**. 

The primary changes required were due to supporting .NET Core's new project structure, conventions and bootstrapping, primarily:

 - Instead of defining NuGet packages in [packages.config](https://github.com/ServiceStackApps/redis-geo/blob/master/src/RedisGeo/RedisGeo/packages.config) 
   - They're now defined in [project.json](https://github.com/NetCoreApps/redis-geo/blob/master/src/RedisGeo/project.json) 
 - Instead of registering ServiceStack as a [HTTP Handler in Web.config](https://github.com/ServiceStackApps/redis-geo/blob/be11bd1c7ad630faf155d574fb39c25532b4ddb2/src/RedisGeo/RedisGeo/Web.config#L37) 
 and [initialized in Global.asax](https://github.com/ServiceStackApps/redis-geo/blob/be11bd1c7ad630faf155d574fb39c25532b4ddb2/src/RedisGeo/RedisGeo/Global.asax.cs#L9) 
   - It's now [configured with code](https://github.com/NetCoreApps/redis-geo/blob/4e8869fb00a5085f546539d47f119f952887a96d/src/RedisGeo/Startup.cs#L27) in .NET Core's `IApplicationBuilder` pipeline
 - Instead of configuring [Razor Namespaces and base-class in Web.config](https://github.com/ServiceStackApps/RazorRockstars/blob/f71a3b30f353f1fed1126f36c86d4d59f2235327/src/RazorRockstars.WebHost/Web.config#L28-L44)
   - It's now [configured in _ViewImports.cshtml](https://github.com/NetCoreApps/RazorRockstars/blob/master/src/RazorRockstars.WebHost/wwwroot/_ViewImports.cshtml) 
   and [_ViewStart.cshtml](https://github.com/NetCoreApps/RazorRockstars/blob/master/src/RazorRockstars.WebHost/Views/_ViewStart.cshtml)

Most of the changes ended up resulting from .NET Core's new convention of hosting Websites from the `/wwwroot` 
WebRoot Folder, requiring copying all servable static resources in `/wwwroot` and fixing all affected 
path references.

### .NET Core ServiceStack packages

We've extended the .NET Core support of the Redis and Service Clients in our last release to now include 
OrmLite (SQL Server, Sqlite, PostgreSQL), ServiceStack, MVC Integration with Razor Pages, Swagger API, 
AutoQuery, AutoQuery Viewer, Stripe, AWS Integration with PocoDynamo and SQS, RedisMq, RabbitMq, ProtoBuf 
and Wire formats:

 - ServiceStack.Text.Core
 - ServiceStack.Interfaces.Core
 - ServiceStack.Client.Core
 - ServiceStack.HttpClient.Core
 - ServiceStack.Common.Core
 - ServiceStack.Redis.Core
 - ServiceStack.OrmLite.Core
 - ServiceStack.OrmLite.Sqlite.Core
 - ServiceStack.OrmLite.SqlServer.Core
 - ServiceStack.OrmLite.PostgreSQL.Core
 - ServiceStack.Core
 - ServiceStack.Mvc.Core
 - ServiceStack.Server.Core
 - ServiceStack.ProtoBuf.Core
 - ServiceStack.Wire.Core
 - ServiceStack.Aws.Core
 - ServiceStack.RabbitMq.Core
 - ServiceStack.Stripe.Core
 - ServiceStack.Admin.Core
 - ServiceStack.Api.Swagger.Core
 - ServiceStack.Kestrel

### ServiceStack features that won't be supported in .NET Core

Whilst we were able to make most of ServiceStack's features available in .NET Core there are a number of 
features that we're not able to support, these include:

 - **HttpListener** inc. all .NET 4.5 Self Host AppHosts - replaced with .NET Core's Kestrel
 - **SOAP Support** inc. WSDLs, XSDs - missing WCF implementations in .NET Core
 - **Mini Profiler** - tightly coupled to System.Web
 - **Markdown Razor** - CodeDom not available in .NET Core (vanilla Markdown still supported)
 - **ServiceStack.Authentication.OAuth2** - DotNetOpenAuth dependency not available in .NET Core
 - **ServiceStack.Authentication.OpenId** - DotNetOpenAuth dependency not available in .NET Core
 - **MVC FluentValidation Validators** - tightly coupled to old System.Web MVC
 - **ServiceStack.Logging** - proxying to .NET Core logging abstractions instead
 - **ServiceStack.Razor** inc all existing `Html.*` helpers - tightly coupled to System.Web Razor

Whilst we lost our beloved **ServiceStack.Razor** support we developed a completely new implementation backed 
by .NET Core MVC where we were able to implement most of ServiceStack.Razor user-facing features
so porting should still be relatively straightforward with some minor syntax and configuration changes needed. 
This new implementation is available in **ServiceStack.Mvc.Core** package and can be seen in action in 
the [Razor Rockstars .NET Core demo](http://razor.netcore.io).

### Referencing .NET Core Packages

After this release we'll be bringing any remaining .NET Core compatible packages as well as focusing on
profiling and performance whilst resolving any issues reported by Customers running on .NET Core. 
In the meantime the .NET Core packages are kept isolated from .NET 4.5 packages with a `*.Core` suffix 
which enables us to make frequent .NET Core releases during the next release cycle without affecting 
existing .NET 4.5 Customers.

As we expect our latest version to be the best version of .NET Core available we recommend referencing 
ServiceStack packages using the `1.0.*` wildcard scheme, e.g:

```json
"dependencies": {
    "Microsoft.NETCore.App": {
      "version": "1.0.1",
      "type": "platform"
    },
    "ServiceStack.Core": "1.0.*",
    "ServiceStack.Redis.Core": "1.0.*",
    "ServiceStack.Common.Core": "1.0.*",
    "ServiceStack.Client.Core": "1.0.*",
    "ServiceStack.Interfaces.Core": "1.0.*",
    "ServiceStack.Text.Core": "1.0.*"
  },
}
```

This will allow your next `dotnet restore` to fetch the latest version of ServiceStack without config changes. 

#### Merging into main NuGet packages

Once we're satisfied ServiceStack .NET Core has been battle-tested in the wild we'll merge the `.Core`
packages back into the main NuGet packages and convert them to use NuGet v3 `.nuspec` format and
bump the major version to **v5.0.0**.

### AppSelfHostBase Source-compatible Self-Host

**ServiceStack.Kestrel** is a new NuGet package which encapsulates .NET Core's Kestrel HTTP Server dependency 
behind a source-compatible `AppSelfHostBase` which can be used to create source-compatible Self Hosted Apps.
This is especially valuable for creating Self Host Integration Tests that can run on both .NET 4.5 and 
.NET Core platforms. E.g. the stand-alone 
[CustomerRestExample](https://github.com/ServiceStack/ServiceStack/blob/master/tests/ServiceStack.WebHost.Endpoints.Tests/CustomerRestExample.cs)
on ServiceStack's [Project Home Page](https://github.com/ServiceStack/ServiceStack/#simple-customer-database-rest-services-example)
is run as-is on both .NET 4.5 and .NET Core without modification.

### AppHostBase .NET Core Module

Whilst `AppSelfHostBase` enables the same development experience for developing Self-Hosted ServiceStack
Solutions, when developing .NET Core-only Web Apps we instead recommend inheriting from `AppHostBase` and 
registering ServiceStack as a .NET Core module in order to remain consistent with all other .NET Core solutions. 

In ASP.NET 4.5, `AppHostBase` is used to create an ASP.NET ServiceStack Host, but in .NET Core all Web Apps 
are Console Apps, `AppHostBase` in this case just refers to a normal ServiceStack `AppHost` you'll use to 
idiomatically register your ServiceStack AppHost into .NET Core's `IApplicationBuilder` pipeline as a standard 
.NET Core Module.

### Binding to .NET Core

To see how ServiceStack integrates with .NET Core we'll walk through porting the stand-alone 
[Todos Live Demo](https://github.com/NetCoreApps/Todos) which contains the entire implementation of
a functional Todos Web App back-end in a single 
[Startup.cs](https://github.com/NetCoreApps/Todos/blob/master/src/Todos/Startup.cs) that we created using 
the **ASP.NET Core Web Application (.NET Core)** Empty VS.NET Template.

The `Program` class remains unchanged from the template and defines the entry-point for your 
Console Application that just Configures and Starts a Kestrel HTTP Server behind an IIS Reverse Proxy via 
the `AspNetCoreModule` HTTP Handler configured in your **web.config**:

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        var host = new WebHostBuilder()
            .UseKestrel()
            .UseContentRoot(Directory.GetCurrentDirectory())
            .UseIISIntegration()
            .UseStartup<Startup>()
            .Build();

        host.Run();
    }
}
```

### .NET Core Startup

The `Startup` class is what you'll use to configure your .NET App. The only difference from the default
VS.NET Template is the single line to Register your ServiceStack AppHost in .NET Core's `IApplicationBuilder`
pipeline:

```csharp
public class Startup
{
    // This method gets called by the runtime. Use this method to add services to the container.
    // For more information on how to configure your application, visit http://go.microsoft.com/fwlink/?LinkID=398940
    public void ConfigureServices(IServiceCollection services)
    {
    }

    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
    public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
    {
        loggerFactory.AddConsole();

        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.UseServiceStack(new AppHost()); //Register your ServiceStack AppHost as a .NET Core module

        app.Run(async (context) =>
        {
            await context.Response.WriteAsync("Hello World!");
        });
    }
}
```

This shows the minimum code required to configure ServiceStack to run in .NET Core. It works similarly to the 
[Wildcard HttpHandler configuration](https://github.com/ServiceStackApps/Todos/blob/fdcffd37d4ad49daa82b01b5876a9f308442db8c/src/Todos/Web.config#L37)
in your ASP.NET **Web.config** telling ASP.NET to route all requests to ServiceStack. The difference here 
is that ServiceStack is registered in a pipeline and only receives requests that weren't handled in any of
the preceding modules. Likewise ServiceStack will just call the next Module in the pipeline for any Requests 
that it's not configured to handle, this is in contrast to ASP.NET 4.5 where ServiceStack was designed to handle 
all requests it receives, so it instead returns a `NotFoundHandler` and responds with a 404 Response.

In the above configuration any Request that ServiceStack doesn't handle gets passed on to the next module 
registered, which in this case will return a `Hello World!` plain text response.

### .NET Core AppHost Integration

Once inside your `AppHost` you're back in ServiceStack-land where it's business as usual and your AppHost 
configuration remains the same as before. The only differences from 
[.NET 4.5 Todos AppHost](https://github.com/ServiceStackApps/Todos/blob/fdcffd37d4ad49daa82b01b5876a9f308442db8c/src/Todos/Global.asax.cs#L62)
is that .NET Framework introduced source-incompatible breaking changes to its Reflection APIs where instead 
of resolving a Types Assembly with `typeof(TodoService).Assembly` you're instead required to call 
`typeof(TodoService).GetTypeInfo().Assembly`. 

To combat this unfortunate design decision we've added 
[Platform Extension Methods](https://github.com/ServiceStack/ServiceStack.Text/blob/master/src/ServiceStack.Text/PlatformExtensions.cs)
providing unified Reflection APIs like `Type.GetAssembly()` allowing you to configure source-compatible
AppHost's that can be used in both .NET 4.5 and .NET Core platforms, e.g:

```csharp
// Create your ServiceStack Web Service with a singleton AppHost
public class AppHost : AppHostBase
{
    // Initializes your AppHost Instance, with the Service Name and assembly containing the Services
    public AppHost() : base("Backbone.js TODO", typeof(TodoService).GetAssembly()) { }

    // Configure your AppHost with the necessary configuration and dependencies your App needs
    public override void Configure(Container container)
    {
        //Register Redis Client Manager singleton in ServiceStack's built-in Func IOC
        container.Register<IRedisClientsManager>(new BasicRedisClientManager("localhost"));
    }
}
```

This minor change was all it took to port the 
[Todos back-end Services](https://github.com/NetCoreApps/Todos/blob/c742da45c9a70217980c2f2b323813fe7821df06/src/Todos/Startup.cs#L61-L110)
to run on .NET Core which was able to reuse the entire existing Service implementation as-is. 

The other change needed outside the Todos ServiceStack implementation was to match .NET Core's default convention 
of serving static files from the **WebRootPath** which just required moving all static resources into the 
[/wwwroot](https://github.com/NetCoreApps/Todos/tree/master/src/Todos/wwwroot) folder.

And with that the Todos port was complete, which you can view from the deployed locations below:

 - [http://todos.netcore.io](http://todos.netcore.io)       - Linux / Docker / nginx / .NET Core
 - [http://todos.servicestack.net](http://todos.servicestack.net) - Windows / IIS / .NET 4.5 / ASP.NET

## Seamless Integration with .NET Core

In addition to running flawlessly on .NET Core we're also actively striving to find how we can best 
integrate with and leverage the surrounding .NET Core ecosystem and have made several changes to that end:

### CamelCase

The JSON and JSV Text serializers are following .NET Core's default convention to use **camelCase** properties
by default. This can be reverted back to **PascalCase** with:

```csharp
SetConfig(new HostConfig { UseCamelCase = false })
```

We also agree with this default, .NET Core seems to be centered around embracing the surrounding 
developer ecosystem where .NET's default **PascalCase** protrudes in a sea of **camelCase** and 
**snake_case** JSON APIs. This won't have an impact on .NET Service Clients or Text Serialization which 
supports case-insensitive properties, however Ajax and JS clients will need to be updated to use matching properties.
You can use [ss-utils normalize()](http://docs.servicestack.net/ss-utils-js.html#normalize-and-normalizekey) methods
to help with handling both conventions by recursively normalizing and converting all properties to **lowercase**.

### .NET Core Container Adapter

Like ServiceStack, .NET Core now has a built-in IOC where you can register any dependencies you need in
your Startup `ConfigureServices()`, e.g:

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddTransient<IFoo, Foo>()
                .AddScoped<IScoped, Scoped>()
                .AddSingleton<IBar>(c => new Bar { Name = "bar" });
    }
}
```

In .NET Core ServiceStack is pre-configured to use a `NetCoreContainerAdapter` where it will also resolve any 
dependencies declared in your .NET Core Startup using `app.ApplicationServices`. One side-effect of this is that 
when resolving **Scoped** dependencies it resolves them in a Singleton scope instead of the Request Scope had 
they instead been resolved from `context.RequestServices.GetService<T>()`. 

If you need to resolve Request Scoped .NET Core dependencies you can resolve them from `IRequest`, e.g:

```csharp
public object Any(MyRequest request)
{
    var requestScope = base.Request.TryResolve<IScoped>();
}
```

Alternatively you can just register the dependencies in ServiceStack's IOC instead, e.g:

```csharp
public override void Configure(Container container)
{
    services.RegisterAutoWiredAs<Scoped,IScoped>()
        .ReusedWithin(ReuseScope.Request);
}
```

> Note: any dependencies registered .NET Core Startup are also available to ServiceStack but dependencies 
registered in ServiceStack's IOC are **only** visible to ServiceStack.

### Consistent Registration APIs

To retain the same nomenclature that .NET Core uses to register dependencies we've added several 
[Overloads to ServiceStack's IOC](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack/ContainerNetCoreExtensions.cs)
letting you use a single consistent API to register dependencies in both ServiceStack and .NET Core IOC's 
making it easy to move registrations between the two, e.g:

```csharp
public void Configure(Container container)
{
    container.AddTransient<IFoo, Foo>()
             .AddScoped<IScoped, Scoped>()
             .AddSingleton<IBar>(c => new Bar { Name = "bar" });
}
```

ServiceStack's `Container` also implements .NET Core's `IServiceProvider` interface giving it access to 
.NET Core's conveneince extension methods for resolving dependencies, e.g:

```csharp
var foo = container.GetService<IFoo>();
```

### ServiceStack.Logging Adapters

Logging is an example of an abstraction which didn't make sense to maintain a separate implementation 
as any adapter would only be able to support logging within ServiceStack. With logging you'll want to
configure it one place and have it apply to your whole application, so we've instead pre-configured 
ServiceStack.Logging to proxy all messages to .NET Core's 
[logging abstraction](https://docs.asp.net/en/latest/fundamentals/logging.html)
where you'll only need to configure logging once in `Startup` and have it handle all logging solution-wide.

### WebRootPath and ContentRootPath

Classic ASP.NET serves static resources from your Host project's folder but in .NET Core this has been moved 
to your App's `/wwwroot` folder, separated from your App's non-public assets which remain in
your projects root folder.

To match this convention we've configured the read-only `VirtualFileSources` to point to the `/wwwroot` 
**WebRootPath** whilst the read/write `IVirtualFiles` is configured to your projects **ContentRootPath** folder.

#### MapProjectPath

As ServiceStack can be hosted in variety of different platforms encompassing several AppHost's, we've
added a new `IAppHost.MapProjectPath()` API that can be used to consistently resolve an absolute file path 
using a **virtualPath** from your Host Project root folder, e.g:

```csharp
var filePath = appHost.MapProjectPath("~/path/to/settings.txt");
```

### HostContext.TryGetCurrentRequest()

ASP.NET Web Applications let you resolve the `IRequest` of the current executing HTTP Request with:

```
IRequest req = HostContext.TryGetCurrentRequest();
```

Which is a wrapper over accessing the `HttpContext.Current` singleton but as there's no equivalent in 
HttpListener self hosts this returns `null`. .NET Core also doesn't enable singleton access to the current 
HttpRequest by default but can be enabled by registering:

```csharp
services.AddSingleton<IHttpContextAccessor, HttpContextAccessor>();
```

Although this should only be registered if needed as it has 
[non-trivial performance costs](https://github.com/aspnet/Hosting/issues/793).

### OrmLite LIKE Queries

One of our primary goals with ServiceStack providers is compatibility which lets you easily switch 
providers without having to change any call-site logic using the abstraction.

One case where this has an impact on performance is in OrmLite **LIKE** queries where some RDBMS 
providers will perform case-sensitive LIKE queries so in order to retain consistent behavior across 
all RDBMS providers OrmLite generates queries using `UPPER(Field)` to ensure case-insensitive searches. 

However given .NET Core's strong focus on performance we've removed this feature and reverted to the 
behavior of the underlying RDBMS so it no longer invalidates any DB Indexes on Fields. A more efficient 
alternative to get case-insensitive LIKE queries (where it's not the default) is to use a case-insensitive 
collation.

This behavior also applies to AutoQuery LIKE queries and can be reverted with:

```csharp
OrmLiteConfig.StripUpperInLike = false;
```

### Register ServiceStack HttpHandlers as .NET Core Modules

Under the hood ServiceStack's functionality is split into different HTTP Handlers that implements 
[ASP.NET's IHttpAsyncHandler](https://msdn.microsoft.com/en-us/library/ms227433.aspx) and is also adapted to 
support HttpListener self-hosts behind ServiceStack's `IRequest` abstractions. 

We're happy to report that ServiceStack's HTTP Handlers can also be registered as a .NET Core module in 
the `IApplicationBuilder` pipeline. This lets you for instance return the same information as ServiceStack's
[?debug=requestinfo](http://docs.servicestack.net/debugging.html#request-info) route for any unhandled 
requests by registering the `RequestInfoHandler` as the last module in .NET Core's `IApplicationBuilder` 
pipeline, e.g:

```csharp
public class Startup
{
    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
    public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
    {
        loggerFactory.AddConsole();

        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.UseServiceStack(new AppHost());

        app.Use(new RequestInfoHandler());
    }
}
```

Some other examples of HTTP Handlers you could use is returning an image by registering a `StaticFileHandler`
configured with the **virtualPath** of the image (from **ContentRootPath**):

```csharp
app.Use(new StaticFileHandler("wwwroot/img/404.png"));
```

Or you can even render an MVC Razor View by returning it in a `RazorHandler`, e.g:

```csharp
app.Use(new RazorHandler("/login"));
```

Which will render the `/wwwroot/login.cshtml` Razor Page using the 
[MVC Smart Razor Pages support](/netcore-razor)

