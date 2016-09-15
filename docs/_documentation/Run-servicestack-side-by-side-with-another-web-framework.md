---
slug: run-servicestack-side-by-side-with-another-web-framework
---
In order to avoid conflicts with your existing ASP.NET web framework it is recommended to host your ServiceStack web services at a custom path.
This will allow you to use ServiceStack together with an existing web framework e.g. ASP.NET MVC 3 or FUBU MVC, etc.

The location configuration (to your root Web.config file) below hosts your webservices at custom path: `/api`

```xml
<configuration>
  <!-- ... --> 
  <location path="api">
    <system.web>
      <httpHandlers>
        <add path="*" type="ServiceStack.HttpHandlerFactory, ServiceStack" verb="*"/>
      </httpHandlers>
    </system.web>

    <!-- Required for IIS 7.0 -->
    <system.webServer>
      <modules runAllManagedModulesForAllRequests="true"/>
      <validation validateIntegratedModeConfiguration="false" />
      <handlers>
        <add path="*" name="ServiceStack.Factory" 
             type="ServiceStack.HttpHandlerFactory, ServiceStack" 
             verb="*" preCondition="integratedMode" 
             resourceType="Unspecified" allowPathInfo="true" />
      </handlers>
    </system.webServer>
  </location>
  <!-- ... --> 
</configuration>
```

Configuration for also running on Mono / IIS 6:

```xml
<!-- Required for MONO -->
<configuration>
  <!-- ... --> 
  <system.web>
    <httpHandlers>
      <add path="api*" type="ServiceStack.HttpHandlerFactory, ServiceStack" verb="*"/>
    </httpHandlers>
  </system.web>
  <system.webServer>
    <validation validateIntegratedModeConfiguration="false" />
  </system.webServer>
  <!-- ... --> 
</configuration>
```
> **Note:** Due to limitations in IIS 6 - the `/custompath` must end with `.ashx`, e.g: `path="api.ashx"`

You also need to configure the root path in your AppHost.

```c#
public override void Configure(Container container)
{
    SetConfig(new HostConfig { HandlerFactoryPath = "api" });
}
```

**To avoid conflicts with ASP.NET MVC add an ignore rule** in `Global.asax RegisterRoutes` method e.g: `routes.IgnoreRoute ("api/{*pathInfo}");`

**For MVC4 applications you also need to unregister WebApi**, by commenting out this line in `Global.asax.cs`:
```csharp
    //WebApiConfig.Register(GlobalConfiguration.Configuration);
```

> **Note:** If you used Nuget to install the bits, remove the original handler from the web.config system.webserver node e.g: 

```xml

<add path="*" name="ServiceStack.Factory"
    type="ServiceStack.HttpHandlerFactory, ServiceStack" verb="*" 
    preCondition="integratedMode" resourceType="Unspecified" allowPathInfo="true" />
```

**Due to Mono bug you need to set up explicitly ASP.NET auth to 'windows' mode** when using ServiceStack auth on Mono, otherwise auth will be always redirected to /login.aspx insted of getting nice 401 (invalid username or password) response.

Add to `web.config` under `system.web` section.

```xml
<!-- Required to use ServiceStack auth under Mono -->
<authentication mode="Windows" />
```

### Resources

* [Example config files for Starter Templates](https://github.com/ServiceStackApps/LiveDemos#starter-templates)