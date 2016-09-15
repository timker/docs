---
slug: creating-your-first-project
---
This is a quick walkthrough of getting your first web service up and running whilst having a look at the how some of the different components work. 

## Step 1: Download and Install ServiceStackVS 
First we want to install ServiceStackVS Visual Studio extension. The easiest way to do this is to look for it from within Visual Studio by going to Tools->Extensions and Updates and searching the Visual Studio Gallery as seen below.

[![](https://raw.githubusercontent.com/ServiceStack/ServiceStackVS/master/Images/tools_extensions.png)](https://raw.githubusercontent.com/ServiceStack/ServiceStackVS/master/Images/tools_extensions.png)

[![](https://raw.githubusercontent.com/ServiceStack/ServiceStackVS/master/Images/search_download.png)](https://raw.githubusercontent.com/ServiceStack/ServiceStackVS/master/Images/search_download.png)

Optionally it can be downloaded and installed from the [VS.NET Gallery](http://visualstudiogallery.msdn.microsoft.com/5bd40817-0986-444d-a77d-482e43a48da7)

[![VS.NET Gallery Download](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/servicestackvs/vsgallery-download.png)](http://visualstudiogallery.msdn.microsoft.com/5bd40817-0986-444d-a77d-482e43a48da7)

ServiceStackVS supports both VS.NET 2013 and 2012.

### VS.NET 2012 Prerequisites

  - VS.NET 2012 Users must install the [Microsoft Visual Studio Shell Redistributable](http://www.microsoft.com/download/details.aspx?id=40764)
  - It's also highly recommended to [Update to the latest NuGet](http://docs.nuget.org/docs/start-here/installing-nuget). 

> Alternatively if continuing to use an older version of the **NuGet Package Manager** you will need to click on **Enable NuGet Package Restore** after creating a new project to ensure its NuGet dependencies are installed.

## Step 2: Selecting a template

Once the ServiceStackVS extension is installed, you will have new project templates available when creating a new project. For this example, let's choose ServiceStack ASP.NET Empty to get started.

[![](https://raw.githubusercontent.com/ServiceStack/ServiceStackVS/master/Images/new_project_aspnet_empty.png)](https://raw.githubusercontent.com/ServiceStack/ServiceStackVS/master/Images/new_project_aspnet_empty.png)

Once you've created your application from the template, you should have 4 projects in your new solution. If you left the default name, you'll end up with a solution with the following structure.

[![](https://raw.githubusercontent.com/ServiceStack/ServiceStackVS/master/Images/empty_project_solution.png)](https://raw.githubusercontent.com/ServiceStack/ServiceStackVS/master/Images/empty_project_solution.png)

## Step 3: Run your project

Press F5 and run your project!

[![](https://raw.githubusercontent.com/ServiceStack/ServiceStackVS/master/Images/empty_project_run.png)](https://raw.githubusercontent.com/ServiceStack/ServiceStackVS/master/Images/empty_project_run.png)

> If you are continuing to use an older version of the **NuGet Package Manager** you will need to click on **Enable NuGet Package Restore** after creating a new project to ensure its NuGet dependencies are installed. Without this enabled, Visual Studio will not pull down the ServiceStack dependencies and successfully build the project.

#### How does it work?

Now that your new project is running, let's have a look at what we have. The template comes with a single web service route which comes from the request DTO (Data Transfer Object) which is located in the WebApplication1.ServiceModel project under `Hello.cs` file.

```csharp
[Route("/hello/{Name}")]
public class Hello : IReturn<HelloResponse>
{
    public string Name { get; set; }
}

public class HelloResponse
{
    public string Result { get; set; }
}
```

The `Route` attribute is specifying what path `/hello/{Name}` where `{Name}` binds it's value to the public string property of 'Name'.

Let's access the route to see what comes back. Go to the following URL in your address bar, where <root_path> is your server address.

    http://<root_path>/hello/world

You will see a snapshot of the Result in a HTML response format. To change the return format to Json, simply add `?format=json` to the end of the URL. You'll learn more about formats, endpoints (URLs, etc) when you continue reading the documentation.

If we go back to the solution and find the WebApplication1.ServiceInterface and open the `MyServices.cs` file, we can have a look at the code that is responding to the browser, giving us the `Result` back.

```csharp
public class MyServices : Service
{
  public object Any(Hello request)
  {
    return new HelloResponse { Result = "Hello, {0}!".Fmt(request.Name) };
  }
}
```

If we look at the code above, there are a few things to note. The name of the method `Any` means the server will run this method for any of the valid HTTP Verbs. Service methods are where you control what returns from your service.

## Step 4: Exploring the AngularJS Template

Starting a new ServiceStack ASP.NET with AngularJS application will also give you 4 new projects.

- Host project
- Service Interface project
- Service Model project
- Unit Testing project

[![AngularJS Solution](https://raw.githubusercontent.com/ServiceStack/ServiceStackVS/master/Images/angularjs_solution.png)](https://raw.githubusercontent.com/ServiceStack/ServiceStackVS/master/Images/angularjs_solution.png)

The Host project contains an AppHost which has been configured with the RazorFormat plugin as well as hosting all the required JavaScript packages like AngularJS, Bootstrap and jQuery. It is setup initially with a single `_Layout.cshtml` using the default Bootstrap template and a `default.cshtml` which contains the HelloWorld demo.

[![AngularJS Project](https://raw.githubusercontent.com/ServiceStack/ServiceStackVS/master/Images/angularjs_main_project.png)](https://raw.githubusercontent.com/ServiceStack/ServiceStackVS/master/Images/angularjs_main_project.png)

The Host project has dependencies on the Service Model and Service Interface projects. These are the projects that contain your request/response DTOs, validators and filters. This structure is trying to encourage have your data structures and services in separate projects make testing and reuse easier.

[![AngularJS Other Projects](https://raw.githubusercontent.com/ServiceStack/ServiceStackVS/master/Images/angularjs_other_projects.png)](https://raw.githubusercontent.com/ServiceStack/ServiceStackVS/master/Images/angularjs_other_projects.png)

The Unit Testing project, also as a dependency on these projects as it tests them in isolation of the main Host project. In the template, we are using the BasicAppHost to mock the AppHost we are using in the Host project. The example unit test is using NUit to setup and run the tests.

### The Demo

The simple HelloWorld angular application that is provided in the template calls the `/hello/{Name}` route and displays the result in the `<p>` below. 

[![AngularJS Demo](https://raw.githubusercontent.com/ServiceStack/ServiceStackVS/master/servicestackvs-templates.gif)](https://raw.githubusercontent.com/ServiceStack/ServiceStackVS/master/servicestackvs-templates.gif)

### Home Page

The `default.cshtml` home page shows how easy it is to call ServiceStack Services from within AngularJS:

[![AngularJS Home](https://github.com/ServiceStack/ServiceStackVS/raw/master/Images/angularjs_hello_app.png)](https://github.com/ServiceStack/ServiceStackVS/raw/master/Images/angularjs_hello_app.png)

### [[Testing]]
The project templates from the ServiceStackVS extension also include a Tests project. The project structure and addition of the Tests project is there to encourage a pattern that will scale to larger applications whilst maintaining a easy to understand and testable application.

## [Create a WebService from scratch](https://github.com/ServiceStack/ServiceStack/wiki/Create-your-first-webservice)

If you prefer, you can instead [create a ServiceStack Web Service from a blank ASP.NET Web Application](https://github.com/ServiceStack/ServiceStack/wiki/Create-your-first-webservice) another popular option is to [Add ServiceStack to an existing ASP.NET MVC Application](https://github.com/ServiceStack/ServiceStack/wiki/Mvc-integration)

## [Explore more ServiceStack features](https://github.com/ServiceStackApps/EmailContacts/)

The [EmailContacts solution](https://github.com/ServiceStackApps/EmailContacts/) is a new guidance available that walks through the recommended setup and physical layout structure of typical medium-sized ServiceStack projects, including complete documentation of how to create the solution from scratch, whilst explaining all the ServiceStack features it makes use of along the way.


## Community Resources

  - [Creating A Simple Service Using ServiceStack](http://shashijeevan.net/2015/09/20/creating-a-simple-service-using-servicestack/) by [Shashi Jeevan](http://shashijeevan.net/author/shashijeevan/)
  - [Introducing ServiceStack](http://www.dotnetcurry.com/showarticle.aspx?ID=1056) by [@dotnetcurry](https://twitter.com/DotNetCurry)
  - [Create web services in .NET in a snap with ServiceStack](http://www.techrepublic.com/article/create-web-services-in-net-in-a-snap-with-servicestack/) by [@techrepublic](https://twitter.com/techrepublic)
  - [How to build web services in MS.Net using ServiceStack](http://kborra.wordpress.com/2014/07/29/how-to-build-web-services-in-ms-net-using-service-stack/) by [@kishoreborra](http://kborra.wordpress.com/about/)
  - [Creating a Web API using ServiceStack](http://blogs.askcts.com/2014/05/15/getting-started-with-servicestack-part-3/) by [Lydon Bergin](http://blogs.askcts.com/)
  - [Getting Started with ServiceStack: Part 1](http://blogs.askcts.com/2014/04/23/getting-started-with-servicestack-part-one/) by [Lydon Bergin](http://blogs.askcts.com/)
  - [Getting started with ServiceStack – Creating a service](http://dilanperera.wordpress.com/2014/02/22/getting-started-with-servicestack-creating-a-service/)
  - [ServiceStack Quick Start](http://mediocresoft.com/things/servicestack-quick-start) by [@aarondandy](https://github.com/aarondandy)
  - [Fantastic Step-by-step walk-thru into ServiceStack with Screenshots!](http://nilsnaegele.com/codeedge/servicestack.html) by [@nilsnagele](https://twitter.com/nilsnagele)
  - [Your first REST service with ServiceStack](http://tech.pro/tutorial/1148/your-first-rest-service-with-servicestack) by [@cyberzeddk](https://twitter.com/cyberzeddk)
  - [New course: Using ServiceStack to Build APIs](http://blog.pluralsight.com/2012/11/29/new-course-using-servicestack-to-build-apis/) by [@pluralsight](http://twitter.com/pluralsight)
  - [ServiceStack the way I like it](http://tonyonsoftware.blogspot.co.uk/2012/09/lessons-learned-whilst-using.html) by [@tonydenyer](https://twitter.com/tonydenyer)
  - [Generating a RESTful Api and UI from a database with LLBLGen](http://www.mattjcowan.com/funcoding/2013/03/10/rest-api-with-llblgen-and-servicestack/) by [@mattjcowan](https://twitter.com/mattjcowan)
  - [ServiceStack: Reusing DTOs](http://korneliuk.blogspot.com/2012/08/servicestack-reusing-dtos.html) by [@korneliuk](https://twitter.com/korneliuk)
  - [Using ServiceStack with CodeFluent Entities](http://blog.codefluententities.com/2013/03/06/using-servicestack-with-codefluent-entities/) by [@SoftFluent](https://twitter.com/SoftFluent)
  - [ServiceStack, Rest Service and EasyHttp](http://blogs.lessthandot.com/index.php/WebDev/ServerProgramming/servicestack-restservice-and-easyhttp) by [@chrissie1](https://twitter.com/chrissie1)
  - [Building a Web API in SharePoint 2010 with ServiceStack](http://www.mattjcowan.com/funcoding/2012/05/04/building-a-web-api-in-sharepoint-2010-with-servicestack)
  - [REST Raiding. ServiceStack](http://dgondotnet.blogspot.de/2012/04/rest-raiding-servicestack.html) by [Daniel Gonzalez](http://www.blogger.com/profile/13468563783321963413)
  - [JQueryMobile and Service Stack: EventsManager tutorial](http://kylehodgson.com/2012/04/21/jquerymobile-and-service-stack-eventsmanager-tutorial-post-2/) / [Part 3](http://kylehodgson.com/2012/04/23/jquerymobile-and-service-stack-eventsmanager-tutorial-post-3/) by [+Kyle Hodgson](https://plus.google.com/u/0/113523377752095590770/posts)
  - [Like WCF: Only cleaner!](http://kylehodgson.com/2012/04/18/like-wcf-only-cleaner-9/) by [+Kyle Hodgson](https://plus.google.com/u/0/113523377752095590770/posts)
  - [ServiceStack I heart you. My conversion from WCF to SS](http://www.philliphaydon.com/2012/02/service-stack-i-heart-you-my-conversion-from-wcf-to-ss/) by [@philliphaydon](https://twitter.com/philliphaydon)
  - [Service Stack vs WCF Data Services](http://codealoc.wordpress.com/2012/03/24/service-stack-vs-wcf-data-services/)
  - [Creating a basic catalogue endpoint with ServiceStack](http://blogs.7digital.com/dev/2011/10/17/creating-a-basic-catalogue-endpoint-with-servicestack/) by [7digital](http://blogs.7digital.com)
  - [Buildiing a Tridion WebService with jQuery and ServiceStack](http://www.curlette.com/?p=161) by [@robrtc](https://twitter.com/#!/robrtc)
  - [Anonymous type + Dynamic + ServiceStack == Consuming cloud has never been easier](http://www.ienablemuch.com/2012/05/anonymous-type-dynamic-servicestack.html) by [@ienablemuch](https://twitter.com/ienablemuch)
  - [Handful of examples of using ServiceStack based on the ServiceStack.Hello Tutorial](https://github.com/jfoshee/TryServiceStack) by [@82unpluggd](https://twitter.com/82unpluggd)