---
layout: default
title: ServiceStack Resources
---

- [StackOverflow](http://stackoverflow.com/search?q=servicestack) - Ask questions on StackOverflow using the `servicestack` tag
- [Customer Forums](https://forums.servicestack.net/) - Customers can get Technical Support in the Customer Forums
- [Issue Tracker](https://github.com/ServiceStack/Issues) - Report issues on our Issue Tracker
- [Feature Requests](http://servicestack.uservoice.com/forums/176786-feature-requests) - Request features on UserVoice
- [Live Demos](https://github.com/ServiceStackApps/LiveDemos) - Checkout Live Demos and source code created with ServiceStack
- [Gistlyn](http://gistlyn.com) - Explore ServiceStack Live on Gistlyn
- [Release Notes](https://servicestack.net/release-notes) - Stay up to date with ServiceStack's latest features
- [servicestack.net](http://www.servicestack.net/) - See ServiceStack's Official Website for product info and pricing 

## ServiceStack Downloads

- [ServiceStack Downloads](https://servicestack.net/download) - Official downloads are available on NuGet
- [Pre-release MyGet Packages](/myget) - The latest downloads in-between each release are available on MyGet

## ServiceStack Community

- [ServiceStack G+ Community](https://plus.google.com/communities/112445368900682590445) Get in touch with the ServiceStack Community
- [Community-Resources](/community-resources) (Blog Posts, Gists, StackOverflow, Pod casts...)
- [Contributors](/contributors) - Special thanks to all of ServiceStack's Contributors
- [Contributing Guide](/contributing) - Read our Contributing Guide before contributing

## ServiceStack Documentation

> [Prerequisites](/prerequisites) - Before you start reading, you should know the basics about HTTP (HTTP methods/verbs, status codes etc) and REST/SOAP

- [Why ServiceStack?](/why-servicestack)
- [Architecture Overview](/architecture-overview)
- [What is a message-based WebService?](/what-is-a-message-based-web-service)
- [Advantages of message-based WebServices](/advantages-of-message-based-web-services)
- [Why remote services should use DTOs?](http://stackoverflow.com/a/15369736/85785)

- Getting Started
    - [Create your first WebService](/create-your-first-webservice)
      - [Creating a WebService from scratch](/create-webservice-from-scratch)
    - [Your first webservice explained](/your-first-webservice-explained)
    - [Example Projects Overview](http://stackoverflow.com/a/15869816/85785)    
    - [Learning Resources](/learning-servicestack)

- Designing APIs
    - [ServiceStack's API Design](/api-design)
    - [Designing a REST-ful service with ServiceStack](http://stackoverflow.com/a/15235822/85785)
      - [Simple Customer REST Example](http://stackoverflow.com/a/30273466/85785)
    - [How to design a Message-Based API](http://stackoverflow.com/a/15941229/85785)
    - [Software complexity, goals of Services and important role of DTOs](http://stackoverflow.com/a/32940275/85785)

- Reference
    - [Order of Operations](?/order-of-operations)
    - [The IoC container](/ioc)
    - [Configuration & AppSettings](/appsettings)
    - [Metadata page](/metadata-page)
    - [REST, SOAP & default endpoints](/endpoints)
    - [SOAP support](/soap-support)
    - [Routing](/routing)
    - [Service return types](/service-return-types)
    - [Customize HTTP Responses](/customize-http-responses)
    - [Customize JSON Responses](/customize-json-responses)
    - [Plugins](/plugins)
    - [Validation](/validation)
    - [Error Handling](/error-handling)
    - [Security](/security)
    - [Debugging](/debugging)
    - [JavaScript Client Library (ss-utils.js)](/ss-utils-js)

- Clients
    - [Overview](/clients-overview)
    - [C#/.NET client](/csharp-client)
        - [.NET Core Clients](https://github.com/ServiceStack/ServiceStack/blob/netcore/docs/pages/netcore.md)
    - [Add ServiceStack Reference](/add-servicestack-reference)
        - [C# Add Reference](/csharp-add-servicestack-reference)
        - [TypeScript Add Reference](/typescript-add-servicestack-reference)
        - [Swift Add Reference](/swift-add-servicestack-reference)
        - [Java Add Reference](/java-add-servicestack-reference)
        - [Kotlin Add Reference](/kotlin-add-servicestack-reference)
        - [F# Add Reference](/fsharp-add-servicestack-reference)
        - [VB.NET Add Reference](/vbnet-add-servicestack-reference)
    - [JavaScript client](/javascript-client)
    - [Silverlight client](/silverlight-client)
    - [Dart Client](/dart-client)
    - [MQ Clients](/messaging#mq-client-architecture)

- Formats
    - [Overview](/formats)
    - [JSON Foramt](https://github.com/ServiceStack/ServiceStack.Text)
    - [JSV Foramt](/jsv-format)
    - [CSV Format](/csv-format)
    - [MessagePack Format](/messagepack-format)
    - [ProtoBuf Format](/protobuf-format)
    - [HTML5 Report Format](/html5reportformat)

- View Engines
    - [Razor & Markdown Razor](http://razor.servicestack.net/)
    - [Razor Notes](/razor-notes)
    - [View and Template Selection](/view-and-template-selection)
      - [Compiled Razor Views](/compiled-razor-views)
      - [Razor Views vs Content Pages](http://stackoverflow.com/questions/13206038/servicestack-razor-default-page/13206221#13206221)
      - [Self-Hosted Razor Views](http://www.ienablemuch.com/2012/12/self-hosting-servicestack-serving.html) 
      - [Theming Razor Views](http://www.mattjcowan.com/funcoding/2013/06/24/theming-servicestack-razor-views/)
    - [Markdown Razor](/markdown-razor)

- Hosts
    - [IIS](/iis)
    - [Self-hosting](/self-hosting)
    - [Messaging](/messaging)
        - [Rabbit MQ](/rabbit-mq)
        - [Rabbit MQ](/rabbit-mq)
        - [Amazon SQS](https://github.com/ServiceStack/ServiceStack.Aws#sqsmqserver)
    - [Mono](/mono)
        - [Run ServiceStack as a daemon on Linux](/servicestack-as-daemon-on-linux)
        - [Recommended guide for hosting on Mono](https://github.com/ServiceStackApps/mono-server-config)
        - [Docker image for running in a docker container](https://github.com/ServiceStackApps/mono-docker-config)
    - [Linux-Hosting-Options](/linux-hosting-options)

- Security
    - [Authentication & Authorization](/authentication-and-authorization)
        - [JWT AuthProvider](/jwt-authprovider)
        - [API Key AuthProvider](/api-key-authprovider)
        - [Open Id 2.0 Support](/openId)
    - [Sessions](/sessions)
    - [Restricting Services](/restricting-services)
    - [Encrypted Messaging](/encrypted-messaging)

- Advanced
    - [Configuration options](/configuration-options)
        - [Run ServiceStack side-by-side with another web framework](/servicestack-side-by-side-with-another-web-framework)
    - [Access HTTP specific features in services](/access-http-specific-features-in-services)
    - [Logging](/logging)
    - [Serialization & Deserialization](/serialization-deserialization)
    - [Request & Response filters](?/request-and-response-filters)
    - [Filter attributes](/filter-attributes)
    - [Concurrency Model](/concurrency-model)
    - [Built-in profiling](/built-in-profiling)
    - [Form Hijacking Prevention](/form-hijacking-prevention)
    - [Auto-Mapping](/auto-mapping)
    - [HTTP Utils](/http-utils)
    - [Dump Utils](/dump-utils)
    - [Virtual File System](/virtual-file-system)
    - [Config API](/config-api)
    - [Physical Project Structure](/physical-project-structure)
    - [Modularizing Services](/modularizing-services)
    - [MVC Integration](/mvc-integration)
    - [ServiceStack Integration](/servicestack-integration)
    - [Embedded Native Desktop Apps](https://github.com/ServiceStack/ServiceStack.Gap)
    - [Auto Batched Requests](/auto-batched-requests)
    - [Versioning](/versioning)
    - [Multitenancy](/multitenancy)

- Caching
    - [Overview](/caching)
    - [HTTP Caching](/http-caching)
        - [CacheResponse Attribute](/cacheresponse-attribute)
        - [Cache Aware Clients](/cache-aware-clients)

- Auto Query
    - [Overview](/autoquery)
    - [Why Not OData](/why-not-odata)
    - [AutoQuery RDBMS](/autoquery-rdbms)
    - [AutoQuery Data](/autoquery-data)
        - [AutoQuery Data Memory](/autoquery-memory)
        - [AutoQuery Data Service](/autoquery-service)
        - [AutoQuery Data DynamoDB](/autoquery-dynamodb)

- Server Events
    - [Overview](/server-events)
    - [JavaScript Client](/javascript-server-events-client)
    - [C# Server Events Client](/csharp-server-events-client)
    - [Redis Server Events](/redis-server-events)

- Service Gateway
    - [Overview](/service-gateway)
    - [Service Discovery](/service-discovery)

- Encrypted Messaging
    - [Overview](/encrypted-messaging)
    - [Encrypted Service Client](/encrypted-messaging#encrypted-service-client)
  
- Plugins
    - [Overview](/plugins)
    - [Swagger API](/swagger-api)
    - [Postman](/postman)
    - [CORS Feature](/corsfeature)
    - [Request logger](/request-logger)
    - [Sitemaps](/sitemaps)
    - [Cancellable Requests](/cancellable-requests)

- Tests
    - [Testing](/testing) 
    - [HowTo write unit/integration tests](/howto-write-unit-integration-tests)

- ServiceStackVS
    - [Install ServiceStackVS](/install-servicestackvs)
    - [Add ServiceStack Reference](/add-servicestack-reference)
    - [TypeScript React Template](https://github.com/ServiceStackApps/typescript-react-template/)
      - [React, Redux Chat App](https://github.com/ServiceStackApps/ReactChat)
    - [AngularJS App Template](https://github.com/ServiceStack/ServiceStackVS/blob/master/docs/angular-spa.md)
    - [React Desktop Apps](https://github.com/ServiceStackApps/ReactDesktopApps)

- Other Languages
    - [Java](https://github.com/ServiceStack/ServiceStack.Java)
        - [Add ServiceStack Reference](/java-add-servicestack-reference)
        - [Android Studio & IntelliJ](/java-add-servicestack-reference#servicestack-idea-android-studio-plugin)
        - [Eclipse](https://github.com/ServiceStack/ServiceStack.Java/tree/master/src/ServiceStackEclipse#eclipse-integration-with-servicestack)
    - [Swift](/swift)
        - [Add ServiceStack Reference](/swift-add-servicestack-reference)
    - [F#](/fsharp)
        - [Add ServiceStack Reference](/fsharp-add-servicestack-reference)
    - [VB.NET](/vbnet)
        - [Add ServiceStack Reference](/vbnet-add-servicestack-reference)

- Amazon Web Services
  - [ServiceStack.Aws](https://github.com/ServiceStack/ServiceStack.Aws)
  - [PocoDynamo](https://github.com/ServiceStack/PocoDynamo)
  - [AWS Live Demos](http://awsapps.servicestack.net)
  - [Getting Started with AWS](https://github.com/ServiceStackApps/AwsGettingStarted)

- Deployment
    - [Deploy Multiple Sites to single AWS Instance](/deploy-multiple-sites-to-aws)
        - [Simple Deployments to AWS with WebDeploy](/simple-deployments-to-aws)
    - [Advanced Deployments with OctopusDeploy](/advanced-deployment-octopus-deploy)

- Install 3rd Party Products
    - [Redis on Windows](https://github.com/ServiceStack/redis-windows)
    - [RabbitMQ on Windows](https://github.com/ServiceStack/rabbitmq-windows)

- Use Cases
    - [Single Page Apps](/single-page-apps) 
      - [HTML, CSS and JavaScript Minification](/html-css-and-javascript-minification)
    - [Azure](/azure)
      - [Connecting to Azure Redis via SSL](/ssl-redis-azure)
    - [Logging](/logging)
    - [Bundling and Minification](/bundling-and-minification)
    - [NHibernate](/nhibernate) 

- Performance
    - [Real world performance](/real-world-performance) 

- Other Products
    - [ServiceStack.Redis](https://github.com/ServiceStack/ServiceStack.Redis)
    - [ServiceStack.OrmLite](https://github.com/ServiceStack/ServiceStack.OrmLite)
    - [ServiceStack.Text](https://github.com/ServiceStack/ServiceStack.Text)
    - [ServiceStack.Aws](https://github.com/ServiceStack/ServiceStack.Aws)
    - [Stripe](https://github.com/ServiceStack/Stripe)

- Future
    - [Roadmap](/roadmap)