---
slug: caching
title: Caching Providers
---

As caching is an essential technology in the development of high-performance web services, Service Stack has a number of different caching options available that each share the same
[common client interface (ICacheClient)](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/Caching/ICacheClient.cs)
for the following cache providers:

  * [Redis](https://github.com/ServiceStack/ServiceStack.Redis) - A very fast key-value store that has  non-volatile persistent storage and support for rich data structures such as lists and sets.
  * [OrmLiteCacheClient](https://www.nuget.org/packages/ServiceStack.Server) - Supports all [OrmLite's RDBMS providers](https://github.com/ServiceStack/ServiceStack.OrmLite/#download) for using an existing RDBMS as a distributed cache.
  * [Memcached](https://nuget.org/packages/ServiceStack.Caching.Memcached) - The tried and tested most widely used cache provider.
  * [In Memory Cache](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack/Caching/MemoryCacheClient.cs) - Useful for single host web services and enabling unit tests to run without needing access to a cache server.
  * [Aws DynamoDB Cache Client](https://www.nuget.org/packages/ServiceStack.Aws/) - For using with Amazon's Dynamo DB backend hosted on Amazon Web Services
  * [Azure Cache Client](https://nuget.org/packages/ServiceStack.Caching.Azure) - For using the Azure DataCache client when your application is hosted on Azure.

To configure which cache should be used, the particular client has to be registered in the IoC container:

#### In-memory cache:
```csharp 
container.Register<ICacheClient>(new MemoryCacheClient());
```

##### NuGet Package: [ServiceStack](http://www.nuget.org/packages/ServiceStack)

#### Redis

```csharp 
container.Register<IRedisClientsManager>(c => 
    new PooledRedisClientManager("localhost:6379"));

container.Register(c => c.Resolve<IRedisClientsManager>().GetCacheClient());
```

##### NuGet Package: [ServiceStack.Redis](http://www.nuget.org/packages/ServiceStack.Redis)

#### OrmLite

```csharp 
//Register OrmLite Db Factory if not already
container.Register<IDbConnectionFactory>(c => 
    new OrmLiteConnectionFactory(connString, SqlServerDialect.Provider)); 

container.RegisterAs<OrmLiteCacheClient, ICacheClient>();

//Create 'CacheEntry' RDBMS table if it doesn't exist already
container.Resolve<ICacheClient>().InitSchema(); 
``` 

#### SQL Server Memory Optimized Cache

SQL Server's Memory Optimized support can be used to improve the performance of `OrmLiteCacheClient` 
by configuring it to use the above In Memory Table Schema instead, e.g:

```csharp
container.Register<ICacheClient>(c => 
    new OrmLiteCacheClient<SqlServerMemoryOptimizedCacheEntry>());
```

##### NuGet Package: [ServiceStack.Server](http://www.nuget.org/packages/ServiceStack.Server)

#### Memcached:
```csharp 
container.Register<ICacheClient>(
    new MemcachedClientCache(new[] { "127.0.0.0" }); //Add Memcached hosts
```

##### NuGet Package: [ServiceStack.Caching.Memcached](http://www.nuget.org/packages/ServiceStack.Caching.Memcached)

#### AWS DynamoDB:

```csharp
var awsDb = new AmazonDynamoDBClient(
    AWS_ACCESS_KEY, AWS_SECRET_KEY, RegionEndpoint.USEast1);
var cache = new DynamoDbCacheClient(new PocoDynamo(awsDb));
cache.InitSchema();
```
##### NuGet Package: [ServiceStack.Aws](http://www.nuget.org/packages/ServiceStack.Aws)

#### Azure:
```csharp 
container.Register<ICacheClient>(
    new AzureCacheClient("MyAppCache")); //Add your Azure CacheName if any
```

##### NuGet Package: [ServiceStack.Caching.Azure](http://www.nuget.org/packages/ServiceStack.Caching.Azure)

## Cache a response of a service

To cache a response you simply have to call `ToOptimizedResultUsingCache` which is an extension method existing in `ServiceStack.ServiceHost`.

In your service:

```csharp
public class OrdersService : Service
{
    public object Get(CachedOrders request)
    {
        var cacheKey = "unique_key_for_this_request";
        return base.Request.ToOptimizedResultUsingCache(base.Cache,cacheKey,()=> 
            {
                //Delegate is executed if item doesn't exist in cache 
                //Any response DTO returned here will be cached automatically
            });
    }
}
```

> Tip: There exists a class named [UrnId](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Common/UrnId.cs) which provides helper methods to create unique keys for an object.

`ToOptimizedResultUsingCache` also has an overload which provides a parameter to set the timespan when the cache should be deleted (marked as expired). If now a client calls the same service method a second time and the cache expired, the provided delegate, which returns the response DTO, will be executed a second time.

```csharp
var cacheKey = "some_unique_key";
//Cache should be deleted in 1h
var expireInTimespan = new TimeSpan(1, 0, 0);
return base.Request.ToOptimizedResultUsingCache(
    base.Cache, cacheKey, expireInTimespan, ...)
```

## Delete cached responses

If now for example an order gets updated and the order was cached before the update, the webservice will still return the same result, because the cache doesn't know that the order has been updated.

So there are two options:

- Use **time based** caching (and expire cache earlier)
- Cache on **validility** 

> When the cache is based on **validility** the caches are invalidated manually (e.g. when a user modified his profile, > clear his cache) which means you always get the latest version and you never need to hit the database again to rehydrate the cache if it hasn't changed, which will save resources.

So if the order gets updated, you should delete the cache manually:

```csharp
public class CachedOrdersService : Service
{
    public object Put(CachedOrders request)
    {
        //The order gets updated...
        var cacheKey = "some_unique_key_for_order";
        return base.Request.RemoveFromCache(base.Cache, cacheKey);
    }
}
```

If now the client calls the webservice to request the order, he'll get the latest version.

### Local MemoryCacheClient

As it sometimes beneficial to have access to a local in-memory Cache in addition to your registered `ICacheClient` 
[Caching Provider](/caching)
we also pre-register a `MemoryCacheClient` that all your Services now have access to from the `LocalCache` 
property, i.e:

```csharp
    MemoryCacheClient LocalCache { get; }
```

This doesn't affect any existing functionality that utilizes a cache like Sessions which continue to use
your registered `ICacheClient`, but it does let you change which cache you want different responses to use, e.g: 

```csharp
var cacheKey = "unique_key_for_this_request";
return base.Request.ToOptimizedResultUsingCache(LocalCache, cacheKey, () => {
    //Delegate is executed if item doesn't exist in cache 
});
```

If you don't register a `ICacheClient` ServiceStack automatically registers a `MemoryCacheClient` for you 
which will also refer to the same instance registered for `LocalCache`.

## [ICacheClientExtended](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/Caching/ICacheClientExtended.cs)

The [ICacheClientExtended](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/Caching/ICacheClientExtended.cs)
API is used to to provide additional non-core functionality to our most popular 
[Caching providers](/caching):

 - Redis
 - OrmLite RDBMS
 - In Memory
 - AWS
 
The new API's are added as Extension methods on `ICacheClient` so they're easily accessible without casting, the new API's available include: 
  
  - GetKeysByPattern(pattern) - return keys matching a wildcard pattern
  - GetAllKeys() - return all keys in the caching provider
  - GetKeysStartingWith() - Streaming API to return all keys Starting with a prefix

With these new API's you can now easily get all active User Sessions using any of the supported Caching providers above with:

```csharp
var sessionPattern = IdUtils.CreateUrn<IAuthSession>(""); //= urn:iauthsession:
var sessionKeys = Cache.GetKeysStartingWith(sessionPattern).ToList();

var allSessions = Cache.GetAll<IAuthSession>(sessionKeys);
```

## Live Example and code

A live demo of the ICacheClient is available in [The ServiceStack.Northwind's example project](http://northwind.servicestack.net/). Here are some requests to cached services:

  * [/customers](http://northwind.servicestack.net/cached/customers)
  * [/customers/ALFKI](http://northwind.servicestack.net/cached/customers/ALFKI)
  * [/customers/ALFKI/orders](http://northwind.servicestack.net/cached/customers/ALFKI/orders)

Which are simply existing web services wrapped using **ICacheClient** that are contained in [CachedServices.cs](https://github.com/ServiceStack/ServiceStack.Examples/blob/master/src/ServiceStack.Northwind/ServiceStack.Northwind.ServiceInterface/CachedServices.cs)