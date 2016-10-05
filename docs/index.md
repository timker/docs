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
- [Pre-release MyGet Packages](?id=MyGet) - The latest downloads in-between each release are available on MyGet

## ServiceStack Community

- [ServiceStack G+ Community](https://plus.google.com/communities/112445368900682590445) Get in touch with the ServiceStack Community
- [Community-Resources](?id=community-resources) (Blog Posts, Gists, StackOverflow, Pod casts...)
- [Contributors](?id=contributors) - Special thanks to all of ServiceStack's Contributors
- [Contributing Guide](?id=contributing) - Read our Contributing Guide before contributing

## ServiceStack Documentation

> [Prerequisites](?id=prerequisites) - Before you start reading, you should know the basics about HTTP (HTTP methods/verbs, status codes etc) and REST/SOAP

- [Why ServiceStack?](?id=why-servicestack)
- [Architecture Overview](?id=architecture-overview)
- [What is a message-based WebService?](?id=What-is-a-message-based-web-service)
- [Advantages of message-based WebServices](?id=advantages-of-message-based-web-services)
- [Why remote services should use DTOs?](http://stackoverflow.com/a/15369736/85785)

- Getting Started
    - [Create your first WebService](?id=create-your-first-webservice)
      - [Creating a WebService from scratch](?id=create-webservice-from-scratch)
    - [Your first webservice explained](?id=your-first-webservice-explained)
    - [Example Projects Overview](http://stackoverflow.com/a/15869816/85785)    
    - [Learning Resources](?id=Learning-ServiceStack)

- Designing APIs
    - [ServiceStack's API Design](?id=api-design)
    - [Designing a REST-ful service with ServiceStack](http://stackoverflow.com/a/15235822/85785)
      - [Simple Customer REST Example](http://stackoverflow.com/a/30273466/85785)
    - [How to design a Message-Based API](http://stackoverflow.com/a/15941229/85785)
    - [Software complexity, goals of Services and important role of DTOs](http://stackoverflow.com/a/32940275/85785)

- Reference
    - [Order of Operations](?id=order-of-operations)
    - [The IoC container](?id=ioc)
    - [Configuration & AppSettings](?id=appsettings)
    - [Metadata page](?id=Metadata-page)
    - [REST, SOAP & default endpoints](?id=endpoints)
    - [SOAP support](?id=soap-support)
    - [Routing](?id=routing)
    - [Service return types](?id=service-return-types)
    - [Customize HTTP Responses](?id=customize-http-responses)
    - [Customize JSON Responses](?id=customize-json-responses)
    - [Plugins](?id=plugins)
    - [Validation](?id=Validation)
    - [Error Handling](?id=error-handling)
    - [Security](?id=security)
    - [Debugging](?id=debugging)
    - [JavaScript Client Library (ss-utils.js)](?id=ss-utils-js)

- Clients
    - [Overview](?id=clients-overview)
    - [C#/.NET client](?id=csharp-client)
        - [.NET Core Clients](https://github.com/ServiceStack/ServiceStack/blob/netcore/docs/pages/netcore.md)
    - [Add ServiceStack Reference](?id=add-servicestack-reference)
        - [C# Add Reference](?id=csharp-add-servicestack-reference)
        - [TypeScript Add Reference](?id=typescript-add-servicestack-reference)
        - [Swift Add Reference](?id=swift-add-servicestack-reference)
        - [Java Add Reference](?id=java-add-servicestack-reference)
        - [Kotlin Add Reference](?id=kotlin-add-servicestack-reference)
        - [F# Add Reference](?id=fsharp-add-servicestack-reference)
        - [VB.NET Add Reference](?id=vbnet-add-servicestack-reference)
    - [JavaScript client](?id=javascript-client)
    - [Silverlight client](?id=silverlight-client)
    - [Dart Client](?id=dart-client)
    - [MQ Clients](?id=messaging#mq-client-architecture)

- Formats
    - [Overview](?id=Formats)
    - [JSON, JSV & XML](?id=json-jsv-and-xml)
    - [CSV Format](?id=csv-format)
    - [MessagePack Format](?id=messagepack-format)
    - [ProtoBuf Format](?id=protobuf-format)
    - [HTML5 Report Format](?id=html5reportformat)

- View Engines
    - [Razor & Markdown Razor](http://razor.servicestack.net/)
    - [Razor Notes](?id=razor-notes)
    - [View and Template Selection](?id=view-and-template-selection)
      - [Compiled Razor Views](?id=compiled-razor-views)
      - [Razor Views vs Content Pages](http://stackoverflow.com/questions/13206038/servicestack-razor-default-page/13206221#13206221)
      - [Self-Hosted Razor Views](http://www.ienablemuch.com/2012/12/self-hosting-servicestack-serving.html) 
      - [Theming Razor Views](http://www.mattjcowan.com/funcoding/2013/06/24/theming-servicestack-razor-views/)
    - [Markdown Razor](?id=markdown-razor)

- Hosts
    - [IIS](?id=iis)
    - [Self-hosting](?id=self-hosting)
    - [Messaging](?id=messaging)
        - [Rabbit MQ](?id=rabbit-mq)
        - [Rabbit MQ](?id=rabbit-mq)
        - [Amazon SQS](https://github.com/ServiceStack/ServiceStack.Aws#sqsmqserver)
    - [Mono](?id=mono)
        - [Run ServiceStack as a daemon on Linux](?id=servicestack-as-daemon-on-linux)
        - [Recommended guide for hosting on Mono](https://github.com/ServiceStackApps/mono-server-config)
        - [Docker image for running in a docker container](https://github.com/ServiceStackApps/mono-docker-config)
    - [Linux-Hosting-Options](?id=linux-hosting-options)

- Security
    - [Authentication & Authorization](?id=authentication-and-authorization)
        - [JWT AuthProvider](?id=jwt-authprovider)
        - [API Key AuthProvider](?id=api-key-authprovider)
        - [Open Id 2.0 Support](?id=openid)
    - [Sessions](?id=sessions)
    - [Restricting Services](?id=restricting-services)
    - [Encrypted Messaging](?id=encrypted-messaging)

- Advanced
    - [Configuration options](?id=configuration-options)
        - [Run ServiceStack side-by-side with another web framework](?id=servicestack-side-by-side-with-another-web-framework)
    - [Access HTTP specific features in services](?id=access-http-specific-features-in-services)
    - [Logging](?id=Logging)
    - [Serialization & Deserialization](?id=serialization-deserialization)
    - [Request & Response filters](?id=request-and-response-filters)
    - [Filter attributes](?id=filter-attributes)
    - [Concurrency Model](?id=concurrency-model)
    - [Built-in profiling](?id=built-in-profiling)
    - [Form Hijacking Prevention](?id=form-hijacking-prevention)
    - [Auto-Mapping](?id=auto-mapping)
    - [HTTP Utils](?id=http-utils)
    - [Dump Utils](?id=dump-utils)
    - [Virtual File System](?id=virtual-file-system)
    - [Config API](?id=config-api)
    - [Physical Project Structure](?id=physical-project-structure)
    - [Modularizing Services](?id=modularizing-services)
    - [MVC Integration](?id=mvc-integration)
    - [ServiceStack Integration](?id=servicestack-integration)
    - [Embedded Native Desktop Apps](https://github.com/ServiceStack/ServiceStack.Gap)
    - [Auto Batched Requests](?id=auto-batched-requests)
    - [Versioning](?id=versioning)
    - [Multitenancy](?id=multitenancy)

- Caching
    - [Overview](?id=caching)
    - [HTTP Caching](?id=http-caching)
        - [CacheResponse Attribute](?id=cacheresponse-attribute)
        - [Cache Aware Clients](?id=cache-aware-clients)

- Auto Query
    - [Overview](?id=autoquery)
    - [Why Not OData](?id=why-not-odata)
    - [AutoQuery RDBMS](?id=autoquery-rdbms)
    - [AutoQuery Data](?id=autoquery-data)
        - [AutoQuery Data Memory](?id=autoquery-memory)
        - [AutoQuery Data Service](?id=autoquery-service)
        - [AutoQuery Data DynamoDB](?id=autoquery-dynamodb)

- Server Events
    - [Overview](?id=server-events)
    - [JavaScript Client](?id=javascript-server-events-client)
    - [C# Server Events Client](?id=csharp-server-events-client)
    - [Redis Server Events](?id=redis-server-events)

- Service Gateway
    - [Overview](?id=service-gateway)
    - [Service Discovery](?id=service-discovery)

- Encrypted Messaging
    - [Overview](?id=encrypted-messaging)
    - [Encrypted Service Client](?id=encrypted-messaging#encrypted-service-client)
  
- Plugins
    - [Overview](?id=plugins)
    - [Swagger API](?id=swagger-api)
    - [Postman](?id=postman)
    - [CORS Feature](?id=corsfeature)
    - [Request logger](?id=request-logger)
    - [Sitemaps](?id=sitemaps)
    - [Cancellable Requests](?id=cancellable-requests)

- Tests
    - [Testing](?id=testing) 
    - [HowTo write unit/integration tests](?id=HowTo-write-unit-integration-tests)

- ServiceStackVS
    - [Install ServiceStackVS](?id=install-servicestackvs)
    - [Add ServiceStack Reference](?id=add-servicestack-reference)
    - [TypeScript React Template](https://github.com/ServiceStackApps/typescript-react-template/)
      - [React, Redux Chat App](https://github.com/ServiceStackApps/ReactChat)
    - [AngularJS App Template](https://github.com/ServiceStack/ServiceStackVS/blob/master/docs/angular-spa.md)
    - [React Desktop Apps](https://github.com/ServiceStackApps/ReactDesktopApps)

- Other Languages
    - [Java](https://github.com/ServiceStack/ServiceStack.Java)
        - [Add ServiceStack Reference](?id=Java-Add-ServiceStack-Reference)
        - [Android Studio & IntelliJ](?id=Java-Add-ServiceStack-Reference#servicestack-idea-android-studio-plugin)
        - [Eclipse](https://github.com/ServiceStack/ServiceStack.Java/tree/master/src/ServiceStackEclipse#eclipse-integration-with-servicestack)
    - [Swift](?id=swift)
        - [Add ServiceStack Reference](?id=Swift-Add-ServiceStack-Reference)
    - [F#](?id=fsharp)
        - [Add ServiceStack Reference](?id=fSharp-add-servicestack-reference)
    - [VB.NET](?id=vbnet)
        - [Add ServiceStack Reference](?id=vbnet-add-servicestack-reference)

- Amazon Web Services
  - [ServiceStack.Aws](https://github.com/ServiceStack/ServiceStack.Aws)
  - [PocoDynamo](https://github.com/ServiceStack/PocoDynamo)
  - [AWS Live Demos](http://awsapps.servicestack.net)
  - [Getting Started with AWS](https://github.com/ServiceStackApps/AwsGettingStarted)

- Deployment
    - [Deploy Multiple Sites to single AWS Instance](?id=deploy-multiple-sites-to-aws)
        - [Simple Deployments to AWS with WebDeploy](?id=simple-deployments-to-aws)
    - [Advanced Deployments with OctopusDeploy](?id=advanced-deployment-octopus-deploy)

- Install 3rd Party Products
    - [Redis on Windows](https://github.com/ServiceStack/redis-windows)
    - [RabbitMQ on Windows](https://github.com/ServiceStack/rabbitmq-windows)

- Use Cases
    - [Single Page Apps](?id=single-page-apps) 
      - [HTML, CSS and JavaScript Minification](?id=html-css-and-javascript-minification)
    - [Azure](?id=azure)
      - [Connecting to Azure Redis via SSL](?id=Secure-SSL-Redis-connections-to-Azure-Redis)
    - [Logging](?id=logging)
    - [Bundling and Minification](?id=bundling-and-minification)
    - [NHibernate](?id=nhibernate) 

- Performance
    - [Real world performance](?id=real-world-performance) 

- Other Products
    - [ServiceStack.Redis](https://github.com/ServiceStack/ServiceStack.Redis)
    - [ServiceStack.OrmLite](https://github.com/ServiceStack/ServiceStack.OrmLite)
    - [ServiceStack.Text](https://github.com/ServiceStack/ServiceStack.Text)
    - [ServiceStack.Aws](https://github.com/ServiceStack/ServiceStack.Aws)
    - [Stripe](https://github.com/ServiceStack/Stripe)

- Future
    - [Roadmap](?id=roadmap)