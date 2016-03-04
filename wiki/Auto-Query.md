### Browse [AutoQuery Services from an iPad!](https://github.com/ServiceStackApps/AutoQueryViewer)

-----

The new AutoQuery support in ServiceStack adds Auto Querying functionality akin to OData's querying support for Web Api, although we've disregarded their approach which we've long considered [promotes web service anti-patterns](http://stackoverflow.com/a/9579090/85785). To explain the design goals behind AutoQuery it's important to identify and avoid the parts of OData we consider make it a poor fit for services:

## Why not OData

[Microsoft's own Services Design guidelines](http://msdn.microsoft.com/en-us/library/ms954638.aspx) provides a good initial summary on why OData-like services are a Services anti-pattern:

 - There is virtually no contract. A service consumer has no idea how to use the service (for example, what are valid Command arguments, encoding expectations, and so on).
 - The interface errs on the side of being too liberal in what it will accept. 
 - The contract does not provide enough information to consumers on how to use the service. If a consumer must read something other than the service's signature to understand how to use the service, the factoring of the service should be reviewed.
 - Consumers are expected to be familiar with the database and table structures prior to consuming the Web service. This results in a tight coupling between service providers and consumers.
 - Performance will suffer due to dependencies on late binding and encoding/decoding between boundaries within the same service.

### OData Example

We previously used one of [Netflix's OData examples to illustrate this](http://stackoverflow.com/a/9579090/85785) showing how to query for movies with specific ratings, a service available before [Netflix retired their OData catalog](http://developer.netflix.com/blog/read/Changes_to_the_Public_API_Program):

 > http://odata.netflix.com/Catalog/Titles?$filter=Type%20eq%20'Movie'%20and%20(Rating%20eq%20'G'%20or%20Rating%20eq%20'PG-13')

This service is effectively coupled to a Table/View called `Titles` and a column called `Type`. You're also coupled to the OData binary implementation inhibiting future optimizations or the ability to move to an optimal implementation in future e.g. leveraging a Search Index to serve part or the entire request. Use of special operators illegal in C#/.NET variable names like `$filter` shows that OData makes a point of exposing an API that wouldn't be developed naturally, it's effectively an isolated technology stack processing queries tunneled within its own custom namespace.

### Out of band knowledge in Free-form expressions

In order to understand how to consume this service you also have to be familiar with [the OData specification](http://www.odata.org/documentation/odata-version-4-0/) which due to its size and complexity ensures OData queries effectively become an opaque text blob invisible to your application that can only be processed by an OData binary that understands the OData spec. This was one of the major disadvantages of SOAP and WS-* specifications which due to its size and complexity forced the use of a SOAP Framework to be able to create services as opposed to simple HTTP API's which allow any language on any platform with a HTTP Server to be able to create Web Services. A pitfall free-form expressions encourage is having knowledge and opinions baked into client libraries, eventually mandating the use of specific consumer implementations. This is avoided with well-defined API boundaries ensuring that any service, of any complexity, can be called with just a URL and the API's published message forms.

### Tight Coupling of Internals

This service also requires knowledge of the internal structure of Netflix's DB schema to know what table and columns to query, more importantly once an OData API for your data model is published and has clients binded to it in production, the DB schema effectively becomes frozen since the OData query-space can reference any table and any column that was exposed.

### The ideal impl-agnostic movie ratings API

In contrast, if you were to create the service without using OData it would something like:

> http://api.netflix.com/movies?ratings=G,PG-13

i.e. just capturing the actual intent of the query, leaves complete freedom in how to best service the request whilst retaining the ability to evolve the underlying implementation without breaking existing clients.

## Goals of Service Design

The primary benefits of Services are that they offer the highest level of software re-use, they're [Real Computers all the way down](http://mythz.servicestack.net/#messaging) retaining the ability to represent anything. Especially at this level, encapsulation and its external interactions are paramount which sees the [Service Layer as its most important Contract](http://stackoverflow.com/a/15369736/85785), constantly evolving to support new capabilities whilst serving and outliving its many consumers. 

Extra special attention should be given to Service design with the primary goals of exposing its capabilities behind [consistent and self-describing](http://stackoverflow.com/a/15941229/85785), intent-based [tell-dont-ask](http://pragprog.com/articles/tell-dont-ask) APIs - given its importance, it's not something that should be dictated by an internal implementation. 

A Services ability to encapsulate complexity is what empowers consumers to be able to perform higher-level tasks like provisioning a cluster of AWS servers or being able to send a tweet to millions of followers in seconds with just a simple HTTP request, i.e. being able to re-use existing hardened functionality without the required effort, resources and infrastructure to facilitate the request yourself. To maximize accessibility it's recommended for Service Interfaces to retain a flat structure, customizable with key value pairs so they're accessible via the built-in QueryString and FormData support present in all HTTP clients, from HTML Forms to command-line utilities like [curl](http://curl.haxx.se/).

## Unnecessary Complexity

Another reason we're opposed to considering technologies like OData is the sheer amount of unnecessary complexity of the implementation itself. Minimizing complexity is at the core essence of ServiceStack, it's why ServiceStack exists and remains the primary design goal in how features are implemented with the least complexity and cognitive overhead required.

### OData is Big

By contrast OData is comically large, there's literally an [entire Organization](http://www.odata.org/) created around it, sporting its own [blog](http://www.odata.org/blog/), [mailing list](http://www.odata.org/join-the-odata-discussion/), multiple [spec versions](http://www.odata.org/documentation/odata-version-4-0/) and [client libraries](http://www.odata.org/documentation/odata-version-4-0/) of which it appears only the Microsoft sponsored client libraries implement the latest v4 of the OData spec, as-is the nature of complicated rolling specs. 

Measuring size by weight shows `Microsoft.Data.OData.dll` alone weighs in at **1,287kb**, even more than `ServiceStack.dll`, surprising given [ServiceStack does a lot](https://servicestack.net/features). Include OData's required `Microsoft.Data.Edm.dll` and `System.Spatial.dll` NuGet dependencies and the payload increases another 50%, include integration with WebApi and client OData libraries and it bloats up further again.

### Why not Complexity

Imagine how much knowledge and cognitive overhead would be required to create a simple web app if every feature was over-engineered in this way? Every new feature introduces a complexity cost which is [why it's critically important](http://mythz.servicestack.net/#engineering) to ensure any complexity introduced [remains proportional](https://gist.github.com/cookrn/4015437) with the [needs being solved](http://worrydream.com/ABriefRantOnTheFutureOfInteractionDesign/). 

The needs in this case is [simplifying](http://www.infoq.com/presentations/Simple-Made-Easy) the creation and consumption of data-driven services, something which could easily have been implemented as a single feature point, has instead [been over-engineered](http://www.tele-task.de/player/embed/5819/0/?iframe) beyond belief and turned into something you can go on a training course and get certifications for! 

[Large implementations](http://steve-yegge.blogspot.com/2007/12/codes-worst-enemy.html) weakens our ability to reason about a system, to make informed decisions, to understand the impact of customization's and optimizations or identify the underlying cause of unintended behavior.

### Adding new Features without new Complexity

Rather than tacking on new libraries or inventing different ways/concepts/specs/dsl's for doing new things, features in ServiceStack are applied thoughtfully so they naturally integrate with its [existing architecture](https://github.com/ServiceStack/ServiceStack/wiki/Architecture-overview) maximizing re-use and leveraging existing functionality wherever possible, strengthening the existing mental model and ensuring new abstractions or concepts only get added for that of which is truly new.

## Introducing AutoQuery

The solution to overcome most of OData issues is ultimately quite simple: enhance the ideal API the developer would naturally write and complete their implementation for them! This is essentially the philosophy behind AutoQuery which utilizes conventions to automate creation of intent-based self-descriptive APIs that are able to specify configurable conventions and leverage extensibility options to maximize the utility of AutoQuery services.

AutoQuerying aims to work like optional typing by making it easy to expose contract-less data services for rapid prototyping, then allowing the API to be gradually formalized by decorating Request DTO's with its supported usage, whilst allowing complete freedom in either utilizing and extending AutoQuery's built-in functionality or replacing it entirely without breaking the Service Contract.

### AutoQuery Services are ServiceStack Services

An important point to highlight is that AutoQuery Services are just normal ServiceStack Services, utilizing the same [Request Pipeline](https://github.com/ServiceStack/ServiceStack/wiki/Order-of-Operations), which can be mapped to any [user-defined route](https://github.com/ServiceStack/ServiceStack/wiki/Routing), is available in all [registered formats](https://github.com/ServiceStack/ServiceStack/wiki/Formats) and can be [consumed from existing typed Service Clients](https://github.com/ServiceStack/ServiceStack/wiki/Clients-overview). 

In addition to leveraging ServiceStack's existing functionality, maximizing re-use in this way reduces the cognitive overhead required for developers who can re-use their existing knowledge in implementing, customizing, introspecting and consuming ServiceStack services. 

### Getting Started

Like ServiceStack's [other composable features](https://github.com/ServiceStack/ServiceStack/wiki/Plugins), the AutoQuery Feature is enabled by registering a Plugin, e.g:

```csharp
Plugins.Add(new AutoQueryFeature { MaxLimit = 100 });
```

Which is all that's needed to enable the AutoQuery feature. The AutoQueryFeature is inside [ServiceStack.Server](https://servicestack.net/download#get-started) NuGet package which contains value-added features that utilize either OrmLite and Redis which can be added to your project with:

    PM> Install-Package ServiceStack.Server

If you don't have OrmLite configured it can be registered with a 1-liner by specifying your preferred DialectProvider and Connection String:

```csharp
container.Register<IDbConnectionFactory>(
    new OrmLiteConnectionFactory(":memory:", SqliteDialect.Provider));
```

The above config registers an In Memory Sqlite database although as the AutoQuery test suite works in all [supported RDBMS providers](https://github.com/ServiceStack/ServiceStack.OrmLite/#download) you're free to use your registered DB of choice.

The `MaxLimit` option ensures each query returns a maximum limit of **100** rows. 

Now that everything's configured we can create our first service. To implement the ideal API for movie ratings above we just need to define the Request DTO for our service, i.e:

```csharp
[Route("/movies")]
public class FindMovies : QueryBase<Movie>
{
    public string[] Ratings { get; set; }
}
```

That's all the code needed! Which we can now call using the ideal route:

    /movies?ratings=G,PG-13

### Clients continue to benefit from a typed API

and because it's a just regular Request DTO, we also get an end-to-end typed API for free with:

```csharp
var movies = client.Get(new FindMovies { Ratings = new[]{"G","PG-13"} })
```

Whilst that gives us the ideal API we want, the minimum code required is to just declare a new Request DTO with the table you wish to query:

```csharp
public class FindMovies : QueryBase<Movie> {}
```

Which just like other Request DTO's in ServiceStack falls back to using ServiceStack's [pre-defined routes](https://github.com/ServiceStack/ServiceStack/wiki/Routing#pre-defined-routes)

`QueryBase<T>` is just an abstract class that implements `IQuery<T>` and returns a typed `QueryResponse<T>`:

```csharp
public abstract class QueryBase<T> : QueryBase, 
    IQuery<T>, IReturn<QueryResponse<T>> { }

public interface IQuery
{
    int? Skip { get; set; }
    int? Take { get; set; }
    string OrderBy { get; set; }
    string OrderByDesc { get; set; }
}
```

The `IQuery` interface includes shared features that all Queries support and acts as a interface marker telling ServiceStack to create query services for each Request DTO. 

### Custom AutoQuery Implementations

The behavior of queries can be completely customized by simply providing your own Service implementation instead, e.g:

```csharp
public class MyQueryServices : Service
{
    public IAutoQuery AutoQuery { get; set; }

    //Override with custom implementation
    public object Any(FindMovies request)
    {
        var q = AutoQuery.CreateQuery(request, Request.GetRequestParams());
        return AutoQuery.Execute(request, q);
    }
}
```

This is essentially what the AutoQuery feature generates for each `IQuery` Request DTO unless a Service for the Request DTO already exists, in which case it uses the existing implementation. 

We can inspect the 2 lines to understand how AutoQuery works:

```csharp
var q = AutoQuery.CreateQuery(dto, Request.GetRequestParams());
```

The first line constructs an [OrmLite SqlExpression](https://github.com/ServiceStack/ServiceStack.OrmLite/#examples) from typed properties on the Request DTO as well as any untyped key/value pairs on the HTTP Requests **QueryString** or **FormData**.

At this point you can inspect the SqlExpression that AutoQuery has constructed or append any additional custom criteria to limit the scope of the request like limiting results to the authenticated user.

After constructing the query from the Request all that's left is executing it:

```csharp
return AutoQuery.Execute(dto, q);
```

As the implementation is trivial we can show the implementation inline:

```csharp
public QueryResponse<Into> Execute<Into>(
    IDbConnection db, ISqlExpression query)
{
    var q = (SqlExpression<From>)query;
    return new QueryResponse<Into>
    {
        Offset = q.Offset.GetValueOrDefault(0),
        Total = (int)db.Count(q),
        Results = db.LoadSelect<Into, From>(q),
    };
}
```

Basically just returns the results in an `QueryResponse<Into>` Response DTO. We also see that each query makes 2 requests, 1 for retrieving the total count for the query and another for the actual results, either limited by the request's `Skip` or `Take` or the configured `AutoQueryFeature.MaxLimit`.  

## Returning Custom Results

To specify returning results in a custom Response DTO you can inherit from the `QueryBase<From, Into>` convenience base class:

```csharp
public abstract class QueryBase<From, Into> 
    : QueryBase, IQuery<From, Into>, IReturn<QueryResponse<Into>> { }
```

As we can tell from the class definition this tells AutoQuery you want to query against `From` but return results into the specified `Into` type. This allows being able to return a curated set of columns, e.g:

```csharp
public class QueryCustomRockstars : QueryBase<Rockstar, CustomRockstar> {}

public class CustomRockstar
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public int? Age { get; set; }
    public string RockstarAlbumName { get; set; }
}
```

In the example above we're returning only a subset of results. Unmatched properties like `RockstarAlbumName` are ignored enabling the re-use of custom DTO's for different queries.

## Returning Nested Related Results

AutoQuery also takes advantage of [OrmLite's References Support](https://github.com/ServiceStack/ServiceStack.OrmLite/#reference-support-poco-style) which lets you return related child records that are annotated with `[Reference]` attribute, e.g:

```csharp
public class QueryRockstars : QueryBase<Rockstar> {}

public class Rockstar
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public int? Age { get; set; }

    [Reference]
    public List<RockstarAlbum> Albums { get; set; } 
}
```

## Joining Tables

AutoQuery lets us take advantage of OrmLite's recent [support for JOINs in typed SqlExpressions](https://github.com/ServiceStack/ServiceStack.OrmLite/#typed-sqlexpression-support-for-joins) 

We can tell AutoQuery to join on multiple tables using the `IJoin<T1,T2>` interface marker:

```csharp
public class QueryRockstarAlbums 
  : QueryBase<Rockstar,CustomRockstar>, IJoin<Rockstar,RockstarAlbum>
{
    public int? Age { get; set; }
    public string RockstarAlbumName { get; set; }
}
```

The above example tells AutoQuery to query against an **INNER JOIN** of the `Rockstar` and `RockstarAlbum` tables using [OrmLite's reference conventions](https://github.com/ServiceStack/ServiceStack.OrmLite/#reference-conventionst) that's implicit between both tables.

The Request DTO lets us query against fields across the joined tables where each field is matched with the [first table containing the field](https://github.com/ServiceStack/ServiceStack.OrmLite/#selecting-multiple-columns-across-joined-tables). You can match against fields using the fully qualified `{Table}{Field}` convention, e.g. `RockstarAlbumName` queries against the `RockstarAlbum`.`Name` column.

This mapping of fields also applies to the Response DTO where now `RockstarAlbumName` from the above `CustomRockstar` type will be populated:

```csharp
public class CustomRockstar
{
    ...
    public string RockstarAlbumName { get; set; }
}
```

### Joining multiple tables

Use the appropriate `Join<,>` generic interface to specify JOINs across multiple tables. AutoQuery supports joining up to 5 tables with 1 `Join<>` interface, e.g:

```csharp
IJoin<T1, T2, T3, T4, T5>
```

Or alternatively any number of tables can be joined together by annotating the Request DTO with multiple `IJoin<>` interfaces, e.g:

```csharp
IJoin<T1,T2>,
IJoin<T2,T3>,
IJoin<T3,T4>,
//...
```

### Support for LEFT JOINS

You can tell AutoQuery to use a **LEFT JOIN** by using the `ILeftJoin<,>` marker interface instead, e.g:

```csharp
public class QueryRockstarAlbumsLeftJoin : QueryBase<Rockstar, CustomRockstar>, 
    ILeftJoin<Rockstar, RockstarAlbum>
{
    public int? Age { get; set; }
    public string AlbumName { get; set; }
}
```

## Customizable Adhoc Queries

The queries up to this point showcase the default behavior of Request DTO fields performing an **Exact Match** using the `=` operand to match fields on queried tables. Advanced queries are available via configurable attributes illustrated in the example below which uses explicit templates to enable custom querying, e.g:

```csharp
public class QueryRockstars : QueryBase<Rockstar>
{
    //Defaults to 'AND FirstName = {Value}'
    public string FirstName { get; set; }  

    //Collections defaults to 'FirstName IN ({Values})'
    public string[] FirstNames { get; set; } 

    [QueryField(Operand = ">=")]
    public int? Age { get; set; }

    [QueryField(Template = "UPPER({Field}) LIKE UPPER({Value})", 
                Field = "FirstName")]
    public string FirstNameCaseInsensitive { get; set; }

    [QueryField(Template="{Field} LIKE {Value}", 
                Field="FirstName", ValueFormat="{0}%")]
    public string FirstNameStartsWith { get; set; }

    [QueryField(Template="{Field} LIKE {Value}", 
                Field="LastName", ValueFormat="%{0}")]
    public string LastNameEndsWith { get; set; }

    [QueryField(Template = "{Field} BETWEEN {Value1} AND {Value2}", 
                Field="FirstName")]
    public string[] FirstNameBetween { get; set; }

    [QueryField(Type = QueryTerm.Or)]
    public string LastName { get; set; }
}
```

We'll go through each of the examples to give a better idea of the customizations available.

### Queries on Collections

ServiceStack properties are case-insensitive and allows populating a collection with comma-delimited syntax, so this query:

    /rockstars?firstNames=A,B,C

Will populate the string array:

```csharp
public string[] FirstNames { get; set; } 
```

That by default `IEnumerable` properties are queried using an SQL `{Field} IN ({Values})` template which will convert the string array into a quoted and escaped comma-delimited list of values suitable for use in SQL that looks like:

```sql
"FirstName" IN ('A','B','C')
```

The `{Field}` property is substituted according to the OrmLite FieldDefinition it matches, which for a Joined table with an `[Alias("FName")]` attribute would substitute to a fully-qualified name, e.g: `"Table"."FName"`

AutoQuerying also supports pluralized versions by trimming the `s` off Field names as a fall-back which is how `FirstNames` was able to match the `FirstName` table property.

### Customizable Operands

One of the easiest customization's available is changing the operand used in the query, e.g:

```csharp 
[QueryField(Operand = ">=")]
public int? Age { get; set; }
```

Will change how the Age is compared to:

```sql
"Age" >= {Value}
```

### Customizable Templates

By providing a customized template  even greater customization achieved and by sticking with TSQL compatible fragments will ensure it's supported by every RDBMS provider, e.g: 

```csharp
[QueryField(Template = "UPPER({Field}) LIKE UPPER({Value})",
            Field="FirstName")]
public string FirstNameCaseInsensitive { get; set; }
```

As noted above `{Field}` variable place holder is substituted with the qualified Table field whilst `{Value}` gets quoted and escaped to prevent SQL Injections.

The `Field="FirstName"` tells AutoQuery to ignore the property name and use the specified field name.

### Formatting Values

You can use the `ValueFormat` modifier to insert custom characters within the quoted `{Value}` placeholder which we use to insert the `%` wildcard in with the quoted value enabling StartsWith and EndsWith queries, e.g:

```csharp
[QueryField(Template="{Field} LIKE {Value}", 
            Field="FirstName", ValueFormat="{0}%")]
public string FirstNameStartsWith { get; set; }

[QueryField(Template="{Field} LIKE {Value}", 
            Field="LastName", ValueFormat="%{0}")]
public string LastNameEndsWith { get; set; }
```

### Specifying Multi Airity Queries

An alternate way to format multiple values is to use `Value{N}` variable placeholders which allows supporting statements with multiple values like SQL's **BETWEEN** statement:

```csharp
[QueryField(Template = "{Field} BETWEEN {Value1} AND {Value2}", 
            Field = "FirstName")]
public string[] FirstNameBetween { get; set; }
```

### Changing Querying Behavior

By default queries act like a filter and every condition is combined with **AND** boolean term to further filter the result-set. This can be changed to use an **OR** at the field-level by specifying `Term=QueryTerm.Or` modifier, e.g:

```csharp
[QueryField(Term=QueryTerm.Or)]
public string LastName { get; set; }
```

However as your API's should strive to [retain the same behavior and call-semantics](http://stackoverflow.com/questions/15927475/servicestack-request-dto-design/15941229#15941229) the recommendation is to instead change the behavior at the API-level so that each property maintains consistent behavior, e.g:

```csharp
[Query(QueryTerm.Or)]
public class QueryRockstars : QueryBase<Rockstar>
{
    public int[] Ids { get; set; }
    public List<int> Ages { get; set; }
    public List<string> FirstNames { get; set; }
}
```

In this example each property is inclusive where every value specified is added to the returned result-set. 

### Dynamic Attributes

Whilst .NET attributes are normally defined with the property they attribute, ServiceStack also allows attributes to be added dynamically allowing them to be defined elsewhere, detached from the DTO's, e.g:

```csharp
typeof(QueryRockstars)
    .GetProperty("Age")
    .AddAttributes(new QueryFieldAttribute { Operand = ">=" });
```

## Implicit Conventions

The examples above show how we can define adhoc **explicit** queries on concrete properties using explicit attributes. Whilst this was necessary to understand how to create customized queries, the concrete properties and explicit attributes can be avoided entirely by utilizing the built-in conventions, configurable on the `AutoQueryFeature` plugin.

With implicit conventions we can get back to defining Request DTO's with an empty body:

```csharp
public class QueryRockstars : QueryBase<Rockstar> {}
```

The built-in conventions allow using convention-based naming to query fields with expected behavior using self-describing properties. To explore this in more detail lets look at what built-in conventions are defined:

```csharp
const string GreaterThanOrEqualFormat = "{Field} >= {Value}";
const string GreaterThanFormat =        "{Field} >  {Value}";
const string LessThanFormat =           "{Field} <  {Value}";
const string LessThanOrEqualFormat =    "{Field} <= {Value}";
const string NotEqualFormat =           "{Field} <> {Value}";

ImplicitConventions = new Dictionary<string, string> 
{
    {"%Above%",         GreaterThanFormat},
    {"Begin%",          GreaterThanFormat},
    {"%Beyond%",        GreaterThanFormat},
    {"%Over%",          GreaterThanFormat},
    {"%OlderThan",      GreaterThanFormat},
    {"%After%",         GreaterThanFormat},
    {"OnOrAfter%",      GreaterThanOrEqualFormat},
    {"%From%",          GreaterThanOrEqualFormat},
    {"Since%",          GreaterThanOrEqualFormat},
    {"Start%",          GreaterThanOrEqualFormat},
    {"%Higher%",        GreaterThanOrEqualFormat},
    {">%",              GreaterThanOrEqualFormat},
    {"%>",              GreaterThanFormat},
    {"%!",              NotEqualFormat},

    {"%GreaterThanOrEqualTo%", GreaterThanOrEqualFormat},
    {"%GreaterThan%",          GreaterThanFormat},
    {"%LessThan%",             LessThanFormat},
    {"%LessThanOrEqualTo%",    LessThanOrEqualFormat},
    {"%NotEqualTo",            NotEqualFormat},

    {"Behind%",         LessThanFormat},
    {"%Below%",         LessThanFormat},
    {"%Under%",         LessThanFormat},
    {"%Lower%",         LessThanFormat},
    {"%Before%",        LessThanFormat},
    {"%YoungerThan",    LessThanFormat},
    {"OnOrBefore%",     LessThanOrEqualFormat},
    {"End%",            LessThanOrEqualFormat},
    {"Stop%",           LessThanOrEqualFormat},
    {"To%",             LessThanOrEqualFormat},
    {"Until%",          LessThanOrEqualFormat},
    {"%<",              LessThanOrEqualFormat},
    {"<%",              LessThanFormat},

    {"Like%",           "UPPER({Field}) LIKE UPPER({Value})"},
    {"%In",             "{Field} IN ({Values})"},
    {"%Ids",            "{Field} IN ({Values})"},
    {"%Between%",       "{Field} BETWEEN {Value1} AND {Value2}"},
};

EndsWithConventions = new Dictionary<string, QueryFieldAttribute>
{
    { "StartsWith", new QueryFieldAttribute { 
          Template= "UPPER({Field}) LIKE UPPER({Value})", 
          ValueFormat= "{0}%" }},
    { "Contains", new QueryFieldAttribute { 
          Template= "UPPER({Field}) LIKE UPPER({Value})", 
          ValueFormat= "%{0}%" }},
    { "EndsWith", new QueryFieldAttribute { 
          Template= "UPPER({Field}) LIKE UPPER({Value})", 
          ValueFormat= "%{0}" }},
};
```

The string key in the above dictionaries describe what property each rule maps to. We'll run through a few examples to see how we can make use of them:

The request below works as you would expect in returning all Rockstars older than 42 years of age.

    /rockstars?AgeOlderThan=42

It works by matching on the rule since `AgeOlderThan` does indeed ends with `OlderThan`:

```csharp
{"%OlderThan", "{Field} > {Value}"},
```

The `%` wildcard is a placeholder for the field name which resolves to `Age`. Now that it has a match it creates a query with the defined `{Field} > {Value}` template to achieve the desired behavior.

If you instead wanted to use the inclusive `>=` operand, we can just use a rule with the `>=` template:

    /rockstars?AgeGreaterThanOrEqualTo=42

Which matches the rule:

```csharp
{"%GreaterThanOrEqualTo%", "{Field} >= {Value}"},
```

And when the wildcard is on both ends of the pattern:

```csharp
{"%GreaterThan%", "{Field} > {Value}"},
```

So to can the field name, which matches both these rules:

    /rockstars?AgeGreaterThan=42
    /rockstars?GreaterThanAge=42

An alternative to using wordy suffixes are the built-in short-hand syntax:

 - **>Age**  `Age >= {Value}`
 - **Age>**  `Age >  {Value}`
 - **<Age**  `Age <  {Value}`
 - **Age<**  `Age <= {Value}`

Which uses the appropriate operand based on whether the `<` `>` operators come before or after the field name:

    /rockstars?>Age=42
    /rockstars?Age>=42
    /rockstars?<Age=42
    /rockstars?Age<=42

The use of `{Values}` or the `Value{N}` formats specifies the query should be treated as a collection, e.g:

```csharp
{"%Ids",       "{Field} IN ({Values})"},
{"%In",        "{Field} IN ({Values})"},
{"%Between%",  "{Field} BETWEEN {Value1} AND {Value2}"},
```

Which allows multiple values to be specified on the QueryString:

    /rockstars?Ids=1,2,3
    /rockstars?FirstNamesIn=Jim,Kurt
    /rockstars?FirstNameBetween=A,F

More advanced conventions can be specified directly on the `StartsWithConventions` and `EndsWithConventions` dictionaries which allow customizations using the full `[QueryField]` attribute, e.g:

```csharp
EndsWithConventions = new Dictionary<string, QueryFieldAttribute>
{
    { "StartsWith", new QueryFieldAttribute { 
          Template = "UPPER({Field}) LIKE UPPER({Value})", 
          ValueFormat = "{0}%" }},
    { "Contains", new QueryFieldAttribute { 
          Template = "UPPER({Field}) LIKE UPPER({Value})", 
          ValueFormat = "%{0}%" }},
    { "EndsWith", new QueryFieldAttribute { 
          Template = "UPPER({Field}) LIKE UPPER({Value})", 
          ValueFormat = "%{0}" }},
};
```

that can be called as normal:

    /rockstars?FirstNameStartsWith=Jim
    /rockstars?LastNameEndsWith=son
    /rockstars?RockstarAlbumNameContains=e

This also shows that Implicit Conventions can also apply to joined table fields like `RockstarAlbumNameContains` using the fully qualified `{Table}{Field}` reference convention.

## Explicit Conventions

Whilst Implicit conventions can be called on an empty "contract-less" DTO, you're also able to specify the concrete properties for the conventions you end up using: 

```csharp
public class QueryRockstars : QueryBase<Rockstar>
{
    public int? AgeOlderThan { get; set; }
    public int? AgeGreaterThanOrEqualTo { get; set; }
    public int? AgeGreaterThan { get; set; }
    public int? GreaterThanAge { get; set; }
    public string FirstNameStartsWith { get; set; }
    public string LastNameEndsWith { get; set; }
    public string LastNameContains { get; set; }
    public string RockstarAlbumNameContains { get; set; }
    public int? RockstarIdAfter { get; set; }
    public int? RockstarIdOnOrAfter { get; set; }
}
```

### Preemptive client requests

A hidden gem in AutoQuery's approach that's not immediately obvious is how everything will still work when concrete properties are only added to client Request DTO's that the server doesn't even know about yet! This is possible as the service client generates the same HTTP Request that matches the implicit conventions, making it possible to perform new preemptive typed queries on the client without having to redeploy the server. Although unlikely to be a popular use-case given that in most cases the client and server shares the same DTOs, it's a nice surprise feature when needed.

### Advantages of well-defined Service Contracts

The advantages of formalizing the conventions you end up using is that they can be consumed with ServiceStack's [Typed Service Clients](https://github.com/ServiceStack/ServiceStack/wiki/C%23-client):

```csharp
var response = client.Get(new QueryRockstars { AgeOlderThan = 42 });
```

As well as making your API self-describing and gives you access to all of ServiceStack's metadata services including:

 - [Metadata pages](https://github.com/ServiceStack/ServiceStack/wiki/Metadata-page)
 - [Swagger UI](https://github.com/ServiceStack/ServiceStack/wiki/Swagger-API)
 - [Postman Support](https://github.com/ServiceStack/ServiceStack/wiki/Postman)

Which is why we recommend formalizing your conventions you want to allow before deploying your API to production. 

When publishing your API, you can also assert that only explicit conventions are ever used by disabling untyped implicit conventions support with:

```csharp
Plugins.Add(new AutoQueryFeature { EnableUntypedQueries = false });
```

## Raw SQL Filters

It's also possible to specify Raw SQL Filters. Whilst offering greater flexibility they also suffer from many of the problems that OData's expressions have. 

Raw SQL Filters also don't benefit from limiting matching to declared fields and auto-escaping and quoting of values although they're still validated against OrmLite's [IllegalSqlFragmentTokens](https://github.com/ServiceStack/ServiceStack.OrmLite/blob/master/src/ServiceStack.OrmLite/OrmLiteUtilExtensions.cs#L172) to protect against SQL Injection attacks. But as they still allow calling SQL Functions, Raw SqlFilters shouldn't be enabled when accessible by untrusted parties unless they've been configured to use a [Named OrmLite DB Connection](https://github.com/ServiceStack/ServiceStack.OrmLite/#multi-nested-database-connections) which is read-only and locked-down so that it only has access to what it's allowed to.

If safe to do so, RawSqlFilters can be enabled with:

```csharp
Plugins.Add(new AutoQueryFeature { 
    EnableRawSqlFilters = true,
    UseNamedConnection = "readonly",
    IllegalSqlFragmentTokens = { "ProtectedSqlFunction" },
});
```

Once enabled you can use the `_select`, `_from`, `_where` modifiers to append custom SQL Fragments, e.g:

    /rockstars?_select=FirstName,LastName

    /rockstars?_where=Age > 42
    /rockstars?_where=FirstName LIKE 'Jim%'

    /rockstars?_select=FirstName&_where=LastName = 'Cobain'

    /rockstars?_from=Rockstar r INNER JOIN RockstarAlbum a ON r.Id = a.RockstarId

> Note: url encoding in the above examples are omitted for readability

Whilst Raw SqlFilters affect the query executed, they don't change what Response DTO is returned.

## Paging and Ordering

Some functionality defined on the core `IQuery` interface and available to all queries is the ability to page and order results:

```csharp
public interface IQuery
{
    int? Skip { get; set; }
    int? Take { get; set; }
    string OrderBy { get; set; }
    string OrderByDesc { get; set; }
}
```

This works as you would expect where you can modify the returned result set with:

    /rockstars?skip=10&take=20&orderBy=Id

Or when accessing via a ServiceClient:

```csharp
client.Get(new QueryRockstars { Skip=10, Take=20, OrderBy="Id" });
``` 

When results are paged an implicit OrderBy is added using the PrimaryKey of the `IQuery<Table>` to ensure predictable ordering of results are returned. You can change the OrderBy used by specifying your own: 

```csharp
client.Get(new QueryRockstars { Skip=10, Take=20, OrderByDesc="Id" });
``` 

Or remove the default behavior with:

```csharp
Plugins.Add(new AutoQueryFeature { 
    OrderByPrimaryKeyOnPagedQuery = false 
});
```

### Multiple OrderBy's

AutoQuery also supports specifying multiple OrderBy's with a comma-delimited list of field names, e.g:

    /rockstars?orderBy=Id,Age,FirstName

Same request via Service Client:

```csharp
client.Get(new QueryRockstars { OrderBy = "Id,Age,FirstName" });
```

#### Sort specific fields in reverse order

When specifying multiple order by's you can sort specific fields in reverse order by prepending a `-` before the field name, e.g:

    ?orderBy=Id,-Age,FirstName
    ?orderByDesc=-Id,Age,-FirstName

## Service Clients Support

One of the major benefits of using Typed DTO's to define your Service Contract is that it allows usage of ServiceStack's [.NET Service Clients](https://github.com/ServiceStack/ServiceStack/wiki/C%23-client) which enables an end-to-end API without code-gen for [most .NET and PCL client platforms](https://github.com/ServiceStack/Hello). 

With the richer semantics available in queries, we've been able to enhance the Service Clients new `GetLazy()` API that allows lazy streaming of responses to provide transparent paging of large result-sets, e.g:

```csharp
var results = client.GetLazy(new QueryMovies { 
    Ratings = new[]{"G","PG-13"}
}).ToList();
```

Since GetLazy returns a lazy `IEnumerable<T>` sequence it can also be used within LINQ expressions, e.g:

```csharp
var top250 = client.GetLazy(new QueryMovies { 
        Ratings = new[]{ "G", "PG-13" } 
    })
    .Take(250)
    .ConvertTo(x => x.Title);
```

## Mime Types and Content-Negotiation

Another benefit we get from AutoQuery Services being regular ServiceStack services is taking advantage of [ServiceStack's built-in formats](https://github.com/ServiceStack/ServiceStack/wiki/Formats). 

The [CSV Format](https://github.com/ServiceStack/ServiceStack/wiki/ServiceStack-CSV-Format) especially shines here given queries return a single tabular resultset making it perfect for CSV. In many ways CSV is one of the most interoperable Data Formats given most data import and manipulation programs including Databases and Spreadsheets have native support for CSV allowing for deep and seamless integration.

ServiceStack provides a number of ways to [request your preferred content-type](https://github.com/ServiceStack/ServiceStack/wiki/Routing#content-negotiation), the easiest of which is to just use the `.{format}` extension at the end of the `/pathinfo` e.g:

    /rockstars.csv
    /movies.csv?ratings=G,PG-13

## Include Aggregates in AutoQuery

AutoQuery supports running additional Aggregate queries on the queried result-set. To include aggregates in your Query's response specify them in the `Include` property of your AutoQuery Request DTO, e.g:

    var response = client.Get(new Query Rockstars { Include = "COUNT(*)" })

Or in the `Include` QueryString param if you're calling AutoQuery Services from a browser, e.g:

    /rockstars?include=COUNT(*)

The result is then published in the `QueryResponse<T>.Meta` String Dictionary and is accessible with:

    response.Meta["COUNT(*)"] //= 7

By default any of the functions in the SQL Aggregate whitelist can be referenced: 

    AVG, COUNT, FIRST, LAST, MAX, MIN, SUM

Which can be added to or removed from by modifying `SqlAggregateFunctions` collection, e.g, you can allow usage of a `CustomAggregate` SQL Function with:

    Plugins.Add(new AutoQueryFeature { 
        SqlAggregateFunctions = { "CustomAggregate" }
    })

### Aggregate Query Usage

The syntax for aggregate functions is modelled after their usage in SQL so they're instantly familiar. At its most basic usage you can just specify the name of the aggregate function which will use `*` as a default argument so you can also query `COUNT(*)` with: 

    ?include=COUNT

It also supports SQL aliases:

    COUNT(*) Total
    COUNT(*) as Total

Which is used to change what key the result is saved into:

    response.Meta["Total"]

Columns can be referenced by name:

    COUNT(LivingStatus)

If an argument matches a column in the primary table the literal reference is used as-is, if it matches a column in a joined table it's replaced with its fully-qualified reference and when it doesn't match any column, Numbers are passed as-is otherwise its automatically escaped and quoted and passed in as a string literal.

The `DISTINCT` modifier can also be used, so a complex example looks like:

    COUNT(DISTINCT LivingStatus) as UniqueStatus

Which saves the result of the above function in:

    response.Meta["UniqueStatus"]

Any number of aggregate functions can be combined in a comma-delimited list:

    Count(*) Total, Min(Age), AVG(Age) AverageAge

Which returns results in:

    response.Meta["Total"]
    response.Meta["Min(Age)"]
    response.Meta["AverageAge"]

#### Aggregate Query Performance

Surprisingly AutoQuery is able to execute any number of Aggregate functions without performing any additional queries as previously to support paging, a `Total` needed to be executed for each AutoQuery. Now the `Total` query is combined with all other aggregate functions and executed in a single query.

### AutoQuery Response Filters

The Aggregate functions feature is built on the new `ResponseFilters` support in AutoQuery which provides a new extensibility option enabling customization and additional metadata to be attached to AutoQuery Responses. As the Aggregate Functions support is itself a Response Filter in can disabled by clearing them:

```csharp
Plugins.Add(new AutoQueryFeature {
    ResponseFilters = new List<Action<QueryFilterContext>>()
})
```

The Response Filters are executed after each AutoQuery and gets passed the full context of the executed query, i.e:

```csharp
class QueryFilterContext
{
    IDbConnection Db             // The ADO.NET DB Connection
    List<Command> Commands       // Tokenized list of commands
    IQuery Request               // The AutoQuery Request DTO
    ISqlExpression SqlExpression // The AutoQuery SqlExpression
    IQueryResponse Response      // The AutoQuery Response DTO
}
```

Where the `Commands` property contains the parsed list of commands from the `Include` property, tokenized into the structure below:

```csharp
class Command 
{
    string Name
    List<string> Args
    string Suffix
}
```

With this we could add basic calculator functionality to AutoQuery with the custom Response Filter below:

```csharp
Plugins.Add(new AutoQueryFeature {
    ResponseFilters = {
        ctx => {
            var supportedFns = new Dictionary<string, Func<int, int, int>>(StringComparer.OrdinalIgnoreCase)
            {
                {"ADD",      (a,b) => a + b },
                {"MULTIPLY", (a,b) => a * b },
                {"DIVIDE",   (a,b) => a / b },
                {"SUBTRACT", (a,b) => a - b },
            };
            var executedCmds = new List<Command>();
            foreach (var cmd in ctx.Commands)
            {
                Func<int, int, int> fn;
                if (!supportedFns.TryGetValue(cmd.Name, out fn)) continue;
                var label = !string.IsNullOrWhiteSpace(cmd.Suffix) ? cmd.Suffix.Trim() : cmd.ToString();
                ctx.Response.Meta[label] = fn(int.Parse(cmd.Args[0]), int.Parse(cmd.Args[1])).ToString();
                executedCmds.Add(cmd);
            }
            ctx.Commands.RemoveAll(executedCmds.Contains);
        }        
    }
})
```

Which now lets users perform multiple basic arithmetic operations with any AutoQuery request!

```csharp
var response = client.Get(new QueryRockstars {
    Include = "ADD(6,2), Multiply(6,2) SixTimesTwo, Subtract(6,2), divide(6,2) TheDivide"
});

response.Meta["ADD(6,2)"]      //= 8
response.Meta["SixTimesTwo"]   //= 12
response.Meta["Subtract(6,2)"] //= 4
response.Meta["TheDivide"]     //= 3
```

### Untyped SqlExpression

If you need to introspect or modify the executed `ISqlExpression`, it’s useful to access it as a `IUntypedSqlExpression` so its non-generic API's are still accessible without having to convert it back into its concrete generic `SqlExpression<T>` Type, e.g:

```csharp
IUntypedSqlExpression q = ctx.SqlExpression.GetUntypedSqlExpression()
    .Clone();
```

> Cloning the SqlExpression allows you to modify a copy that won't affect any other Response Filter.

### AutoQuery Property Mapping

AutoQuery can map `[DataMember]` property aliases on Request DTO's to the queried table, e.g:

```csharp
public class QueryPerson : QueryBase<Person>
{
    [DataMember(Name = "first_name")]
    public string FirstName { get; set; }
}

public class Person
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
}
```

Which can be queried with:

    ?first_name=Jimi

or by setting the global `JsConfig.EmitLowercaseUnderscoreNames=true` convention:

```csharp
public class QueryPerson : QueryBase<Person>
{
    public string LastName { get; set; }
}
```

Where it's also queryable with:

    ?last_name=Hendrix

## Extensibility with QueryFilters

We've already covered some of extensibility options with Customizable **QueryFields** and **Implicit Conventions**, the most customizable would be to override the default implementation with a custom one, e.g:

```csharp
public class MyQueryServices : Service
{
    public IAutoQuery AutoQuery { get; set; }

    //Override with custom implementation
    public object Any(FindMovies dto)
    {
        var q = AutoQuery.CreateQuery(dto, Request.GetRequestParams());
        return AutoQuery.Execute(request, q);
    }
}
```

There's also a lighter weight option by registering a typed Query Filter, e.g:

```csharp
var autoQuery = new AutoQueryFeature()
  .RegisterQueryFilter<QueryRockstarsFilter, Rockstar>((req, q, dto) =>
      q.And(x => x.LastName.EndsWith("son"))
  )
  .RegisterQueryFilter<IFilterRockstars, Rockstar>((req, q, dto) =>
      q.And(x => x.LastName.EndsWith("son"))
  );

Plugins.Add(autoQuery);
```

Registering an interface like `IFilterRockstars` is especially useful as it enables applying custom logic to a number of different Query Services sharing the same interface. 

## Intercept and Introspect Every Query

As mentioned earlier AutoQuery works by auto-generating ServiceStack Services for each Request DTO marked with `IQuery` unless an custom implementation has been defined for that Query. It's also possible to change the base class for the generated services so that all queries executes your custom implementation instead.

To do this, create a custom Service that inherits from `AutoQueryServiceBase` and overrides both `Exec` methods with your own implementation, e.g:

```csharp
public abstract class MyAutoQueryServiceBase : AutoQueryServiceBase
{
    public override object Exec<From>(IQuery<From> dto)
    {
        var q = AutoQuery.CreateQuery(dto, Request.GetRequestParams());
        return AutoQuery.Execute(dto, q);
    }

    public override object Exec<From, Into>(IQuery<From, Into> dto)
    {
        var q = AutoQuery.CreateQuery(dto, Request.GetRequestParams());
        return AutoQuery.Execute(dto, q);
    }
}
```

Then tell AutoQuery to use your base class instead, e.g:

```csharp
Plugins.Add(new AutoQueryFeature { 
    AutoQueryServiceBaseType = typeof(MyAutoQueryServiceBase)
});
```

Which will now get AutoQuery to execute your implementations instead.


# AutoQuery Examples

## [Northwind](https://github.com/ServiceStackApps/Northwind)

> Northwind database viewer, showing how to easily expose read and cached view services of an internal dataset with ServiceStack + OrmLite

[![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/northwind.png)](http://northwind.servicestack.net)

#### Features

 - [AutoQuery](https://github.com/ServiceStack/ServiceStack/wiki/Auto-Query)
   - [Northwind AutoQuery DTOs](https://github.com/ServiceStackApps/Northwind/blob/master/src/Northwind/Northwind.ServiceModel/AutoQuery.cs)
   - [OrmLite Sqlite](https://github.com/ServiceStack/ServiceStack.OrmLite#download)

## [StackApis](https://github.com/ServiceStackApps/StackApis)

> AngularJS Single Page App leveraging AutoQuery in <50 lines of JavaScript + 1 AutoQuery DTO 

[![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/stackapis.png)](http://stackapis.servicestack.net)

#### Features

 - [AutoQuery](https://github.com/ServiceStack/ServiceStack/wiki/Auto-Query)
   - [StackApis AutoQuery DTO](https://github.com/ServiceStackApps/StackApis#stackapis-autoquery-service)
   - [OrmLite Sqlite](https://github.com/ServiceStack/ServiceStack.OrmLite#download)

## [AutoQuery Viewer](https://github.com/ServiceStackApps/AutoQueryViewer)

AutoQuery Viewer is a native iPad App that provides an automatic UI for browsing, inspecting and querying any publicly accessible [ServiceStack AutoQuery Service](https://github.com/ServiceStack/ServiceStack/wiki/Auto-Query) from an iPad. 

[![AutoQuery Viewer on AppStore](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/wikis/autoquery/autoqueryviewer-appstore.png)](https://itunes.apple.com/us/app/autoquery-viewer/id968625288?ls=1&mt=8)

#### Features

  - [TechStacks AutoQuery DTOs](https://github.com/ServiceStackApps/AutoQueryViewer#techstacks-autoquery-reqeust-dtos)  
  - [GitHub AutoQuery DTOs](https://github.com/ServiceStackApps/AutoQueryViewer#githubautoquery-request-dtos)  
  - [Stack Apis AutoQuery DTO](https://github.com/ServiceStackApps/AutoQueryViewer#stakapi-autoquery-request-dto)  

## [TechStacks Cocoa OSX Desktop App](https://github.com/ServiceStackApps/TechStacksDesktopApp)

TechStacks OSX Desktop App is built around 2 AutoQuery Services showing how much querying functionality [AutoQuery Services](https://github.com/ServiceStack/ServiceStack/wiki/Auto-Query) provides for free and how easy they are to call with [ServiceStack's new support for Swift and XCode](https://github.com/ServiceStack/ServiceStack/wiki/Swift-Add-ServiceStack-Reference).

[![TechStack Desktop Search Fields](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/release-notes/techstacks-desktop-field.png)](https://github.com/ServiceStackApps/TechStacksDesktopApp)
