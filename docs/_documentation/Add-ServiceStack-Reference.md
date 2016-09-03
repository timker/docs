---
#File header for Jekyll to pick up 
---
 ServiceStack's **Add ServiceStack Reference** feature allows adding generated Native Types for the most popular typed languages and client platforms directly from within most major IDE's starting with [ServiceStackVS](https://github.com/ServiceStack/ServiceStack/wiki/Creating-your-first-project#step-1-download-and-install-servicestackvs) - providing a simpler, cleaner and more versatile alternative to WCF's **Add Service Reference** feature that's built into VS.NET. 

Add ServiceStack Reference now supports 
[Swift](https://github.com/ServiceStack/ServiceStack/wiki/Swift-Add-ServiceStack-Reference), 
[Java](https://github.com/ServiceStack/ServiceStack/wiki/Java-Add-ServiceStack-Reference), 
[Kotlin](https://github.com/ServiceStack/ServiceStack/wiki/Kotlin-Add-ServiceStack-Reference), 
[C#](https://github.com/ServiceStack/ServiceStack/wiki/CSharp-Add-ServiceStack-Reference), 
[TypeScript](https://github.com/ServiceStack/ServiceStack/wiki/TypeScript-Add-ServiceStack-Reference), 
[F#](https://github.com/ServiceStack/ServiceStack/wiki/FSharp-Add-ServiceStack-Reference) and 
[VB.NET](https://github.com/ServiceStack/ServiceStack/wiki/VB.Net-Add-ServiceStack-Reference) 
including integration with most leading IDE's to provide a flexible alternative than sharing your DTO assembly with clients. Clients can now easily add a reference to a remote ServiceStack url and update DTOs directly from within VS.NET, Xcode, Android Studio, IntelliJ and Eclipse. We plan on expanding on this foundation into adding seamless, typed, end-to-end integration with other languages - Add a [feature request for your favorite language](http://servicestack.uservoice.com/forums/176786-feature-requests) to prioritize support for it sooner!

Our goal with Native Types is to provide an alternative for sharing DTO dlls, that can enable a better dev workflow for external clients who are now able to generate (and update) Typed APIs for your Services from a remote url within their favorite IDE - reducing the burden and effort required to consume ServiceStack Services whilst benefiting from clients native language strong-typing feedback.

ServiceStackVS offers the generation and updating of these clients through the same context for all supported languages giving developers a consistent way of creating and updating your DTOs regardless of their preferred language of choice.

### Supported Languages

* [C# Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/CSharp-Add-ServiceStack-Reference)
* [Swift Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/Swift-Add-ServiceStack-Reference)
* [Java Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/Java-Add-ServiceStack-Reference)
* [Kotlin Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/Kotlin-Add-ServiceStack-Reference)
* [TypeScript Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/TypeScript-Add-ServiceStack-Reference)
* [F# Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/FSharp-Add-ServiceStack-Reference)
* [VB.NET Add ServiceStack Reference](https://github.com/ServiceStack/ServiceStack/wiki/VB.Net-Add-ServiceStack-Reference)

## Example Usage

> C# Android PCL Client example

![C# Android PCL Client example](https://raw.githubusercontent.com/ServiceStack/ServiceStackVS/master/Images/android-add-ref-demo.gif)

> VB.NET client talking with C# Server example

![CSharp server with VB.Net client example](https://github.com/ServiceStack/Assets/raw/master/img/servicestackvs/servicestack%20reference/csharp-server-vb-client.gif)

Options for the generated DTOs can be changed by updating the commented section in the header of the file. Each language will have different options based on what is applicable to that language. For details on these options, please see the specific language wiki page.

* [C# Options](https://github.com/ServiceStack/ServiceStack/wiki/CSharp-Add-ServiceStack-Reference#change-default-server-configuration)
* [Swift Options](https://github.com/ServiceStack/ServiceStack/wiki/Swift-Add-ServiceStack-Reference#swift-configuration)
* [Java Options](https://github.com/ServiceStack/ServiceStack/wiki/Java-Add-ServiceStack-Reference#java-configuration)
* [Kotlin Options](https://github.com/ServiceStack/ServiceStack/wiki/Kotlin-Add-ServiceStack-Reference#kotlin-configuration)
* [TypeScript Options](https://github.com/ServiceStack/ServiceStack/wiki/TypeScript-Add-ServiceStack-Reference#change-default-server-configuration)
* [F# Options](https://github.com/ServiceStack/ServiceStack/wiki/FSharp-Add-ServiceStack-Reference#change-default-server-configuration)
* [VB.Net Options](https://github.com/ServiceStack/ServiceStack/wiki/VB.Net-Add-ServiceStack-Reference)

### ssutil.exe - Command line ServiceStack Reference tool

Add ServiceStack Reference is also moving beyond our growing list of supported IDEs and is now available in a single cross-platform .NET command-line **.exe** making it easy for build servers and automated tasks or command-line runners of your favorite text editors to easily Add and Update ServiceStack References!

To Get Started download **ssutil.exe** and open a command prompt to the containing directory:

#### Download [ssutil.exe](https://github.com/ServiceStack/ServiceStackVS/raw/master/dist/ssutil.exe)

#### ssutil.exe Usage:

![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/servicestackvs/ssutil-help.png)

**Adding a new ServiceStack Reference**

To create a new ServiceStack Reference, pass the remote ServiceStack **BaseUrl** then specify both which `-file` and `-lang` you want, e.g:

    ssutil http://techstacks.io -file TechStacks -lang CSharp

Executing the above command fetches the C# DTOs and saves them in a local file named `TechStacks.dtos.cs`.

**Available Languages**

 - CSharp
 - Swift
 - Java
 - Kotlin
 - TypeScript / TypeScript.d
 - FSharp
 - VbNet

**Update existing ServiceStack Reference**

Updating a ServiceStack Reference is even easier we just specify the path to the existing generated DTOs. E.g. Update the `TechStacks.dtos.cs` we just created with:

    ssutil TechStacks.dtos.cs

### Running on Windows

[![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/servicestackvs/ssutil-demo.gif)](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/servicestackvs/ssutil-demo.gif)

### Running on OSX

[![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/servicestackvs/ssutil-demo-osx.gif)](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/servicestackvs/ssutil-demo-osx.gif)

## Advantages over WCF

 - **Simple** Server provides DTOs based on metadata and options provided. No heavy client side tools, just a HTTP request!
 - **Versatile** Clean DTOs works in all JSON, XML, JSV, MsgPack and ProtoBuf [generic service clients](https://github.com/ServiceStack/ServiceStack/wiki/C%23-client#built-in-clients)
 - **Reusable** Generated DTOs are not coupled to any endpoint or format. Defaults are both partial and virtual for maximum re-use 
 - **Resilient** Messaging-based services offer a number of [advantages over RPC Services](https://github.com/ServiceStack/ServiceStack/wiki/Advantages-of-message-based-web-services)
 - **Flexible** DTO generation is customizable, Server and Clients can override built-in defaults
 - **Integrated** Rich Service metadata annotated on DTO's, [Internal Services](https://github.com/ServiceStack/ServiceStack/wiki/Restricting-Services) are excluded when accessed externally


## In Contrast with WCF's Add Service Reference

WCF's **Add Service Reference** also allows generating a typed client from a single url, and whilst it's a great idea, the complexity upon what it's built-on and the friction it imposes were the primary reasons we actively avoided using it (pre-ServiceStack). We instead opted to reuse our server DTO types and created Generic WCF Proxies, to provide a cleaner and simpler solution when consuming our own WCF services. 

### Complexity of WCF's Add Service Reference

To achieve this feature WCF generates its client proxies using a remote services WSDL. A WSDL is basically a machine-readable XML definition language for describing SOAP Services. It's abstract enough to cover different styles of services and introduces a number of artificial concepts to facilitate it, including: Service, Port, Binding, PortType, Operation, Message and Types. As WSDL's are complex they mandate the use of heavy tooling to generate and maintain both the WSDL file, the generated client proxies and its necessary client configuration. Despite all this complexity it's coupled and limited into using the verbose SOAP protocol and XML wire format which when coupled with WCF's promotion of RPC method signatures meant even minor changes would break existing clients, resulting in a heavy and fragile solution for evolving web services.

### How Message based Services would benefit WCF

A small part of a WSDL is the XSD definitions of Types used in the Services. Had WCF only supported a message-based style it could dispense with the overhead of using a WSDL at all and just use XSD schema to generate the DTO's, eliminating the neeed for a SOAP envelope where it could just send [Plain Old XML](http://en.wikipedia.org/wiki/Plain_Old_XML) across the wire. As an added benefit it would've got JSON support for free by reusing the generated types in .NET's JSON DataContract Serializer. 

### Unnecessary Complexity of XSDs

Despite being much simpler, even XSD's by themselves are more complex than it needs to be. The XML Schema specification is itself several hundred pages long and contains many elements which make it a poor programmatic fit for any programming language. E.g. use of XML namespaces and attributes in addition to elements does not naturally map to any language type system and causes unnecessary friction and additional boilerplate to handle this mismatch during serialization. 

This is in stark contrast with the JSON spec which [fits on a single page](http://www.json.org/) yet manages to include most of the core elements required for data interchange consisting of `Arrays`, `Objects` and primitive `number`, `string`, `boolean` and `null` types. It's also a perfect fit for most languages where all valid JSON is always convertible to a valid JavaScript object. When more specialized types are required, you have access to the full power of the host programming language to perform custom conversions, providing a more flexible alternative than otherwise breaking clients requests on minor schema changes. 

## ServiceStack's Native Types Feature

As with any ServiceStack feature one of our primary goals is to [minimize unnecessary complexity](https://github.com/ServiceStack/ServiceStack/wiki/Auto-Query#why-not-complexity) by opting for approaches that yield maximum value and minimal complexity, favoring re-use and simple easy to reason about solutions over opaque heavy black-box tools.

We can already see from the WCF scenario how ServiceStack already benefits from its message-based design, where as it's able to reuse any [Generic Service Client](https://github.com/ServiceStack/ServiceStack/wiki/Clients-overview), only application-specific DTO's ever need to be generated, resulting in a much cleaner, simpler and friction-less solution.

Code-first is another approach that lends itself to simpler solutions, which saves the effort and inertia from adapting to interim schemas/specs, often with impedance mismatches and reduced/abstract functionality. In ServiceStack your code-first DTOs are the master authority where all other features are projected off. 

C# also has great language support for defining POCO Data Models, that's as terse as a DSL but benefits from great IDE support and minimal boilerplate, e.g:

```csharp
[Route("/path")]
public class Request : IReturn<Response>
{
    public int Id { get; set; }
    public string Name { get; set; }
    ...
}
```

Starting from a C# model, whilst naturally a better programmatic fit also ends up being richer and more expressive than XSD's which supports additional metadata annotations like Attributes and Interfaces. 

### Enabled by default from v4.0.30+ ServiceStack Projects

Native Types is now available by default on all **v4.0.30+** ServiceStack projects. It can be disabled by removing the `NativeTypesFeature` plugin with:

```csharp
Plugins.RemoveAll(x => x is NativeTypesFeature);
```

### Generating Types from Metadata

Behind the scenes ServiceStack captures all metadata on your Services DTOs including Sub -classes, Routes, `IReturn` marker, C# Attributes, textual Description as well as desired configuration into a serializable object model accessible from `/types/metadata`: 

#### Live examples

  - [httpbenchmarks.servicestack.net/types/metadata](https://httpbenchmarks.servicestack.net/types/metadata) ([JSON](https://httpbenchmarks.servicestack.net/types/metadata.json))
  - [stackapis.servicestack.net/types/metadata](http://stackapis.servicestack.net/types/metadata) ([JSON](http://stackapis.servicestack.net/types/metadata.json))

This model is then used to generate the generated types, which for C# is at `/types/csharp`.

### Excluding Types from Add ServiceStack Reference

To remove a type from the metadata and code generation you can annotate Request DTOs with `[Exclude(Feature.Metadata)]`, e.g:

```csharp
[Exclude(Feature.Metadata)]
public class ExcludedFromMetadata
{
    public int Id { get; set; }
}
```

An alternative is it add it to the `IgnoreTypes` collection in the NativeTypes Feature Metadata Config in your AppHost:

```csharp
var nativeTypes = this.GetPlugin<NativeTypesFeature>();
nativeTypes.MetadataTypesConfig.IgnoreTypes.Add(typeof(TypeToIgnore));
```

If you only want to limit code generation based on where the reference is being added from you can use the 
[Restrict Attribute](https://github.com/ServiceStack/ServiceStack/wiki/Restricting-Services), 
E.g you can limit types to only appear when the reference is added from localhost:

```csharp
[Restrict(LocalhostOnly = true)]
public class ResrtictedToLocalhost { }
```

Or when added from within an internal network:

```csharp
[Restrict(InternalOnly = true)]
public class RestrictedToInternalNetwork { }
```

There's also the rarer option when you only want a service accessible from external requests with:

```csharp
[Restrict(ExternalOnly = true)]
public class RestrictedToExternalRequests { }
```

### How it works

The Add ServiceStack Reference dialog just takes the URL provided and requests the appropriate route for the current project. Eg, for C#, the path used is at `/types/csharp`. The defaults are specified by the server and the resultant DTOs are saved and added the the project as {Name}.dtos.{LanguageExtension}. The `Update ServiceStack Reference` menu is available when any file matches same naming convention of {Name}.dtos.{LanguageExtension}. An update then looks at the comments at the top of the file and parses them to provide overrides when requesting new DTOs from the server. ServiceStackVS also watches these DTO files for updates, so just by saving them these files are updated from the server.

#### Language Paths

- `/types/csharp` - C# 
- `/types/swift` - Swift 
- `/types/java` - Java 
- `/types/kotlin` - Kotlin 
- `/types/typescript` - TypeScript 
- `/types/typescript.d` - Ambient TypeScript Definitions
- `/types/fsharp` - F# 
- `/types/vbnet` - VB.NET 
- `/types/metadata` - Metadata 

## Limitations

In order for Add ServiceStack Reference to work consistently across all supported languages without .NET semantic namespaces, DTOs includes an additional restriction where each Type must be uniquely named. You can get around this restriction by sharing the ServiceModel.dll where your DTOs are defined instead.

## Using with IIS Windows Authentication

If you have configured your NativeTypes service to run on IIS with Windows Authentication enabled, you need to ensure that the _/types_ routes are reachable and do not require the system-level authentication from IIS. To accomplish this, add the following to Web.config. 

```xml
<configuration>
    <location path="types">
        <system.web>
            <authorization>
                <allow users="?" />
            </authorization>
        </system.web>
    </location>
</configuration>
```
