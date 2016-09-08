---
layout: default
title: Welcome to ServiceStack docs!
id: home
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

1. Getting Started
    1. [Creating your first Project](https://github.com/ServiceStack/ServiceStack/wiki/Creating-your-first-project)
      1. [Creating a WebService from scratch](https://github.com/ServiceStack/ServiceStack/wiki/Create-your-first-webservice)
    2. [[Your first webservice explained]]
    3. [Example Projects Overview](http://stackoverflow.com/a/15869816/85785)    
    4. [Learning Resources](https://github.com/ServiceStack/ServiceStack/wiki/Learning-ServiceStack)

2. Designing APIs
    1. [ServiceStack API Design](https://github.com/ServiceStack/ServiceStack/wiki/New-API)
    2. [Designing a REST-ful service with ServiceStack](http://stackoverflow.com/a/15235822/85785)
      1. [Simple Customer REST Example](http://stackoverflow.com/a/30273466/85785)
    3. [How to design a Message-Based API](http://stackoverflow.com/a/15941229/85785)
    4. [Software complexity, goals of Services and important role of DTOs](http://stackoverflow.com/a/32940275/85785)

2. Reference
    1. [[Order of Operations]]
    2. [[The IoC container]]
    3. [Configuration and AppSettings](https://github.com/ServiceStack/ServiceStack/wiki/AppSettings)
    4. [Metadata page](https://github.com/ServiceStack/ServiceStack/wiki/Metadata-page)
    5. [Rest, SOAP & default endpoints](https://github.com/ServiceStack/ServiceStack/wiki/Endpoints)
    6. [[SOAP support]]
    7. [[Routing]]
    8. [[Service return types]]
    9. [[Customize HTTP Responses]]
    10. [[Customize JSON Responses]]
    12. [[Plugins]]
    13. [Validation](https://github.com/ServiceStack/ServiceStack/wiki/Validation)
    14. [[Error Handling]]
    15. [[Security]]
    16. [[Debugging]]
    17. [JavaScript Client Library (ss-utils.js)](https://github.com/ServiceStack/ServiceStack/wiki/ss-utils.js-JavaScript-Client-Library)

3. Clients
    1. [Overview](https://github.com/ServiceStack/ServiceStack/wiki/Clients-overview)
    2. [C#/.NET client](https://github.com/ServiceStack/ServiceStack/wiki/C%23-client)
        1. [.NET Core Clients](https://github.com/ServiceStack/ServiceStack/blob/netcore/docs/pages/netcore.md)
    3. [[Add ServiceStack Reference]]
        1. [C# Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/CSharp-Add-ServiceStack-Reference)
        2. [F# Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/FSharp-Add-ServiceStack-Reference)
        3. [VB.NET Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/VB.Net-Add-ServiceStack-Reference)
        4. [Swift Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/Swift-Add-ServiceStack-Reference)
        5. [Java Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/Java-Add-ServiceStack-Reference)
    4. [Silverlight client](https://github.com/ServiceStack/ServiceStack/wiki/SilverlightServiceClient)
    5. [[JavaScript client]]
        1. [Add TypeScript Reference](https://github.com/ServiceStack/ServiceStack/wiki/TypeScript-Add-ServiceStack-Reference)
    6. [[Dart Client]]
    7. [MQ Clients](https://github.com/ServiceStack/ServiceStack/wiki/Messaging)

4. Formats
    1. [Overview](https://github.com/ServiceStack/ServiceStack/wiki/Formats)
    2. [[JSON/JSV and XML]]
    3. [HTML5 Report Format](https://github.com/ServiceStack/ServiceStack/wiki/HTML5ReportFormat)
    4. [[CSV Format]]
    5. [[MessagePack Format]]
    6. [[ProtoBuf Format]]

5. View Engines
    4. [Razor & Markdown Razor](http://razor.servicestack.net/)
      - [[Razor Notes]]
      - [[View and Template Selection]]
      - [[Compiled Razor Views]]
      - [Razor Views vs Content Pages](http://stackoverflow.com/questions/13206038/servicestack-razor-default-page/13206221#13206221)
      - [Self-Hosted Razor Views](http://www.ienablemuch.com/2012/12/self-hosting-servicestack-serving.html) 
      - [Theming Razor Views](http://www.mattjcowan.com/funcoding/2013/06/24/theming-servicestack-razor-views/)
    5. [[Markdown Razor]]

6. Hosts
    1. [[IIS]]
    2. [[Self-hosting]]
    3. [[Messaging]]
        - [[Rabbit MQ]]
        - [Redis MQ](Messaging-and-Redis)
        - [Amazon SQS](https://github.com/ServiceStack/ServiceStack.Aws#sqsmqserver)
    4. [[Mono]]
        - [[Run ServiceStack as a daemon on Linux]]
        - [Recommended guide for hosting on Mono](https://github.com/ServiceStackApps/mono-server-config)
        - [Docker image for running in a docker container](https://github.com/ServiceStackApps/mono-docker-config)
        - [[Linux-Hosting-Options]]

7. Security
    1. [Authentication and Authorization](https://github.com/ServiceStack/ServiceStack/wiki/Authentication-and-authorization)
       - [[JWT AuthProvider]]
       - [[API Key AuthProvider]]
       - [Open Id 2.0 Support](https://github.com/ServiceStack/ServiceStack/wiki/OpenId)
    2. [[Sessions]]
    3. [[Restricting Services]]
    4. [[Encrypted Messaging]]

8. Advanced
    1. [Configuration options](https://github.com/ServiceStack/ServiceStack/wiki/Configuration-options)
        - [[Run ServiceStack side-by-side with another web framework]]
    2. [[Access HTTP specific features in services]]
    3. [Logging](https://github.com/ServiceStack/ServiceStack/wiki/Logging)
    4. [[Serialization/deserialization]]
    5. [Request/response filters](https://github.com/ServiceStack/ServiceStack/wiki/Request-and-response-filters)
    6. [Filter attributes](https://github.com/ServiceStack/ServiceStack/wiki/Filter-attributes)
    7. [[Concurrency Model]]
    9. [Built-in profiling](https://github.com/ServiceStack/ServiceStack/wiki/Built-in-profiling)
    10. [[Form Hijacking Prevention]]
    11. [[Auto-Mapping]]
    12. [[HTTP Utils]]
    13. [[Dump Utils]]
    14. [[Virtual File System]]
    15. [[Config API]]
    16. [[Physical Project Structure]]
    17. [[Modularizing Services]]
    18. [[MVC Integration]]
    19. [[ServiceStack Integration]]
    20. [Embedded Native Desktop Apps](https://github.com/ServiceStack/ServiceStack.Gap)
    21. [[Auto Batched Requests]]
    22. [[Versioning]]
    23. [[Multitenancy]]

9. Caching
  1. [Caching Providers](https://github.com/ServiceStack/ServiceStack/wiki/Caching)
  2. [[HTTP Caching]]
    1. [[CacheResponse Attribute]]
    2. [[Cache Aware Clients]]

10. Auto Query
  1. [Overview](https://github.com/ServiceStack/ServiceStack/wiki/Auto-Query)
  1. [[Why Not OData]]
  2. [[AutoQuery RDBMS]]
  3. [[AutoQuery Data]]
    1. [[AutoQuery Memory]]
    2. [[AutoQuery Service]]
    3. [[AutoQuery DynamoDB]] 

11. Server Events
    1. [Overview](https://github.com/ServiceStack/ServiceStack/wiki/Server-Events)
    2. [JavaScript Client](https://github.com/ServiceStack/ServiceStack/wiki/JavaScript-Server-Events-Client)
    3. [[C# Server Events Client]]
    4. [[Redis Server Events]]

12. Service Gateway
    1. [Overview](https://github.com/ServiceStack/ServiceStack/wiki/Service-Gateway)
    2. [[Service Discovery]]

13. Encrypted Messaging
    1. [Overview](https://github.com/ServiceStack/ServiceStack/wiki/Encrypted-Messaging)
    2. [Encrypted Client](https://github.com/ServiceStack/ServiceStack/wiki/Encrypted-Messaging#encrypted-service-client)
  
14. Plugins
    3. [[Swagger API]]
    4. [[Postman]]
    5. [[Request logger]]
    6. [[Sitemaps]]
    7. [[Cancellable Requests]]
    8. [[CorsFeature]]

15. Tests
    1. [[Testing]] 
    2. [HowTo write unit/integration tests](https://github.com/ServiceStack/ServiceStack/wiki/HowTo-write-unit-integration-tests)

16. ServiceStackVS
    1. [[Install ServiceStackVS]]
    2. [[Add ServiceStack Reference]]
    3. [TypeScript React Template](https://github.com/ServiceStackApps/typescript-react-template/)
      1. [React, Redux Chat App](https://github.com/ServiceStackApps/ReactChat)
    4. [AngularJS App Template](https://github.com/ServiceStack/ServiceStackVS/blob/master/docs/angular-spa.md)
    5. [React Desktop Apps](https://github.com/ServiceStackApps/ReactDesktopApps)

17. Other Languages
    1. [[FSharp]]
        1. [Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/FSharp-Add-ServiceStack-Reference)
    2. [[VB.NET]]
        1. [Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/VB.Net-Add-ServiceStack-Reference)
    3. [[Swift]]
        1. [Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/Swift-Add-ServiceStack-Reference)
    4. [Java](https://github.com/ServiceStack/ServiceStack.Java)
        1. [Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/Java-Add-ServiceStack-Reference)
        2. [Android Studio & IntelliJ](https://github.com/ServiceStack/ServiceStack/wiki/Java-Add-ServiceStack-Reference#servicestack-idea-android-studio-plugin)
        3. [Eclipse](https://github.com/ServiceStack/ServiceStack.Java/tree/master/src/ServiceStackEclipse#eclipse-integration-with-servicestack)

18. Amazon Web Services
  1. [ServiceStack.Aws](https://github.com/ServiceStack/ServiceStack.Aws)
  2. [PocoDynamo](https://github.com/ServiceStack/PocoDynamo)
  3. [AWS Live Demos](http://awsapps.servicestack.net)
  4. [Getting Started with AWS](https://github.com/ServiceStackApps/AwsGettingStarted)

19. Deployment
    1. [[Deploy Multiple Sites to single AWS Instance]]
        1. [[Simple Deployments to AWS with WebDeploy]]
    2. [[Advanced Deployments with OctopusDeploy]]

20. Install 3rd Party Products
    1. [Redis on Windows](https://github.com/ServiceStack/redis-windows)
    2. [RabbitMQ on Windows](https://github.com/ServiceStack/rabbitmq-windows)

21. Use Cases
    1. [[Single Page Apps]] 
      1. [[HTML, CSS and JavaScript Minification]]
    2. [[Azure]]
      1. [Connecting to Azure Redis via SSL](https://github.com/ServiceStack/ServiceStack/wiki/Secure-SSL-Redis-connections-to-Azure-Redis)
    3. [[Logging]] 
    4. [[Bundling and Minification]]
    5. [[NHibernate]] 

22. Performance
    1. [[Real world performance]] 

23. Other Products
    1. [ServiceStack.Redis](https://github.com/ServiceStack/ServiceStack.Redis)
    2. [ServiceStack.OrmLite](https://github.com/ServiceStack/ServiceStack.OrmLite)
    3. [ServiceStack.Text](https://github.com/ServiceStack/ServiceStack.Text)

24. Future
    1. [[Roadmap]]