***  

 - [Why ServiceStack?](?id=why-servicestack)
  - [Important role of DTOs](http://stackoverflow.com/a/32940275/85785)
  - [What is a message based web service?](?id=what-is-a-message-based-web-service)
  - [Advantages of message based web services](?id=advantages-of-message-based-web-services)
  - [Why remote services should use separate DTOs](http://stackoverflow.com/a/15369736/85785)

***

- Getting Started
    - [Creating your first project](?id=creating-your-first-project)
      - [Create Service from scratch](?id=create-your-first-webservice)
    - [Your first webservice explained](?id=your-first-webservice-explained)
    - [Example Projects Overview](http://stackoverflow.com/a/15869816/85785)    
    - [Learning Resources](?id=learning-servicestack)

- Designing APIs
    - [ServiceStack API Design](?id=new-api)
    - [Designing a REST-ful service with ServiceStack](http://stackoverflow.com/a/15235822/85785)
      - [Simple Customer REST Example](http://stackoverflow.com/a/30273466/85785)
    - [How to design a Message-Based API](http://stackoverflow.com/a/15941229/85785)
    - [Software complexity and role of DTOs](http://stackoverflow.com/a/32940275/85785)

- Reference
    - [Order of Operations](?id=order-of-operations)
    - [The IoC container](?id=the-ioc-container)
    - [Configuration and AppSettings](?id=appsettings)
    - [Metadata page](?id=metadata-page)
    - [Rest, SOAP & default endpoints](?id=endpoints)
    - [SOAP support](?id=soap-support)
    - [Routing](?id=routing)
    - [Service return types](?id=service-return-types)
    - [Customize HTTP Responses](?id=customize-http-responses)
    - [Customize JSON Responses](?id=customize json responses)
    - [Plugins](?id=plugins)
    - [Validation](?id=validation)
    - [Error Handling](?id=error-handling)
    - [Security](?id=security)
    - [Debugging](?id=debugging)
    - [JavaScript Client Library (ss-utils.js)](?id=ss-utils-js-javascript-client-library)

- Clients
    - [Overview](?id=clients-overview)
    - [C#/.NET client](?id=c#-client)
        - [.NET Core Clients](https://github.com/ServiceStack/ServiceStack/blob/netcore/docs/pages/netcore.md)
    - [Add ServiceStack Reference](?id=add-servicestack-reference)
        - [C# Add Reference](?id=csharp-add-servicestack-reference)
        - [F# Add Reference](?id=fsharp-add-servicestack-reference)
        - [VB.NET Add Reference](?id=vbnet-add-servicestack-reference)
        - [Swift Add Reference](?id=swift-add-servicestack-reference)
        - [Java Add Reference](?id=java-add-servicestack-reference)
    - [Silverlight client](?id=silverlight-service-client)
    - [JavaScript client](?id=javascript-client)
        - [Add TypeScript Reference](?id=typescript-add-servicestack-reference)
    - [Dart Client](?id=dart-client)
    - [MQ Clients](?id=messaging)

- Formats
    - [Overview](?id=formats)
    - [JSON/JSV and XML](?id=json-jsv-and-xml)
    - [HTML5 Report Format](?id=html5reportformat)
    - [CSV Format](?id=csv-format)
    - [MessagePack Format](?id=messagepack-format)
    - [ProtoBuf Format](?id=protobuf-format)

- View Engines
    - [Razor & Markdown Razor](http://razor.servicestack.net/)
      - [Razor Notes](?id=razor-notes)
      - [View and Template Selection](?id=view-and-template-selection)
      - [Compiled Razor Views](?id=compiled-razor-views)
      - [Razor Views vs Content Pages](http://stackoverflow.com/questions/13206038/servicestack-razor-default-page/13206221#13206221)
      - [Self-Hosted Razor Views](http://www.ienablemuch.com/2012/12/self-hosting-servicestack-serving.html) 
    - [Markdown Razor](?id=markdown-razor)

- Hosts
    - [IIS](?id=iis)
    - [Self-hosting](?id=self-hosting)
    - [Messaging](?id=messaging)
        - [Rabbit MQ](?id=rabbit-mq)
        - [Redis MQ](?id=messaging-and-redis)
        - [Amazon SQS](https://github.com/ServiceStack/ServiceStack.Aws#sqsmqserver)
    - [Mono](?id=mono)
        - [Run ServiceStack as a daemon on Linux](?id=run-servicestack-as-a-daemon-on-linux)
        - [Guide for hosting on Mono](https://github.com/ServiceStackApps/mono-server-config)
        - [Running in a Docker container](https://github.com/ServiceStackApps/mono-docker-config)
        - [Linux-Hosting-Options](?id=linux-hosting-options)

- Security
    - [Authentication](?id=authentication-and-authorization)
       - [JWT AuthProvider](?id=jwt-authprovider)
       - [API Key AuthProvider](?id=api-key-authprovider)
       - [Open Id 2.0 Support](?id=openid)
    - [Sessions](?id=sessions)
    - [Restricting Services](?id=restricting-services)
    - [Encrypted Messaging](?id=encrypted-messaging)

- Advanced
    - [Configuration options](?id=configuration-options)
        - [Run ServiceStack side-by-side with another web framework](?id=run-servicestack-side-by-side-with-another-web-framework)
    - [Access HTTP specific features in services](?id=access-http-specific-features-in-services)
    - [Logging](?id=logging)
    - [Serialization/deserialization](?id=serialization-deserialization)
    - [Request/response filters](?id=request-and-response-filters)
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
  - [Caching Providers](?id=caching)
  - [HTTP Caching](?id=http-caching)
    - [CacheResponse Attribute](?id=cacheresponse-attribute)
    - [Cache Aware Clients](?id=cache-aware-clients)

- Auto Query
  - [Overview](?id=auto-query)
  - [Why Not OData](?id=why-not-odata)
  - [AutoQuery RDBMS](?id=autoquery-rdbms)
  - [AutoQuery Data](?id=autoquery-data)
    - [AutoQuery Memory](?id=autoquery-memory)
    - [AutoQuery Service](?id=autoquery-service)
    - [AutoQuery DynamoDB](?id=autoquery-dynamodb) 

- Server Events
    - [Overview](?id=server-events)
    - [JavaScript Client](?id=javascript-server-events-client)
    - [C# Server Events Client](?id=c#-server-events-client)
    - [Redis Server Events](?id=redis-server-events)

- Service Gateway
    - [Overview](?id=service-gateway)
    - [Service Discovery](?id=service-discovery)

- Encrypted Messaging
    - [Overview](?id=encrypted-messaging)
    - [Encrypted Client](?id=encrypted-messaging#encrypted-service-client)
  
- Plugins
    - [Auto Query](?id=auto-query)
    - [Server Sent Events](https://github.com/ServiceStackApps/Chat#server-sent-events)
    - [Swagger API](?id=swagger-api)
    - [Postman](?id=postman)
    - [Request logger](?id=request-logger)
    - [Sitemaps](?id=sitemaps)
    - [Cancellable Requests](?id=cancellable-requests)
    - [CorsFeature](?id=corsfeature)

- Tests
    - [Testing](?id=testing) 
    - [HowTo write unit/integration tests](?id=howto-write-unit-integration-tests)

- ServiceStackVS
    - [Install ServiceStackVS](?id=install-servicestackvs)
    - [Add ServiceStack Reference](?id=Add-ServiceStack-Reference)
    - [TypeScript React Template](https://github.com/ServiceStackApps/typescript-react-template/)
      - [React, Redux Chat App](https://github.com/ServiceStackApps/ReactChat)
    - [AngularJS App Template](https://github.com/ServiceStack/ServiceStackVS/blob/master/docs/angular-spa.md)
    - [React Desktop Apps](https://github.com/ServiceStackApps/ReactDesktopApps)

- Other Languages
    - [FSharp](?id=)
        - [Add ServiceStack Reference](?id=FSharp-Add-ServiceStack-Reference)
    - [VB.NET](?id=)
        - [Add ServiceStack Reference](?id=VB.Net-Add-ServiceStack-Reference)
    - [Swift](?id=)
      - [Swift Add Reference](?id=Swift-Add-ServiceStack-Reference)
    - [Java](https://github.com/ServiceStack/ServiceStack.Java)
        - [Add ServiceStack Reference](?id=Java-Add-ServiceStack-Reference)
        - [Android Studio & IntelliJ](?id=Java-Add-ServiceStack-Reference#servicestack-idea-android-studio-plugin)
        - [Eclipse](https://github.com/ServiceStack/ServiceStack.Java/tree/master/src/ServiceStackEclipse#eclipse-integration-with-servicestack)


- Amazon Web Services
  - [ServiceStack.Aws](https://github.com/ServiceStack/ServiceStack.Aws)
  - [PocoDynamo](https://github.com/ServiceStack/PocoDynamo)
  - [AWS Live Demos](http://awsapps.servicestack.net)
  - [Getting Started with AWS](https://github.com/ServiceStackApps/AwsGettingStarted)

- Deployment
    - [Deploy Multiple Sites to single AWS Instance](?id=)
        - [Simple Deployments to AWS with WebDeploy](?id=)
    - [Advanced Deployments with OctopusDeploy](?id=)

- Install 3rd Party Products
    - [Redis on Windows](https://github.com/ServiceStack/redis-windows)
    - [RabbitMQ on Windows](https://github.com/ServiceStack/rabbitmq-windows)

- Use Cases
    - [Single Page Apps](?id=) 
      - [HTML, CSS and JS Minifiers](?id=HTML%2C-CSS-and-JavaScript-Minification)
    - [Azure](?id=)
      - [Connecting to Azure Redis via SSL](?id=Secure-SSL-Redis-connections-to-Azure-Redis)
    - [Logging](?id=) 
    - [Bundling and Minification](?id=)
    - [NHibernate](?id=) 

- Performance
    - [Real world performance](?id=) 

- Other Products
    - [ServiceStack.Redis](https://github.com/ServiceStack/ServiceStack.Redis)
    - [ServiceStack.OrmLite](https://github.com/ServiceStack/ServiceStack.OrmLite)
    - [ServiceStack.Text](https://github.com/ServiceStack/ServiceStack.Text)

- Future
    - [Roadmap](?id=)