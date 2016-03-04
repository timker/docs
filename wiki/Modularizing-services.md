ServiceStack only allows a **single App Host** for each App Domain. As you might be able to infer from the name, the role of the **Host** project is to be the conduit for binding all your services concrete dependencies, plugins, filters and everything else your service needs. The configuration of your service should be immutable after everything is initialized in your `AppHost.Configure()` method. The [Physical project structure wiki page](https://github.com/ServiceStack/ServiceStack/wiki/Physical-project-structure) wiki shows the recommended physical project structure for typical solutions.

### Modularizing services in multiple assemblies

Whilst you can only have 1 AppHost, you can have multiple services spread across multiple assemblies. In that case you have to provide a list of assemblies containing the services to the AppHostBase constructor, e.g:

```C#
public class AppHost : AppHostBase
{
    //Tell ServiceStack the name of your app and which assemblies to scan for services
    public AppHost() : base("Hello ServiceStack!", 
       typeof(ServicesFromDll1).Assembly,
       typeof(ServicesFromDll2).Assembly
       /*, etc */) {}

    public override void Configure(Container container) {}
}
```

You can also provide your own strategy for discovering and resolving the service types that ServiceStack should auto-wire by overriding `CreateServiceManager`, e.g:

```C#
public class AppHost : AppHostBase
{
    public AppHost(): base("Hello ServiceStack!", typeof(ServicesFromDll1).Assembly) {}
    public override void Configure(Container container) {}

    //Provide Alternative way to inject IOC Container + Service Resolver strategy
    protected override ServiceManager CreateServiceManager(
        params Assembly[] assembliesWithServices)
    {       
        return new ServiceManager(new Container(),
            new ServiceController(() => assembliesWithServices.ToList()
                .SelectMany(x => x.GetTypes())));
    }
}
```

### Encapsulating Services inside Plugins

One way of modularizing services is to encapsulate them inside [Plugins](https://github.com/ServiceStack/ServiceStack/wiki/Plugins) which allows you to manually register services, custom routes, filters, content types, allow customization and anything else your module needs.

To illustrate this point, we'll show what a Basic [Auth Feature](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack/AuthFeature.cs) example might look like:

```C#
public class BasicAuthFeature : IPlugin 
{
    public string HtmlRedirect { get; set; }   //User-defined configuration

    public void Register(IAppHost appHost)
    {
        //Register Services exposed by this module
        appHost.RegisterService<AuthService>("/auth", "/auth/{provider}");
        appHost.RegisterService<AssignRolesService>("/assignroles");
        appHost.RegisterService<UnAssignRolesService>("/unassignroles");

        //Load dependent plugins
        appHost.LoadPlugin(new SessionFeature());
    }
}
```

With everything encapsulated inside a plugin, your users can easily enable them in your AppHost with:

```C#
Plugins.Add(new BasicAuthFeature { HtmlRedirect = "~/login" });
```