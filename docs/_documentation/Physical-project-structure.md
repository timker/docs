---
#File header for Jekyll to pick up 
---
Ideally the root-level **AppHost** project should be kept lightweight and implementation-free. Although for small projects with only a few services it's ok for everything to be in a single project and to simply grow your architecture when and as needed. 

For medium-to-large projects we recommend the physical structure below which we'll model using [this concrete Events example](http://stackoverflow.com/a/15235822/85785) to describe how we'd typically layout a ServiceStack project. For the purposes of this illustration we'll assume our Application is called **EventMan**. 

The order of the projects also show its dependencies, e.g. the top-level `EventMan` project references **all** sub projects whilst the last `EventMan.ServiceModel` project references **none**:

```
/EventMan
  AppHost.cs                  //The ServiceStack ASP.NET Web or Console Host Project

/EventMan.ServiceInterface     //All Service implementations (akin to MVC Controllers)
  EventsService.cs
  EventsReviewsService.cs

/EventMan.Logic                //For larger projs: pure C# logic deps, data models, etc
  IGoogleCalendarGateway      //E.g of a external dependency this project could use

/EventMan.ServiceModel         //Service Request/Response DTOs and DTO types in /Types
  Events.cs                   //Events, CreateEvent, GetEvent, UpdateEvent DTOs 
  EventReviews.cs             //EventReviews, GetEventReview, CreateEventReview DTOs
  /Types
    Event.cs                  //Event type
    EventReview.cs            //EventReview type
```

With the `EventMan.ServiceModel` DTO's kept in their own separate implementation and dependency-free dll, you're freely able to share this dll in any .NET client project as-is - which you can use with any of the generic [C# Service Clients](https://github.com/ServiceStack/ServiceStack/wiki/C%23-client) to provide an end-to-end typed API without any code-gen.

### Example of Recommended Project Structure

The [EmailContacts solution](https://github.com/ServiceStack/EmailContacts/) details the recommended setup and physical layout structure of typical medium-sized ServiceStack projects. It includes the complete documentation going through how to create the solution from scratch, and explains all the ServiceStack hidden features it makes use of along the way.