---
#File header for Jekyll to pick up 
---
## An implementation-free logging API for .Net

**ServiceStack.Logging** is an implementation and dependency-free logging API with adapters for all of .NET's popular logging providers.
It allows your business logic to bind to an easily-mockable and testable dependency-free interface whilst providing the flexibility to switch logging providers at runtime.

## Download on NuGet

Currently there are 5 different .NET logging providers available on NuGet:

#### Install-Package [ServiceStack.Logging.NLog](https://nuget.org/packages/ServiceStack.Logging.NLog)
#### Install-Package [ServiceStack.Logging.Elmah](https://nuget.org/packages/ServiceStack.Logging.Elmah)
#### Install-Package [ServiceStack.Logging.Log4Net](https://nuget.org/packages/ServiceStack.Logging.Log4Net)
#### Install-Package [ServiceStack.Logging.Log4Netv129](https://nuget.org/packages/ServiceStack.Logging.Log4Netv129)
#### Install-Package [ServiceStack.Logging.EventLog](https://nuget.org/packages/ServiceStack.Logging.EventLog)

Note: The ConsoleLogger and DebugLogger and are already built-in and bind to .NET Framework's Console and Debug loggers

-----

Even in the spirit of **Bind to interfaces, not implemenations**, many .NET projects still have
a hard dependency to [log4net](http://logging.apache.org/log4net/index.html). 

Although log4net is the standard for logging in .NET, potential problems can arise from your libraries having a hard dependency on it:

* Your library needs to be shipped with a third-party dependency
* Potential conflicts can occur when different libraries have dependency on different versions of log4net (e.g. the 1.2.9 / 1.2.10 dependency problem).
* You may want to use a different logging provider (i.e. network distributed logging)
* You want your logging for Unit and Integration tests to redirect to the Console or Debug logger without any configuraiton.
* Something better like [elmah](http://code.google.com/p/elmah/) can come along requiring a major rewrite to take advantage of it

ServiceStack.Logging solves these problems by providing an implementation-free ILog interface that your application logic can bind to 
where your Application Host project can bind to the concrete logging implementation at deploy or runtime.

ServiceStack.Logging also includes adapters for the following logging providers:

* Elmah
* NLog
* Log4Net 1.2.10+
* Log4Net 1.2.9
* EventLog
* Console Log
* Debug Log
* Null / Empty Log

# Registration Examples

Once on your App Startup, either In your `AppHost.cs` or `Global.asax` file inject the concrete logging implementation that your app should use, e.g.

## Built-in Loggers

```csharp
//Console.WriteLine
LogManager.LogFactory = new ConsoleLogFactory(debugEnabled:true); 
//Debug.WriteLine
LogManager.LogFactory = new DebugLogFactory(debugEnabled:true);   
//Capture logs in StringBuilder
LogManager.LogFactory = new StringBuilderLogFactory(); 
//No Logging (default)
LogManager.LogFactory = new NullLogFactory();   
```

## NLog

```csharp
LogManager.LogFactory = new NLogFactory(); 
```

## Log4Net

```csharp
//Also runs log4net.Config.XmlConfigurator.Configure()
LogManager.LogFactory = new Log4NetFactory(configureLog4Net:true); 
```

## Event Log

```csharp
LogManager.LogFactory = new EventLogFactory("Logging.Tests", "Application");
```

Then your application logic can bind to and use a lightweight implementation-free ILog which at runtime will be an instance of the concrete implementation configured in your host:

```csharp
ILog log = LogManager.GetLogger(GetType());

log.Debug("Debug Event Log Entry.");
log.Warn("Warning Event Log Entry.");
```

## Elmah

To configure Elmah register it before initializing ServiceStack's AppHost, passing in the Global HttpApplication Instance and an alternate logger you'd like to use to for your Debug and Info messages, e.g:

```csharp
public class Global : System.Web.HttpApplication
{
    protected void Application_Start(object sender, EventArgs e)
    {
        var debugMessagesLog = new ConsoleLogFactory();
        LogManager.LogFactory = new ElmahLogFactory(debugMessagesLog, this);
        new AppHost().Init();
    }
}
```

Elmah also requires its handlers and modules to be registered in the **Web.config** which lets you view your Elmah Error Log at: `/elmah.xsd`:

```xml
<configuration>
  <system.web>
    ...
    <httpHandlers>
      <add verb="POST,GET,HEAD" path="elmah.axd" type="Elmah.ErrorLogPageFactory, Elmah" />
    </httpHandlers>
    <httpModules>
      <add name="ErrorLog" type="Elmah.ErrorLogModule, Elmah"/>
    </httpModules>
  </system.web>

  <system.webServer>
    <handlers>
      ...
      <add name="Elmah" path="elmah.axd" verb="POST,GET,HEAD" 
           type="Elmah.ErrorLogPageFactory, Elmah" preCondition="integratedMode" />
    </handlers>
    <modules>
      <add name="Elmah.ErrorLog" type="Elmah.ErrorLogModule, Elmah" preCondition="managedHandler" />
    </modules>
  </system.webServer>

</configuration>
```

For a working example see the [Logging.Elmah UseCase](https://github.com/ServiceStack/ServiceStack.UseCases/tree/master/Logging.Elmah) which has ServiceStack and Elmah configured together.

# Usage Example

Using a logger in your Service is similar to other .NET Logging providers, e.g. you can initialize a static property for the class and use it in your services, e.g:

```csharp
public class MyService : Service
{
    public static ILog Log = LogManager.GetLogger(typeof(MyService));

    public object Any(Request request)
    {
        if (Log.IsDebugEnabled)
            Log.Debug("In Request Service");
    }
}
```
# Community Resources

  - [Using Slack for Logging (with ServiceStack)](http://buildclassifieds.com/2016/02/01/using-slack-for-logging-with-servicestack/) by [@markholdt](https://twitter.com/markholdt)
  - [Elmah, Emails, and ServiceStack](http://rossipedia.com/blog/2013/03/elmah-emails-and-servicestack/) by [@rossipedia](https://twitter.com/rossipedia)
