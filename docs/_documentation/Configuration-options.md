---
slug: configuration-options
---
As you might have seen in the last tutorials, the main configuration is made in the app host `Configure` method.

```csharp
//Set JSON web services to return idiomatic JSON camelCase properties	
ServiceStack.Text.JsConfig.EmitCamelCaseNames = true;
```

```csharp
//Change the default ServiceStack configuration
Feature disableFeatures = Feature.Jsv | Feature.Soap;
SetConfig(new HostConfig {
    EnableFeatures = Feature.All.Remove(disableFeatures), //all formats except of JSV and SOAP
    DebugMode = true, //Show StackTraces in service responses during development
    WriteErrorsToResponse = false, //Disable exception handling
    DefaultContentType = ContentType.Json, //Change default content type
    AllowJsonpRequests = true //Enable JSONP requests
});
```

| Property name          | Default value | Description                                                                 |
|:-----------------------|--------------:|:---------------------------------------------------------------------------:|
| EnableFeatures         | Feature.All   | Defines the formats which the webservice should understand                  |    
| DebugMode              | false         | If true, stack traces will be shown in service responses during development |   
| WriteErrorsToResponse  |    true       |
| DefaultContentType     | ContentType.Json |     
| AllowJsonpRequests      |  true |
| GlobalResponseHeaders | | Sets global headers which will be added to every request response. ([Example](http://stackoverflow.com/questions/8211930/servicestack-rest-api-and-cors)) |

> Note: `DebugMode` should be set to false in production because of security reasons!

## Logging

To ensure every ServiceStack service uses the same Global Logger, set it before you initialize ServiceStack's AppHost, e.g:

```csharp
  LogManager.LogFactory = new DebugLogFactory(); 
  new AppHost().Init();
```