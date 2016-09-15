---
slug: 
---
A more complete listing of ServiceStack NuGet packages is maintained at: https://servicestack.net/download

-----

# Install ServiceStack Web Service Framework via NuGet

To make it easier for developers to get started we're now maintaining NuGet packages for ServiceStack and its components.

If you have [NuGet](http://nuget.org) installed, the easiest way to get started is to install ServiceStack via NuGet:

ServiceStack with Razor Support: Create an empty ASP.NET Web or Console Application and (.NET 4.0+)
![Install-Package ServiceStack.Razor](http://mono.servicestack.net/img/nuget-servicestack.razor.png)
`Install-Package ServiceStack.Razor`

If you want just the binaries added to your web application, you just need to install
![Install-Package ServiceStack](http://mono.servicestack.net/img/nuget-servicestack.png)
`Install-Package ServiceStack`

#### Pre-configured Starter template with AppHost + Basic REST service examples

If you want to host ServiceStack Side-by-Side with MVC: Hosted at `/api` - Create an empty MVC Web Application and
![Install-Package ServiceStack.Host.Mvc](http://mono.servicestack.net/img/nuget-servicestack.host.mvc.png)
`Install-Package ServiceStack.Host.Mvc`

Otherwise if you just want ServiceStack hosted at `/` - Create an empty ASP.NET Web Application and
![Install-Package ServiceStack.Host.Aspnet](http://mono.servicestack.net/img/nuget-servicestack.host.aspnet.png)
`Install-Package ServiceStack.Host.Aspnet`

This automates the following manual steps: 

* Add the ServiceStack dlls to your standard VS.NET ASP.NET Web Application 
* Register the ServiceStack handler in your Web.Config
* Configure your AppHost 
* Create a **[Hello](http://mono.servicestack.net/ServiceStack.Hello/)** web service
* Create a **[TODO](http://mono.servicestack.net/Backbone.Todos/)** REST-ful web service

The above NuGet packages include a simple Hello World service and a REST service back-end for a simple TODO application. 

Although we believe this to be a popular starting point, it is not the only one as we have examples of Windows Services, Stand-alone Console Hosts, Hosting together with an existing web framework at a Custom Path - Templates available in the **/StarterTemplates** folder in the [ServiceStack.Examples project](https://github.com/ServiceStack/ServiceStack.Examples/tree/master/src/StarterTemplates).

## NuGet ServiceStack.Text

Downloadable separately from ServiceStack itself is its string powers. Inside [ServiceStack.Text](https://github.com/ServiceStack/ServiceStack.Text) are some of .NET's fastest Text Serializers:

* [JsonSerializer](http://mono.servicestack.net/mythz_blog/?p=344)
* [TypeSerializer (JSV-Format)](https://github.com/ServiceStack/ServiceStack.Text/wiki/JSV-Format)
* CsvSerializer
* [T.Dump extension method](http://mono.servicestack.net/mythz_blog/?p=202)
* StringExtensions - Xml/Json/Csv/Url encoding, BaseConvert, Rot13, Hex escape, etc.
* Stream, Reflection, List, DateTime, etc extensions and utils

![Install-Package ServiceStack.Text](http://mono.servicestack.net/img/nuget-servicestack.text.png)
`Install-Package ServiceStack.Text`

## NuGet ServiceStack.Redis

With a hope to introduce more .NET developers to the high-performance and productive NoSQL worlds, we also include a full-featured [C# Redis client](https://github.com/ServiceStack/ServiceStack.Redis) allowing you to build [complete apps with it](http://mono.servicestack.net/RedisStackOverflow/). [Redis](http://redis.io/) is the fastest NoSQL database in the world that is capable of achieving [about 110000 SETs and 81000 GETs per second](http://redis.io/topics/benchmarks).

The C# Redis Client features:

* High-level [Redis](https://github.com/ServiceStack/ServiceStack.Redis/wiki/IRedisClient), [RedisTypedClient](https://github.com/ServiceStack/ServiceStack.Redis/wiki/IRedisTypedClient) as well as [RedisNativeClient](https://github.com/ServiceStack/ServiceStack.Redis/wiki/IRedisNativeClient) for raw byte access.
* Thread-safe Basic and Pooled Redis clients managers
* [Creating Transactions and custom Atomic Operations](https://github.com/ServiceStack/ServiceStack.Redis/wiki/RedisTransactions)
* [Fast, efficient distributed locking with Redis](https://github.com/ServiceStack/ServiceStack.Redis/wiki/RedisLocks)
* [Publish/Subscribe messaging patterns](https://github.com/ServiceStack/ServiceStack.Redis/wiki/RedisPubSub)

For .NET developers new to Redis, we invite you to check out the following tutorials:

* [Designing a NoSQL Database using Redis](https://github.com/ServiceStack/ServiceStack.Redis/wiki/DesigningNoSqlDatabase)
* [Painless data migrations with schema-less NoSQL datastores](https://github.com/ServiceStack/ServiceStack.Redis/wiki/MigrationsUsingSchemalessNoSql)

![Install-Package ServiceStack.Redis](http://mono.servicestack.net/img/nuget-servicestack.redis.png)
`Install-Package ServiceStack.Redis`

## NuGet ServiceStack.OrmLite

When you're developing small to medium systems and you don't need the features of an advanced heavy-weight ORM, you're sometimes better off with a fast, light-weight POCO ORM. OrmLite fills this niche, where it's just a set of lightweight extension methods on .NET's ADO.NET System.Data interfaces that non-invasively works off POCO's. 

It's primary feature over other ORMs is its auto-support for blobs where any complex property is automatically persisted in a schema-less text blob using ServiceStack.Text's fast TypeSerializer. This allows you to persist most web service requests and responses directly into an RDBMS as-is without the tedious tasks of configuring tables, ORM and mapping files.

Currently OrmLite comes in 6 RDBMS's flavours and each are downloadable separately via NuGet:
[Sql Server](http://nuget.org/List/Packages/ServiceStack.OrmLite.SqlServer), [MySql](http://nuget.org/List/Packages/ServiceStack.OrmLite.MySql), [PostgreSQL](http://nuget.org/List/Packages/ServiceStack.OrmLite.PostgreSQL), [Firebird](http://nuget.org/List/Packages/ServiceStack.OrmLite.Firebird), [Sqlite32](http://nuget.org/List/Packages/ServiceStack.OrmLite.Sqlite32) and [Sqlite64](http://nuget.org/List/Packages/ServiceStack.OrmLite.Sqlite64).

![Install-Package ServiceStack.OrmLite.SqlServer](http://mono.servicestack.net/img/nuget-servicestack.ormlite.sqlserver.png)
`Install-Package ServiceStack.OrmLite.SqlServer`

![Install-Package ServiceStack.OrmLite.PostgreSQL](http://dl.dropbox.com/u/7024475/nuget-servicestack.ormlite.postgresql.png)
`Install-Package ServiceStack.OrmLite.PostgreSQL`

![Install-Package ServiceStack.OrmLite.Sqlite32](http://mono.servicestack.net/img/nuget-servicestack.ormlite.sqlite32.png)
`Install-Package ServiceStack.OrmLite.Sqlite32`

![Install-Package ServiceStack.OrmLite.Sqlite64](http://mono.servicestack.net/img/nuget-servicestack.ormlite.sqlite64.png)
`Install-Package ServiceStack.OrmLite.Sqlite64`

## Download published NuGet binaries without NuGet

GitHub has disabled its download feature so currently NuGet is the best way to get ServiceStack published releases.
For environments that don't have NuGet installed (e.g. OSX/Linux) you can still download the published binaries by 
extracting them from the published NuGet packages. The url to download a nuget package is: 

    http://packages.nuget.org/api/v1/package/{PackageName}/{Version}
    
 So to get the core ServiceStack and ServiceStack.Text libs in OSX/Linux (or using gnu tools for Windows) you can just do:

    wget -O ServiceStack http://packages.nuget.org/api/v1/package/ServiceStack/4.0.5
    unzip ServiceStack 'lib/*'
    
    wget -O ServiceStack.Text http://packages.nuget.org/api/v1/package/ServiceStack.Text/4.0.5
    unzip ServiceStack.Text 'lib/*'

which will download and extract the dlls into your local local `lib/` folder.