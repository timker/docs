![VB.NET Header](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/wikis/vb-header.png)

ServiceStack's **Add ServiceStack Reference** feature allows clients to generate Native Types from directly within VS.NET using [ServiceStackVS VS.NET Extension](https://github.com/ServiceStack/ServiceStack/wiki/Creating-your-first-project) - providing a simpler, cleaner and more versatile alternative to WCF's Add Service Reference feature that's built into VS.NET.

The article outlines ServiceStack's support generating VB.Net DTO's - providing a flexible alternative than sharing your compiled DTO .NET assembly with clients. Now VB.Net clients can easily add a reference to a remote ServiceStack instance and update typed DTO's directly from within VS.NET - reducing the burden and effort required to consume ServiceStack Services whilst benefiting from clients native language strong-typing feedback. 

## [[Add ServiceStack Reference]]

The easiest way to Add a ServiceStack reference to your project is to right-click on your project to bring up [ServiceStackVS's](https://github.com/ServiceStack/ServiceStack/wiki/Creating-your-first-project) `Add ServiceStack Reference` context-menu item. This opens a dialog where you can add the url of the ServiceStack instance you want to typed DTO's for, as well as the name of the DTO source file that's added to your project.

[![Add ServiceStack Reference](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/apps/StackApis/add-service-ref-flow.png)](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/apps/StackApis/add-service-ref-flow.png)

After clicking OK, the servers DTO's and [ServiceStack.Client](https://www.nuget.org/packages/ServiceStack.Client) NuGet package are added to the project, providing an instant typed API:
![VB.Net Console client](https://github.com/ServiceStack/Assets/raw/master/img/apps/StackApis/call-service-vb.png)

With the VB.Net code generated on the Server, the role of [ServiceStackVS's](https://github.com/ServiceStack/ServiceStack/wiki/Creating-your-first-project) **Add ServiceStack Reference** is there just to integrate the remote VB.Net DTO's into the clients VS.NET project. This is just getting the generated DTOs from the server with default options set by the server and adding them locally to your project within Visual Studio.

![Add VB.Net ServiceStack Reference Demo](https://github.com/ServiceStack/Assets/raw/master/img/servicestackvs/servicestack%20reference/addref-vbnet.gif)

## Update ServiceStack Reference
If your server has been updated and you want to update to client DTOs, simply right-click on the DTO file within VS.NET and select `Update ServiceStack Reference`. 

![VBNet update demo](https://github.com/ServiceStack/Assets/raw/master/img/servicestackvs/servicestack%20reference/updateref-vbnet.gif)

## DTO Customization Options 

The header comments in the generated DTO's allows for further customization of how they're generated where ServiceStackVS automatically watches for any file changes and updates the generated DTO's with any custom Options provided. Options that are preceded by a VB.Net Class comment `'''` are defaults from the server that can be overridden, e.g:

```VB.Net
' Options:
'Date: 2014-10-21 00:45:05
'Version: 1
'BaseUrl: http://stackapis.servicestack.net
'
'''MakePartial: True
'''MakeVirtual: True
'''MakeDataContractsExtensible: False
'''AddReturnMarker: True
'''AddDescriptionAsComments: True
'''AddDataContractAttributes: False
'''AddIndexesToDataMembers: False
'''AddGeneratedCodeAttributes: False
'''AddResponseStatus: False
'''AddImplicitVersion: 
'''InitializeCollections: True
'''AddDefaultXmlNamespace: http://schemas.servicestack.net/types
```
To override these options on the client, the comment has to be changed to start with a single `'` instead of triple `'''`. This convention is due to VB.Net not having block quotes. For example, if we did't want our classes to be partial by default for the VB.Net client, our options would look like below.

```VB.Net
' Options:
'Date: 2014-10-21 00:45:05
'Version: 1
'BaseUrl: http://stackapis.servicestack.net
'
'MakePartial: False
'''MakeVirtual: True
'''MakeDataContractsExtensible: False
'''AddReturnMarker: True
'''AddDescriptionAsComments: True
'''AddDataContractAttributes: False
'''AddIndexesToDataMembers: False
'''AddGeneratedCodeAttributes: False
'''AddResponseStatus: False
'''AddImplicitVersion: 
'''InitializeCollections: True
'''AddDefaultXmlNamespace: http://schemas.servicestack.net/types
```
Options that do not start with a `'''` are sent to the server to override any defaults set by the server.

### Change Default Server Configuration

The above defaults are also overridable on the ServiceStack Server by modifying the default config on the `NativeTypesFeature` Plugin, e.g:

```csharp
//Server example in CSharp
var nativeTypes = this.GetPlugin<NativeTypesFeature>();
nativeTypes.MetadataTypesConfig.MakeVirtual = false;
...
```

We'll go through and cover each of the above options to see how they affect the generated DTO's:

### MakePartial

Adds the `partial` modifier to all types, letting you extend generated DTO's with your own class separate from the generated types:

```VB.Net
Public Partial Class GetAnswers
```

### MakeVirtual

Adds the `virtual` modifier to all properties:

```VB.Net
Public Partial Class GetAnswers
    ...
    Public Overridable Property QuestionId As Integer
End Class
```

### MakeDataContractsExtensible

Add .NET's DataContract's [ExtensionDataObject](http://msdn.microsoft.com/en-us/library/system.runtime.serialization.extensiondataobject(v=vs.110).aspx) to all DTO's:

```VB.Net
Public Partial Class Hello
            ...
    Implements IExtensibleDataObject
            ...
    Public Overridable Property ExtensionData As ExtensionDataObject Implements IExtensibleDataObject.ExtensionData
End Class
```

### AddReturnMarker

AddReturnMarker annotates Request DTO's with an `IReturn(Of T)` marker referencing the Response type ServiceStack infers your Service to return:

```VB.Net
Public Partial Class GetAnswers
    Implements IReturn(Of GetAnswersResponse)
``` 

> Original DTO doesn't require a return marker as response type can be inferred from Services return type or when using the `%Response` DTO Naming convention

### AddDescriptionAsComments

Converts any textual Description in `<Description>` attributes as VB.Net class Doc comments which allows your API to add intellisense in client projects:

```VB.Net
'''<Summary>
'''Get a list of Answers for a Question
'''</Summary>
Public Class GetAnswers
```

### AddDataContractAttributes

Decorates all DTO types with `<DataContract>` and properties with `<DataMember>` as well as adding default XML namespaces for all VB.Net namespaces used:

```VB.Net
<Assembly: ContractNamespace("http://schemas.servicestack.net/types", ClrNamespace:="StackApis.ServiceModel.Types")>
<Assembly: ContractNamespace("http://schemas.servicestack.net/types", ClrNamespace:="StackApis.ServiceModel")>
...
<DataContract>
Partial Public Class GetAnswers
    Implements IReturn(Of GetAnswersResponse)
    <DataMember>
    Public Overridable Property QuestionId As Integer
End Class
```

### AddIndexesToDataMembers

Populates a DataMember Order index for all properties:

```VB.Net
<DataContract>
Public Partial Class GetAnswers
    ...
    <DataMember(Order:=1)>
    Public Overridable Property QuestionId As Integer
End Class
```

> Requires **AddDataContractAttributes=true**

### AddGeneratedCodeAttributes

Emit `<GeneratedCode>` attribute on all generated Types:

```VB.Net
<GeneratedCode>
Public Partial Class GetAnswers ...
```

### AddResponseStatus

Automatically add a `ResponseStatus` property on all Response DTO's, regardless if it wasn't already defined:

```VB.Net
Public Partial Class GetAnswers
    ...
    Public Overridable Property ResponseStatus As ResponseStatus
End Class
```

### AddImplicitVersion

Lets you specify the Version number to be automatically populated in all Request DTO's sent from the client: 

```VB.Net
Public Partial Class GetAnswers
    Public Overridable Property Version As Integer
    Public Sub New()
                Version = 1
    End Sub
    ...
End Class
```

This lets you know what Version of the Service Contract that existing clients are using making it easy to implement ServiceStack's [recommended versioning strategy](http://stackoverflow.com/a/12413091/85785). 

### InitializeCollections

Lets you automatically initialize collections in Request DTO's:

```VB.Net
Public Partial Class SearchQuestions
    Public Sub New()
        Tags = New List(Of String)
    End Sub
    Public Overridable Property Tags As List(Of String)
    ...
}
```
### AddDefaultXmlNamespace

This lets you change the default DataContract XML namespace used for all namespaces:

```csharp
<Assembly: ContractNamespace("http://my.types.net", ClrNamespace:="StackApis.ServiceModel.Types")>
<Assembly: ContractNamespace("http://my.types.net", ClrNamespace:="StackApis.ServiceModel")>
```

> Requires AddDataContractAttributes=true