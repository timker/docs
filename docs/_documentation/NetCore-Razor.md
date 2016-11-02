---
title: Smart MVC Razor Pages
slug: netcore-razor
---

Driven by our preference for 
[API-first style of Web Development](https://github.com/ServiceStackApps/EmailContacts#api-first-development)
we never saw value in the boilerplate required in MVC's forced Controller and Action pattern so we instead 
developed our own [ServiceStack Razor Pages](http://razor.netcore.io) which let you develop dynamic
Web Pages using Razor to generate the HTML view of your existing Services - saving you from maintaining a 
parallel Controller implementation that's limited to just Web Pages. The benefits of an API-first
approach is that you'll naturally get a well-defined servicified interface which can be consumed by all 
consumers including Web, Native Mobile and Desktop Apps whilst also enabling simplified B2B Integrations, 
Automation, Integration testing, etc.

#### [No Ceremony Dynamic Pages without Controllers](http://razor.netcore.io/#no-ceremony)

In a lot of cases where you're not developing Web Forms accepting User Input (e.g. generating a dynamic 
page using simple page-specific db queries) you won't need a Controller or Service at all. 
For this scenario we developed [Controller-less Razor Pages](http://razor.netcore.io/#smart-views), 
where if you specify a Typed `@model`, ServiceStack automatically populates it from the HTTP Request Params 
and when no `@model` exists ServiceStack instead populates the Request params in a `ViewDataDictionary` - 
in both cases letting you access any Request Params using **@Model.Name** notation.

Razor Pages also lets you layout your Razor Views in whatever structure you want under `/wwwroot` 
which it will let you call using [Pretty URLs by default](http://razor.netcore.io/#no-ceremony) so you're 
not led into following the MVC-specific `{Controller}/{Action}` pattern or made to define Custom Routes.

.NET Core Razor Pages implementation also lets you structure your Razor Pages under `/Views/Pages` as an 
alternative to maintaining them under `/wwwroot`.

### MVC Razor Pages

Unfortunately in .NET Core we weren't able to reuse any of our existing **ServiceStack.Razor** implementation,
but as we found the development model and end-user experience of Razor without MVC Controllers and Actions 
much more productive we investigated how it could best be implemented in .NET Core. Unfortunately 
.NET Core's Razor support is tightly coupled to MVC's implementation, but fortunately for us MVC also 
provided the necessary APIs where we could re-implement ServiceStack.Razor's user-facing features using 
just MVC Razor Views. 

In many ways this turned out to be a blessing in disguise as by using MVC's implementation we also get 
access to new MVC .NET Core features and its surrounding ecosystem like Tag Helpers. MVC also takes care 
of live-reloading Razor Views behind-the-scenes so we're also able to get the same iterative development 
experience we're used to. By using MVC Views we also naturally get good tooling support which 
[can be a dark art in .NET 4.5](http://docs.servicestack.net/razor-notes.html)
which was tightly coupled to **Web.config** configuration and therefore poorly supported in Self-Hosting 
Console Apps. 

> Currently ReSharper's tooling has issues with Razor Views inheriting Custom base classes - 
which can be resolved by installing the latest EAP or disabling its **ASP.NET Razor** support.

Overall we're ecstatic with the end-result, we retain our Controller-free development model whilst Razor under 
.NET Core executes noticeably quicker than ASP.NET and significantly faster on Linux vs using Mono.

### RazorFormat Usage

You can find .NET Core Razor features documented in [razor.netcore.io](http://razor.netcore.io) which 
is maintained in our MVC NuGet package that can be installed with: 

    PM> Install-Package ServiceStack.Mvc.Core 

Then to enable, register the `RazorFormat` plugin:

```csharp
public override void Configure(Container container)
{
    Plugins.Add(new RazorFormat());
}
```
