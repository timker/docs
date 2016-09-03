---
#File header for Jekyll to pick up 
---
## VS.NET Intelli-sense for self-hosting projects

VS.NET Intelli-sense relies on the `Web.config` that VS.NET looks for in the root directory of your host projects. As self-hosting projects are **Console Applications** they instead use `App.config` instead which is all ServiceStack looks at for configuring Razor. 

Unfortunately as VS.NET's Razor intelli-sense is coupled to ASP.NET MVC, it requires a dummy Web.config in your self-hosted projects which just contains a copy of the Razor configuration in your **App.config** (which was originally populated when adding the [ServiceStack.Razor](http://www.nuget.org/packages/ServiceStack.Razor) NuGet Package to your project). The Web.config is otherwise benign and has no other effect other than enabling VS.NET's intelli-sense.

### Intelli-sense for View Models

The `@model T` attribute isn't known to VS.NET intelli-sense when self-hosting which means you need to its more verbose alias:

    @inherits ViewPage<T>

## Missing Razor configuration

The razor Web.config sections added by [ServiceStack.Razor](https://www.nuget.org/packages/ServiceStack.Razor) normally use the version of ASP.NET WebPages that's installed on your computer included with VS.NET installation and updates and is normally located under: 

    C:\Program Files (x86)\Microsoft ASP.NET\ASP.NET Web Pages\

This holds the different version of ASP.NET Web pages installed, e.g:

    v1.0\
    v2.0\

#### Install [ASP.NET MVC3 using the Web Platform Installer](http://www.microsoft.com/web/downloads/platform.aspx)

If this is missing, you can add it to your local machine by installing [ASP.NET MVC3 using the Web Platform Installer](http://www.microsoft.com/web/downloads/platform.aspx).

Another option for installing [ASP.NET WebPages](https://www.nuget.org/packages/Microsoft.AspNet.WebPages) is via NuGet, i.e:

    PM> Install-Package Microsoft.AspNet.WebPages -Version 1.0.20105.408

If you instead installed the latest version of WebPages (currently at v3.1.1), e.g:

    PM> Install-Package Microsoft.AspNet.WebPages

You would need to change the version number in the [Razor Web.config](https://github.com/ServiceStack/ServiceStack/blob/master/NuGet/ServiceStack.Razor/content/web.config.transform) to reference **Version=3.0.0.0**.

#### Only configuration section used

ServiceStack doesn't use the ASP.NET WebPages implementation itself, the configuration is primarily included to enable VS.NET intelli-sense and provide a way to configure the default namespaces added to Razor pages. 

This can also be done in code by adding to the `Config.RazorNamespaces` collection, but adding them to the config section lets VS.NET knows about them so you can get proper intelli-sense. 