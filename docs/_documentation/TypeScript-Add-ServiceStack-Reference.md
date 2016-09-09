---
topic: clients
title: TypeScript Add Reference
priority: 4
---
![ServiceStack and TypeScript Banner](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/wikis/typescript-banner.png)

ServiceStack's **Add ServiceStack Reference** feature allows clients to generate Native Types from directly within VS.NET using [ServiceStackVS VS.NET Extension](https://github.com/ServiceStack/ServiceStack/wiki/Creating-your-first-project) - providing a simple way to give clients typed access to your ServiceStack Services.

[TypeScript](http://www.typescriptlang.org/) is a superset of JavaScript that enhances it with an optional type system for annotating existing JavaScript data structures - bringing many of the code-analysis, insights and tooling benefits that we get to enjoy developing in a typed language like C#/VS.NET. 

This article outlines ServiceStack's support for generating TypeScript DTO definitions - allowing JavaScript clients to benefit from TypeScript's strong typing feedback valuable during development when consuming evolvable Web API's by adding a reference to a remote ServiceStack instance directly from within VS.NET.

### TypeScript Ambient Interface Definitions or Concrete Types

Use the `ExportAsTypes=true` option to generate non-ambient concrete TypeScript Types. This option is enabled at the 
new `/types/typescript` route so you can get both concrete types and interface definitions at:

  - [/types/typescript](http://techstacks.io/types/typescript) - for generating concrete module and classes
  - [/types/typescript.d](http://techstacks.io/types/typescript.d) - for generating ambient interface definitions

## [[Add ServiceStack Reference]]

The easiest way to Add a ServiceStack reference to your project is to **right-click** on your project to bring up [ServiceStackVS's](https://github.com/ServiceStack/ServiceStack/wiki/Creating-your-first-project) VS.NET context-menu item, then click on `Add -> TypeScript Reference...`. This opens a dialog where you can add the url of the ServiceStack instance you want to typed DTO's for, as well as the name of the DTO source file that's added to your project.

[![Add ServiceStack Reference](https://github.com/ServiceStack/Assets/blob/master/img/servicestackvs/add-typescript-reference-js.png)](https://github.com/ServiceStack/Assets/blob/master/img/servicestackvs/add-typescript-reference-js.png)

After clicking OK, the servers DTO's are added to the project, yielding an instant typed API:

![TypeScript native types](https://github.com/ServiceStack/Assets/blob/master/img/servicestackvs/add-typescript-reference-dtos.png)

### Update ServiceStack Reference

If your server has been updated and you want to update the client DTOs, simply **right-click** on the DTO file within VS.NET and select **Update ServiceStack Reference**, alternatively modifying or **Saving** the generated `*.dtos.d.ts` file also instructs **ServiceStackVS** to download a fresh update. 

## TypeScript Interface Definitions

The Native TypeScript DTO's are made available in the form of a [TypeScript declaration file](http://www.typescriptlang.org/Handbook#writing-dts-files) (.d.ts). TypeScript declarations are just pure static type annotations, i.e. they don't generate any code or otherwise have any effect on runtime behavior. This makes them useful as a non-invasive drop-in into existing JavaScript code where it's just used to provide type annotations on existing JavaScript objects, letting you continue using your existing data types and ajax libraries.

### TypeScript Reference Example

Lets walk through a simple example to see how we can use ServiceStack's TypeScript DTO annotations in our JavaScript clients. Firstly we'll need to add a TypeScript Reference to the remote ServiceStack Service by **right-clicking** on your project and clicking on `Add > TypeScript Reference...` (as seen in the above screenshot).

This will import the remote Services dtos into your local project which looks similar to:

```
/* Options:
Date: 2014-12-08 17:24:02
Version: 1
BaseUrl: http://api.example.com

GlobalNamespace: dtos
//MakePropertiesOptional: True
//AddServiceStackTypes: True
//AddResponseStatus: False
*/

declare module dtos
{
    // @Route("/hello")
    // @Route("/hello/{Name}")
    interface Hello extends IReturn<HelloResponse>
    {
        // @Required()
        name: string;
        title?: string;
    }

    interface HelloResponse
    {
        result?:string;
    }

    interface IReturn<T> {}
    ...
}
```

Initially the TypeScript module containing all the DTO definitions will default to the C#/.NET Service Model namespace, but can be made more readable by adding`GlobalNamespace: dtos` in the header properties.

Looking at the types we'll notice the DTO's are just interface type definitions with any .NET attributes added in comments using AtScript's proposed [meta-data annotations format](https://docs.google.com/document/d/11YUzC-1d0V1-Q3V0fQ7KSit97HnZoKVygDxpWzEYW0U/mobilebasic?viewopt=127). This lets you view helpful documentation about your DTO's like the different custom routes available for each Request DTO.

By default DTO properties are optional but can be made a required field by annotating the .NET property with the `[Required]` attribute or by uncommenting `MakePropertiesOptional: False` in the header comments which instead defaults to using required properties.

Properties always reflect to match the remote servers JSON Serialization configuration, i.e. will use **camelCase** properties when the `AppHost` is configured with:

```csharp
JsConfig.EmitCamelCaseNames = true;
```

### Referencing TypeScript DTO's

Once added to your project, use VS.NET's JavaScript doc comments to reference the TypeScript definitions in your `.ts` scripts. The example below shows how to use the above TypeScript definitions to create a typed Request/Response utilizing jQuery's Ajax API to fire off a new Ajax request on every keystroke:

```html
/// <reference path="MyApis.dtos.d.ts"/>
...

<input type="text" id="txtHello" data-keyup="sayHello" /> 
<div id="result"></div>

<script>
$(document).bindHandlers({
    sayHello: function () {
        var request: dtos.Hello = {};
        request.title = "Dr";
        request.name = this.value;
        
        $.getJSON(createUrl("/hello", request), request, 
            function (r: dtos.HelloResponse) {
                $("#result").html(r.result);
            });
    }
});

function createUrl(path: string, params: any): string {
    for (var key in params) {
        path += path.indexOf('?') < 0 ? "?" : "&";
        path += key + "=" + encodeURIComponent(params[key]);
    }
    return path;
}
</script>
```

Here we're just using a simple inline `createUrl()` function to show how we're creating the url for the **GET** HTTP Request by appending all Request DTO properties in the QueryString, resulting in a HTTP GET Request that looks like:

    /hello?title=Dr&name=World

There's also a new `$.ss.createUrl()` API in [ss-utils.js](https://github.com/ServiceStack/ServiceStack/wiki/ss-utils.js-JavaScript-Client-Library) which also handles .NET Route definitions where it will populate any variables in the `/path/{info}` instead of adding them to the `?QueryString`, e.g:

```js
$(document).bindHandlers({
    sayHello: function () {
        var request: dtos.Hello = {};
        request.title = "Dr";
        request.name = this.value;
        
        $.getJSON($.ss.createUrl("/hello/{Name}", request), request, 
            function (r: dtos.HelloResponse) {
                $("#result").html(r.result);
            });
    }
});
```

Which results in a HTTP GET request with the expected Url:

    /hello/World?title=Dr

## DTO Customization Options 

The header in the generated DTO's show the different options TypeScript native types support with their defaults. Default values are shown with the comment prefix of `//`. To override a value, remove the `//` and specify the value to the right of the `:`. Any uncommented value will be sent to the server to override any server defaults.

The DTO comments allows for customizations for how DTOs are generated. The default options that were used to generate the DTO's are repeated in the header comments of the generated DTOs, options that are preceded by a TypeScript comment `//` are defaults from the server, any uncommented value will be sent to the server to override any server defaults.

```ts
/* Options:
Date: 2015-11-21 00:32:00
Version: 4.048
BaseUrl: 

GlobalNamespace: dtos
//ExportAsTypes: False
//MakePropertiesOptional: True
//AddServiceStackTypes: True
//AddResponseStatus: False
//AddImplicitVersion: 
//IncludeTypes: 
//ExcludeTypes: 
//DefaultImports: 
*/
```

We'll go through and cover each of the above options to see how they affect the generated DTO's:

### Change Default Server Configuration

The above defaults are also overridable on the ServiceStack Server by modifying the default config on the `NativeTypesFeature` Plugin, e.g:

```csharp
//Server example in CSharp
var nativeTypes = this.GetPlugin<NativeTypesFeature>();
nativeTypes.MetadataTypesConfig.GlobalNamespace = "dtos";
...
```

We'll go through and cover each of the above options to see how they affect the generated DTO's:

### GlobalNamespace

Changes the name of the module that contain the generated TypeScript definitions:

```ts
declare module dtos
{
    ...
}
```

### ExportAsTypes

Changes whether types should be generated as ambient interface definitions or exported as concrete Types:

```ts
module dtos
{
    export interface IReturnVoid
    {
    }
    ...
}
```

### MakePropertiesOptional

Changes whether the default of whether each property is optional or not:

```ts
interface Answer
{
    AnswerId: number;
    Owner: User;
    IsAccepted: boolean;
    Score: number;
    LastActivityDate: number;
    LastEditDate: number;
    CreationDate: number;
    QuestionId: number;
}
```

### AddResponseStatus

Automatically add a `ResponseStatus` property on all Response DTO's, regardless if it wasn't already defined:

```ts
interface GetAnswers extends IReturn<GetAnswersResponse>
{
    ...
    ResponseStatus: ResponseStatus;
}
```

### AddImplicitVersion

Lets you specify the Version number to be automatically populated in all Request DTO's sent from the client: 

```ts
interface GetAnswers extends IReturn<GetAnswersResponse>
{
    Version: number; //1
    ...
}
```

This lets you know what Version of the Service Contract that existing clients are using making it easy to implement ServiceStack's [recommended versioning strategy](http://stackoverflow.com/a/12413091/85785). 