---
slug: why-servicestack
title: Why ServiceStack?
---

Developed in the modern age, Service Stack provides an alternate, cleaner POCO-driven way of creating web services.
 
- Simplicity
- Speed
- Best Practices
- Model-driven, code-first, friction-free development
- Clean, No XML config, no code-gen, conventional defaults
- Smart - Infers greater intelligence from strongly typed DTOs
- .NET 4.5 and .NET Core
- Multiple Hosts - Run in Web, Console, native Windows/OSX Desktop Apps, Windows Services
- Host Agnostic - Services are decoupled from HTTP and can be hosted in MQ
- Highly testable
- Mature - Stable with over 8+ years of development 
- Preserve Investment - ServiceStack libraries are clean, modern and [Continuously Improved](http://docs.servicestack.net/release-notes-history.html) (not abandoned or replaced)
- Commercially supported and actively developed

### [Features Overview](https://servicestack.net/features)

ServiceStack is a simple, fast, versatile and highly-productive full-featured [Web](http://razor.servicestack.net) and 
[Web Services](http://docs.servicestack.net/web-services.html) Framework that's 
thoughtfully-architected to [reduce artificial complexity](http://docs.servicestack.net/why-not-odata.html#why-not-complexity) and promote 
[remote services best-practices](http://docs.servicestack.net/advantages-of-message-based-web-services.html) 
with a [message-based design](http://docs.servicestack.net/what-is-a-message-based-web-service.html) 
that allows for maximum re-use that can leverage an integrated 
[Service Gateway](http://docs.servicestack.net/service-gateway.html) 
for the creation of loosely-coupled 
[Modularized Service](http://docs.servicestack.net/modularizing-services.html) Architectures.
ServiceStack Services are consumable via an array of built-in fast data formats (inc. 
[JSON](https://github.com/ServiceStack/ServiceStack.Text), 
XML, 
[CSV](http://docs.servicestack.net/csv-format.html), 
[JSV](http://docs.servicestack.net/json-jsv-and-xml.html), 
[ProtoBuf](http://docs.servicestack.net/protobuf-format.html), 
[Wire](http://docs.servicestack.net/wire-format.html) and 
[MsgPack](http://docs.servicestack.net/messagepack-format.html)) 
as well as XSD/WSDL for [SOAP endpoints](http://docs.servicestack.net/soap-support.html) and 
[Rabbit MQ](http://docs.servicestack.net/rabbit-mq.html), 
[Redis MQ](http://docs.servicestack.net/messaging-and-redis.html) and
[Amazon SQS](https://github.com/ServiceStack/ServiceStack.Aws#sqsmqserver) MQ hosts. 

Its design and simplicity focus offers an unparalleled suite of productivity features that can be declaratively enabled 
without code, from creating fully queryable Web API's with just a single Typed Request DTO with
[Auto Query](http://docs.servicestack.net/autoquery.html) supporting 
[every major RDBMS](https://github.com/ServiceStack/ServiceStack.OrmLite#8-flavours-of-ormlite-is-on-nuget) 
to the built-in support for
[Auto Batched Requests](http://docs.servicestack.net/auto-batched-requests.html) 
or effortlessly enabling rich [HTTP Caching](http://docs.servicestack.net/http-caching.html) and
[Encrypted Messaging](http://docs.servicestack.net/encrypted-messaging.html) 
for all your existing services via [Plugins](http://docs.servicestack.net/plugins.html).

Your same Services also serve as the Controller in ServiceStack's [Smart Razor Views](http://razor.servicestack.net/)
reducing the effort to serve both 
[Web and Single Page Apps](https://github.com/ServiceStackApps/LiveDemos) as well as 
[Rich Desktop and Mobile Clients](https://github.com/ServiceStackApps/HelloMobile) that are able to deliver instant interactive 
experiences using ServiceStack's real-time [Server Events](http://docs.servicestack.net/server-events.html).

ServiceStack Services also maximize productivity for consumers providing an 
[instant end-to-end typed API without code-gen](http://docs.servicestack.net/csharp-client.html) enabling
the most productive development experience for developing .NET to .NET Web Services.

### Define web services following Martin Fowlers Data Transfer Object Pattern

Service Stack was heavily influenced by [**Martin Fowlers Data Transfer Object Pattern**](http://martinfowler.com/eaaCatalog/dataTransferObject.html):

>When you're working with a remote interface, such as Remote Facade (388), each call to it is expensive. 
>As a result you need to reduce the number of calls, and that means that you need to transfer more data 
>with each call. One way to do this is to use lots of parameters. 
>However, this is often awkward to program - indeed, it's often impossible with languages such as Java 
>that return only a single value.
>
>The solution is to create a Data Transfer Object that can hold all the data for the call. It needs 
to be serializable to go across the connection. 
>Usually an assembler is used on the server side to transfer data between the DTO and any domain objects.

The Request- and Response DTO's used to define web services in ServiceStack are standard POCO's while 
the implementation just needs to inherit from a testable and dependency-free `IService` marker interface. 
As a bonus for keeping your DTO's in a separate dependency-free .dll, you're able to re-use them in 
your C#/.NET clients providing a strongly-typed API without any code-gen what-so-ever. Also your DTO's 
*define everything* Service Stack does not pollute your web services with any additional custom 
artefacts or markup.

Service Stack re-uses the custom artefacts above and with zero-config and without imposing any extra 
burden on the developer adds discover-ability and provides hosting of your web service on a number 
of different physical end-points which as of today includes: 
 
 - JSON
 - XML
 - CSV
 - JSV
 - MsgPack
 - ProtoBuf
 - Wire
 - SOAP 1.1/1.2

### WCF the anti-DTO Web Services Framework

Unfortunately this best-practices convention is effectively discouraged by Microsoft's WCF SOAP Web Services framework as they encourage you to develop API-specific RPC method calls by mandating the use of method signatures to define your web services API. This results in less re-usable, more client-specific APIs that encourages more remote method calls. 

Unhappy with this perceived anti-pattern in WCF, ServiceStack was born providing a Web Service framework that embraces best-practices for calling remote services, using config-free, convention-based DTO's.

### Encourages development of message-style, re-usable and batch-full web services

Entire POCO types are used to define the request- and response DTO's to promote the creation well-defined coarse-grained web services. Message-based interfaces are best-practices when dealing with out-of-process calls as they can batch more work using less network calls and are ultimately more re-usable as the same operation can be called using different calling semantics. This is in stark contrast to WCF's Operation or Service contracts which encourage RPC-style, application-specific web services by using method signatures to define each operation.

As it stands in general-purpose computing today, there is nothing more expensive you can do than a remote network call. Although easier for the newbie developer, by using _methods_ to define web service operations, WCF is promoting bad-practices by encouraging them to design and treat web-service calls like normal function calls even though they are millions of times slower. Especially at the app-server tier, nothing hurts performance and scalability of your client and server than multiple dependent and synchronous web service calls.

Batch-full, message-based web services are ideally suited in development of SOA services as they result in fewer, richer and more re-usable web services that need to be maintained. RPC-style services normally manifest themselves from a *client perspective* that is the result of the requirements of a single applications data access scenario. Single applications come and go over time while your data and services are poised to hang around for the longer term. Ideally you want to think about the definition of your web service from a *services and data perspective* and how you can expose your data so it is more re-usable by a number of your clients.

## Difference between an RPC-chatty and message-based API

```csharp
public interface IWcfCustomerService
{
    Customer GetCustomerById(int id);
    List<Customer> GetCustomerByIds(int[] id);
    Customer GetCustomerByUserName(string userName);
    List<Customer> GetCustomerByUserNames(string[] userNames);
    Customer GetCustomerByEmail(string email);
    List<Customer> GetCustomerByEmails(string[] emails);
}
```

### contrast with an equivalent message based service:

```csharp
public class Customers : IReturn<List<Customer>> 
{
   public int[] Ids { get; set; }
   public string[] UserNames { get; set; }
   public string[] Emails { get; set; }
}
```

**Any combination of the above can be fulfilled by 1 remote call, by the same single web service - i.e what ServiceStack encourages!**

Fewer and more batch-full services require less maintenance and promote the development of more re-usable and efficient services. 
In addition, message APIs are much more resilient to changes as you're able to safely add more functionality or return more data without breaking or needing to re-gen existing clients. Message-based APIs also lend them better for cached, asynchronous, deferred, proxied and reliable execution with the use of brokers and proxies.

Comparatively there is almost no win for a remote RPC API, except to maybe [hide a remote service even exists](http://en.wikipedia.org/wiki/Fallacies_of_Distributed_Computing) by making a remote call look like a method call even though they're millions of times slower, leading new developers to develop inefficient, brittle systems from the start. 

# Community Resources

  - [Interview with ServiceStack on InfoQ](http://www.infoq.com/articles/interview-servicestack)
    / [Part II](http://www.infoq.com/articles/interview-servicestack-2) by [@infoq](http://twitter.com/infoq)
  - [Great example showing how you can re-use ServiceStack services in Websites and REST APIs](http://northwind.mattjcowan.com/customers) by [@mattjcowan](https://twitter.com/mattjcowan)
  - [RESTful Data Services in .NET](https://coldie.net/?tag=servicestack)
  - [Making mPACT API Better](https://web.archive.org/web/20130401041733/http://blog.mblast.com/mbwordpress/mpact-api/making-mpact-api-better/) by [@mBLAST](https://twitter.com/mBLAST)
  - ["Service Stack" framework](http://dragansr.blogspot.com/2012/12/service-stack-framework.html) by [@dragansr](https://twitter.com/dragansr)
  - [servicestack.net – Skip WCF and use this](http://fafx.wordpress.com/category/tools/servicestack-net/) by [thefafxproject](http://fafx.wordpress.com/)
  - [Libraries and components used by DiceFeud](http://dicefeud.blogspot.se/2012/08/libraries-and-components.html) by [John Hård](http://www.blogger.com/profile/17364481663076074340)
  - [ServiceStack: a good alternative to WCF (French)](http://sgbd.arbinada.com/node/77)
  - [ReST Web Services in .NET Framework](http://www.aminemami.com/blog/2011/09/rest-web-services-in-net-framework/) by [@amin_emami](https://twitter.com/amin_emami)
  - [ServiceStack Awesomeness](http://emmanuelnelson.com/blogs/service-stack-awesomeness) by [@emmanuelnelson](http://emmanuelnelson.com/about-me)
  - [After Glow Web UI progress (ServiceStack it is!)](http://afterglow.frozenpickle.com/2013/01/02/web-ui-progress-servicestack-it-is/) by [@spazzarama](https://github.com/spazzarama)