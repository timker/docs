---
layout: default
---
## Where can I ask questions?

- **[StackOverflow](http://stackoverflow.com/search?q=servicestack)** - Use `servicestack` tag for questions on StackOverflow
  - Support options for ServiceStack are listed in your account page: https://servicestack.net/account/support
- **[ServiceStack G+ Community](https://plus.google.com/communities/112445368900682590445)** Get in touch with the ServiceStack Community
- **[Live Demos!](https://github.com/ServiceStackApps/LiveDemos)**

## Resources
- [Release Notes](https://servicestack.net/release-notes)
- [[Architecture Overview]]
- [Official ServiceStack website](http://www.servicestack.net/)
- [Feature Requests](http://servicestack.uservoice.com/forums/176786-feature-requests)
- [Documentation for v3](https://github.com/ServiceStackV3/ServiceStackV3)

***

- **[ServiceStack's packages available on NuGet](https://servicestack.net/download)**
  - [Pre-release packages available on MyGet](https://github.com/ServiceStack/ServiceStack/wiki/MyGet)

***
- **[[Contributors]]** Special thanks to all of ServiceStack's Contributors
- **[[Community-Resources]]** (Blog Posts, Gists, StackOverflow, Pod casts...)
- [Contributing Guide](https://github.com/ServiceStack/ServiceStack/wiki/Contributing) 

# Documentation
_This is the documentation about the core web service framework. Please see the individual projects for documentation on the [Redis client](https://github.com/ServiceStack/ServiceStack.Redis), [OrmLite](https://github.com/ServiceStack/ServiceStack.OrmLite) or [Text Serializers](https://github.com/ServiceStack/ServiceStack.Text)._

> [[Prerequisites]] - Before you start reading, you should know the basics about HTTP (HTTP methods/verbs, status codes etc) and REST/SOAP

  - [Why ServiceStack?](https://github.com/ServiceStack/ServiceStack/wiki/Why-Servicestack)
  - [What is a message based web service?](https://github.com/ServiceStack/ServiceStack/wiki/What-is-a-message-based-web-service)
  - [[Advantages of message based web services]]
  - [Why remote services should use separate Data Transfer Objects](http://stackoverflow.com/a/15369736/85785)

- Getting Started
    - [Creating your first Project](https://github.com/ServiceStack/ServiceStack/wiki/Creating-your-first-project)
      - [Creating a WebService from scratch](https://github.com/ServiceStack/ServiceStack/wiki/Create-your-first-webservice)
    - [[Your first webservice explained]]
    - [Example Projects Overview](http://stackoverflow.com/a/15869816/85785)    
    - [Learning Resources](https://github.com/ServiceStack/ServiceStack/wiki/Learning-ServiceStack)

- Designing APIs
    - [ServiceStack API Design](https://github.com/ServiceStack/ServiceStack/wiki/New-API)
    - [Designing a REST-ful service with ServiceStack](http://stackoverflow.com/a/15235822/85785)
      - [Simple Customer REST Example](http://stackoverflow.com/a/30273466/85785)
    - [How to design a Message-Based API](http://stackoverflow.com/a/15941229/85785)
    - [Software complexity, goals of Services and important role of DTOs](http://stackoverflow.com/a/32940275/85785)

- Reference
    - [[Order of Operations]]
    - [[The IoC container]]
    - [Configuration and AppSettings](https://github.com/ServiceStack/ServiceStack/wiki/AppSettings)
    - [Metadata page](https://github.com/ServiceStack/ServiceStack/wiki/Metadata-page)
    - [Rest, SOAP & default endpoints](https://github.com/ServiceStack/ServiceStack/wiki/Endpoints)
    - [[SOAP support]]
    - [[Routing]]
    - [[Service return types]]
    - [[Customize HTTP Responses]]
    - [[Customize JSON Responses]]
    - [[Plugins]]
    - [Validation](https://github.com/ServiceStack/ServiceStack/wiki/Validation)
    - [[Error Handling]]
    - [[Security]]
    - [[Debugging]]
    - [JavaScript Client Library (ss-utils.js)](https://github.com/ServiceStack/ServiceStack/wiki/ss-utils.js-JavaScript-Client-Library)

- Clients
    - [Overview](https://github.com/ServiceStack/ServiceStack/wiki/Clients-overview)
    - [C#/.NET client](https://github.com/ServiceStack/ServiceStack/wiki/C%23-client)
        - [.NET Core Clients](https://github.com/ServiceStack/ServiceStack/blob/netcore/docs/pages/netcore.md)
    - [[Add ServiceStack Reference]]
        - [C# Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/CSharp-Add-ServiceStack-Reference)
        - [F# Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/FSharp-Add-ServiceStack-Reference)
        - [VB.NET Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/VB.Net-Add-ServiceStack-Reference)
        - [Swift Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/Swift-Add-ServiceStack-Reference)
        - [Java Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/Java-Add-ServiceStack-Reference)
    - [Silverlight client](https://github.com/ServiceStack/ServiceStack/wiki/SilverlightServiceClient)
    - [[JavaScript client]]
        - [Add TypeScript Reference](https://github.com/ServiceStack/ServiceStack/wiki/TypeScript-Add-ServiceStack-Reference)
    - [[Dart Client]]
    - [MQ Clients](https://github.com/ServiceStack/ServiceStack/wiki/Messaging)

- Formats
    - [Overview](https://github.com/ServiceStack/ServiceStack/wiki/Formats)
    - [JSON/JSV and XML](?id=json-jsv-and-xml)
    - [HTML5 Report Format](https://github.com/ServiceStack/ServiceStack/wiki/HTML5ReportFormat)
    - [[CSV Format]]
    - [[MessagePack Format]]
    - [[ProtoBuf Format]]

- View Engines
    - [Razor & Markdown Razor](http://razor.servicestack.net/)
      - [[Razor Notes]]
      - [[View and Template Selection]]
      - [[Compiled Razor Views]]
      - [Razor Views vs Content Pages](http://stackoverflow.com/questions/13206038/servicestack-razor-default-page/13206221#13206221)
      - [Self-Hosted Razor Views](http://www.ienablemuch.com/2012/12/self-hosting-servicestack-serving.html) 
      - [Theming Razor Views](http://www.mattjcowan.com/funcoding/2013/06/24/theming-servicestack-razor-views/)
    - [[Markdown Razor]]

- Hosts
    - [[IIS]]
    - [[Self-hosting]]
    - [[Messaging]]
        - [[Rabbit MQ]]
        - [Redis MQ](?id=messaging-and-redis)
        - [Amazon SQS](https://github.com/ServiceStack/ServiceStack.Aws#sqsmqserver)
    - [[Mono]]
        - [[Run ServiceStack as a daemon on Linux]]
        - [Recommended guide for hosting on Mono](https://github.com/ServiceStackApps/mono-server-config)
        - [Docker image for running in a docker container](https://github.com/ServiceStackApps/mono-docker-config)
        - [[Linux-Hosting-Options]]

- Security
    - [Authentication and Authorization](https://github.com/ServiceStack/ServiceStack/wiki/Authentication-and-authorization)
       - [[JWT AuthProvider]]
       - [[API Key AuthProvider]]
       - [Open Id 2.0 Support](https://github.com/ServiceStack/ServiceStack/wiki/OpenId)
    - [[Sessions]]
    - [[Restricting Services]]
    - [[Encrypted Messaging]]

- Advanced
    - [Configuration options](https://github.com/ServiceStack/ServiceStack/wiki/Configuration-options)
        - [[Run ServiceStack side-by-side with another web framework]]
    - [[Access HTTP specific features in services]]
    - [Logging](https://github.com/ServiceStack/ServiceStack/wiki/Logging)
    - [Serialization/deserialization](?id=serialization-deserialization)
    - [Request/response filters](https://github.com/ServiceStack/ServiceStack/wiki/Request-and-response-filters)
    - [Filter attributes](https://github.com/ServiceStack/ServiceStack/wiki/Filter-attributes)
    - [[Concurrency Model]]
    - [Built-in profiling](https://github.com/ServiceStack/ServiceStack/wiki/Built-in-profiling)
    - [[Form Hijacking Prevention]]
    - [[Auto-Mapping]]
    - [[HTTP Utils]]
    - [[Dump Utils]]
    - [[Virtual File System]]
    - [[Config API]]
    - [[Physical Project Structure]]
    - [[Modularizing Services]]
    - [[MVC Integration]]
    - [[ServiceStack Integration]]
    - [Embedded Native Desktop Apps](https://github.com/ServiceStack/ServiceStack.Gap)
    - [[Auto Batched Requests]]
    - [[Versioning]]
    - [[Multitenancy]]

- Caching
  - [Caching Providers](https://github.com/ServiceStack/ServiceStack/wiki/Caching)
  - [[HTTP Caching]]
    - [[CacheResponse Attribute]]
    - [[Cache Aware Clients]]

- Auto Query
  - [Overview](https://github.com/ServiceStack/ServiceStack/wiki/Auto-Query)
  - [[Why Not OData]]
  - [[AutoQuery RDBMS]]
  - [[AutoQuery Data]]
    - [[AutoQuery Memory]]
    - [[AutoQuery Service]]
    - [[AutoQuery DynamoDB]] 

- Server Events
    - [Overview](https://github.com/ServiceStack/ServiceStack/wiki/Server-Events)
    - [JavaScript Client](https://github.com/ServiceStack/ServiceStack/wiki/JavaScript-Server-Events-Client)
    - [C# Server Events Client](?id=csharp-server-events-client)
    - [[Redis Server Events]]

- Service Gateway
    - [Overview](https://github.com/ServiceStack/ServiceStack/wiki/Service-Gateway)
    - [[Service Discovery]]

- Encrypted Messaging
    - [Overview](https://github.com/ServiceStack/ServiceStack/wiki/Encrypted-Messaging)
    - [Encrypted Client](https://github.com/ServiceStack/ServiceStack/wiki/Encrypted-Messaging#encrypted-service-client)
  
- Plugins
    - [[Swagger API]]
    - [[Postman]]
    - [[Request logger]]
    - [[Sitemaps]]
    - [[Cancellable Requests]]
    - [[CorsFeature]]

- Tests
    - [[Testing]] 
    - [HowTo write unit/integration tests](https://github.com/ServiceStack/ServiceStack/wiki/HowTo-write-unit-integration-tests)

- ServiceStackVS
    - [[Install ServiceStackVS]]
    - [[Add ServiceStack Reference]]
    - [TypeScript React Template](https://github.com/ServiceStackApps/typescript-react-template/)
      - [React, Redux Chat App](https://github.com/ServiceStackApps/ReactChat)
    - [AngularJS App Template](https://github.com/ServiceStack/ServiceStackVS/blob/master/docs/angular-spa.md)
    - [React Desktop Apps](https://github.com/ServiceStackApps/ReactDesktopApps)

- Other Languages
    - [[FSharp]]
        - [Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/FSharp-Add-ServiceStack-Reference)
    - [[VB.NET]]
        - [Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/VB.Net-Add-ServiceStack-Reference)
    - [[Swift]]
        - [Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/Swift-Add-ServiceStack-Reference)
    - [Java](https://github.com/ServiceStack/ServiceStack.Java)
        - [Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/Java-Add-ServiceStack-Reference)
        - [Android Studio & IntelliJ](https://github.com/ServiceStack/ServiceStack/wiki/Java-Add-ServiceStack-Reference#servicestack-idea-android-studio-plugin)
        - [Eclipse](https://github.com/ServiceStack/ServiceStack.Java/tree/master/src/ServiceStackEclipse#eclipse-integration-with-servicestack)

- Amazon Web Services
  - [ServiceStack.Aws](https://github.com/ServiceStack/ServiceStack.Aws)
  - [PocoDynamo](https://github.com/ServiceStack/PocoDynamo)
  - [AWS Live Demos](http://awsapps.servicestack.net)
  - [Getting Started with AWS](https://github.com/ServiceStackApps/AwsGettingStarted)

- Deployment
    - [[Deploy Multiple Sites to single AWS Instance]]
        - [[Simple Deployments to AWS with WebDeploy]]
    - [[Advanced Deployments with OctopusDeploy]]

- Install 3rd Party Products
    - [Redis on Windows](https://github.com/ServiceStack/redis-windows)
    - [RabbitMQ on Windows](https://github.com/ServiceStack/rabbitmq-windows)

- Use Cases
    - [[Single Page Apps]] 
      - [[HTML, CSS and JavaScript Minification]]
    - [[Azure]]
      - [Connecting to Azure Redis via SSL](https://github.com/ServiceStack/ServiceStack/wiki/Secure-SSL-Redis-connections-to-Azure-Redis)
    - [[Logging]] 
    - [[Bundling and Minification]]
    - [[NHibernate]] 

- Performance
    - [[Real world performance]] 

- Other Products
    - [ServiceStack.Redis](https://github.com/ServiceStack/ServiceStack.Redis)
    - [ServiceStack.OrmLite](https://github.com/ServiceStack/ServiceStack.OrmLite)
    - [ServiceStack.Text](https://github.com/ServiceStack/ServiceStack.Text)

- Future
    - [[Roadmap]]