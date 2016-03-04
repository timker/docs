![F# Header](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/wikis/fsharp-header.png)

ServiceStack's **Add ServiceStack Reference** feature allows clients to generate Native Types from directly within VS.NET using [ServiceStackVS VS.NET Extension](https://github.com/ServiceStack/ServiceStack/wiki/Creating-your-first-project) - providing a simpler, cleaner and more versatile alternative to WCF's Add Service Reference feature that's built into VS.NET.

The article outlines ServiceStack's support generating F# DTO's - providing a flexible alternative than sharing your compiled DTO .NET assembly with clients. Now F# clients can easily add a reference to a remote ServiceStack instance and update typed DTO's directly from within VS.NET - reducing the burden and effort required to consume ServiceStack Services whilst benefiting from clients native language strong-typing feedback. 

## [[Add ServiceStack Reference]]

The easiest way to Add a ServiceStack reference to your project is to right-click on your project to bring up [ServiceStackVS's](https://github.com/ServiceStack/ServiceStack/wiki/Creating-your-first-project) `Add ServiceStack Reference` context-menu item. This opens a dialog where you can add the url of the ServiceStack instance you want to typed DTO's for, as well as the name of the DTO source file that's added to your project.

[![Add ServiceStack Reference](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/apps/StackApis/add-service-ref-flow.png)](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/apps/StackApis/add-service-ref-flow.png)

After clicking OK, the servers DTO's and [ServiceStack.Client](https://www.nuget.org/packages/ServiceStack.Client) NuGet package are added to the project, providing an instant typed API:

[![Calling ServiceStack Service with FSharp](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/release-notes/fsharp-add-servicestack-reference.png)](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/release-notes/fsharp-add-servicestack-reference.png)

### Update ServiceStack Reference

Updating a ServiceStack reference works intuitively where you just have to click on the DTO's you want to update and click **Update ServiceStack Reference** on the context menu:

[![Calling ServiceStack Service with FSharp](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/release-notes/fsharp-update-servicestack-reference.png)](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/release-notes/fsharp-update-servicestack-reference.png)

As there's no configuration stored about the ServiceStack Reference you might be wondering how this works? The **Update ServiceStack Reference** context menu only appears on F# source files ending with `.dto.fs` or `.dtos.fs`. It then uses the `BaseUrl` in the metadata comments for where to fetch and update to the latest F# server DTO's.

### Console Demo
![FSharp Console Demo](https://github.com/ServiceStack/Assets/raw/master/img/servicestackvs/servicestack%20reference/fsharp-demo.gif)

### F# Client Example

Just like with C#, F# Native Types can be used in ServiceStack's [Generic Service Clients](https://github.com/ServiceStack/ServiceStack/wiki/C%23-client) providing and end-to-end Typed API whose PCL support also allows F# to be used in [mobile clients apps](https://github.com/ServiceStackApps/HelloMobile) without having to share compiled DTOs:

```fsharp
let client = new JsonServiceClient("http://stackapis.servicestack.net")
let response = client.Get(new SearchQuestions(
    Tags = new List<string>([ "redis"; "ormlite" ])))        

TypeSerializer.PrintDump(response)
```

### Change Default Server Configuration

The above defaults are also overridable on the ServiceStack Server by modifying the default config on the `NativeTypesFeature` Plugin, e.g:

```csharp
var typesConfig = this.GetPlugin<NativeTypesFeature>().MetadataTypesConfig;
typesConfig.AddDataContractAttributes = false;
...
```

## Constraints

As the ordering contraint in F# conflicted with the ordering of types by C# namespaces, the cleanest approach was to add all DTO's under a single namespace. By default the namespace used will be the base **ServiceModel** namespace which is overridable with the `GlobalNamespace` Config:

```csharp
typesConfig.GlobalNamespace = "Client.Namespace";
```

This does mean that each type name needs to be unique which is a best-practice that's now a requirement in order to make use of F# native types. Another semantic difference is that any C# nested classes become top-level classes when translated to F#.  

### FSharp Configuration

The header comments in the generated DTO's allows for further customization of how they're generated where ServiceStackVS automatically watches for any file changes and updates the generated DTO's with any custom Options provided. Options that are preceded by a F# single line comment `//` are defaults from the server that can be overridden, e.g:

```fsharp
(* Options:
Date: 2014-10-21 00:45:38
Version: 1
BaseUrl: http://stackapis.servicestack.net

//MakeDataContractsExtensible: False
//AddReturnMarker: True
//AddDescriptionAsComments: True
//AddDataContractAttributes: False
//AddIndexesToDataMembers: False
//AddGeneratedCodeAttributes: False
//AddResponseStatus: False
//AddImplicitVersion: 
//InitializeCollections: True
*)
```

To override a value, remove the `//` and specify the value to the right of the `:`. Any value uncommented will be sent to the server to override any server defaults.

We'll go through and cover each of the above options to see how they affect the generated DTO's:

### MakeDataContractsExtensible

Add .NET's DataContract's [ExtensionDataObject](http://msdn.microsoft.com/en-us/library/system.runtime.serialization.extensiondataobject(v=vs.110).aspx) to all DTO's:

```fsharp
[<AllowNullLiteral>]
type GetAnswersResponse() = 
    interface IExtensibleDataObject with
        member val ExtensionData:ExtensionDataObject = null with get, set
    end
    member val Ansnwer:Answer = null with get,set
    member val ExtensionData:ExtensionDataObject = null with get,set
```

### AddReturnMarker

AddReturnMarker annotates Request DTO's with an `IReturn<TResponse>` marker referencing the Response type ServiceStack infers your Service to return:

```fsharp
type GetAnswers() = 
    interface IReturn<GetAnswersResponse>
    member val QuestionId:Int32 = new Int32() with get,set
``` 

> Original DTO doesn't require a return marker as response type can be inferred from Services return type or when using the `%Response` DTO Naming convention

### AddDescriptionAsComments

Converts any textual Description in `[Description]` attributes as F# Doc comments which allows your API to add intellisense in client projects:

```fsharp
///<summary>
///Get a list of Answers for a Question
///</summary>
type GetAnswers() = 
...
```

### AddDataContractAttributes

Decorates all DTO types with `[DataContract]` and properties with `[DataMember]`:

```fsharp
[<DataContract>]
[<AllowNullLiteral>]
type GetAnswers() = 
    interface IReturn<GetAnswersResponse>
    [<DataMember>]
    member val QuestionId:Int32 = new Int32() with get,set
```

### AddIndexesToDataMembers

Populates a DataMember Order index for all properties:

```fsharp
[<DataContract>]
type GetAnswers() = 
    interface IReturn<GetAnswersResponse>
    [<DataMember(Order=1)>]
    member val QuestionId:Int32 = new Int32() with get,set
```

> Requires AddDataContractAttributes=true

### AddGeneratedCodeAttributes

Emit `[<GeneratedCode>]` attribute on all generated Types:

```fsharp
[<GeneratedCode>]
type GetAnswers() = ...
```

### AddResponseStatus

Automatically add a `ResponseStatus` property on all Response DTO's, regardless if it wasn't already defined:

```fsharp
type GetAnswersResponse() = 
    ...
    member val ResponseStatus:ResponseStatus = null with get,set
```

### AddImplicitVersion

Lets you specify the Version number to be automatically populated in all Request DTO's sent from the client: 

```csharp
type GetAnswers() = 
    interface IReturn<GetAnswersResponse>
    member val Version:int = 1 with get, set
    ...
```

This lets you know what Version of the Service Contract that existing clients are using making it easy to implement ServiceStack's [recommended versioning strategy](http://stackoverflow.com/a/12413091/85785). 

### InitializeCollections

Lets you automatically initialize collections in Request DTO's:

```fsharp
type SearchQuestions() = 
    interface IReturn<SearchQuestionsResponse>
    member val Tags:List<String> = new List<String>() with get,set
```