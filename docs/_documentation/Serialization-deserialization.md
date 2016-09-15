---
slug: serialization-deserialization
---
# Serialization and deserialization

### Passing complex objects in the Query String

ServiceStack uses the [JSV-Format](https://github.com/ServiceStack/ServiceStack.Text/wiki/JSV-Format) (JSON without quotes) to parse QueryStrings.

JSV lets you embed deep object graphs in QueryString as seen [this example url](http://test.servicestack.net/json/reply/StoreLogs?Loggers=%5B%7BId:786,Devices:%5B%7BId:5955,Type:Panel,TimeStamp:1199303309,Channels:%5B%7BName:Temperature,Value:58%7D,%7BName:Status,Value:On%7D%5D%7D,%7BId:5956,Type:Tank,TimeStamp:1199303309,Channels:%5B%7BName:Volume,Value:10035%7D,%7BName:Status,Value:Full%7D%5D%7D%5D%7D%5D):

    http://test.servicestack.net/json/reply/StoreLogs?Loggers=[{Id:786,Devices:[{Id:5955,Type:Panel,
      Channels:[{Name:Temperature,Value:58},{Name:Status,Value:On}]},
      {Id:5956,Type:Tank,TimeStamp:1199303309,
      Channels:[{Name:Volume,Value:10035},{Name:Status,Value:Full}]}]}]

If you want to change the default binding ServiceStack uses, you can register your own **Custom Request Binder**.

## Custom Media Types

ServiceStack serializes and deserializes your DTOs automatically. If you want to override the default serializers or you want to add a new format, you have to register your own `Content-Type`:

### Register a custom format

```csharp
string contentType = "application/yourformat"; //To override JSON eg, write "application/json"
var serialize = (IRequest req, object response, Stream stream) => ...;
var deserialize = (Type type, Stream stream) => ...;

//In AppHost Configure method
//Pass two delegates for serialization and deserialization
this.ContentTypes.Register(contentType, serialize, deserialize);	
```
The [[Protobuf-format]] shows an example of registering a new format whilst the [Northwind VCard Format](http://northwind.servicestack.net/vcard-format.htm) shows an example of creating a custom media type in ServiceStack.

***

## Reading in and De-Serializing ad-hoc custom requests

There are 2 ways to deserialize your own custom format, via attaching a custom request binder for a particular service or marking your service with `IRequiresRequestStream` which will skip auto-deserialization and inject the ASP.NET Request stream instead.

### Create a custom request dto binder

You can register custom binders in your AppHost by using the example below:

    base.RequestBinders.Add(typeof(MyRequest), httpReq => ... requestDto);

This gives you access to the IHttpRequest object letting you parse it manually so you can construct and return the strong-typed request DTO manually which will be passed to the service instead.

### Uploading Files

You can access uploaded files independently of the Request DTO using `Request.Files`. e.g:

```csharp
public object Post(MyFileUpload request)
{
    if (this.Request.Files.Length > 0)
    {
        var uploadedFile = base.Request.Files[0];
        uploadedFile.SaveTo(MyUploadsDirPath.CombineWith(file.FileName));
    }
    return HttpResult.Redirect("/");
}
```

ServiceStack's [imgur.servicestack.net](http://imgur.servicestack.net) example shows how to access the [byte stream of multiple uploaded files](https://github.com/ServiceStackApps/Imgur/blob/master/src/Imgur/Global.asax.cs#L62), e.g:

```csharp
public object Post(Upload request)
{
    foreach (var uploadedFile in base.Request.Files
       .Where(uploadedFile => uploadedFile.ContentLength > 0))
    {
        using (var ms = new MemoryStream())
        {
            uploadedFile.WriteTo(ms);
            WriteImage(ms);
        }
    }
    return HttpResult.Redirect("/");
}
```

### Reading directly from the Request Stream

Instead of registering a custom binder you can skip the serialization of the request DTO, you can add the [IRequiresRequestStream](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/ServiceHost/IRequiresRequestStream.cs) interface to directly retrieve the stream without populating the request DTO.

```csharp
//Request DTO
public class Hello : IRequiresRequestStream
{
    /// <summary>
    /// The raw Http Request Input Stream
    /// </summary>
    Stream RequestStream { get; set; }
}
```

You can access raw WCF Message when accessed with the SOAP endpoints in your Service with `IHttpRequest.GetSoapMessage()` extension method, e.g:

    Message requestMsg = base.Request.GetSoapMessage();

To tell ServiceStack to skip Deserializing the SOAP request entirely, add the `IRequiresSoapMessage` interface to your Request DTO, e.g:

    public class RawWcfMessage : IRequiresSoapMessage {
    	public Message Message { get; set; }
    }

    public object Post(RawWcfMessage request) { 
    	request.Message... //Raw WCF SOAP Message
    }

### Buffering the Request and Response Streams

ServiceStack's Request and Response stream are non-buffered (i.e. forward-only) by default. This can be changed at runtime using a PreRequestFilter to allow the Request Body and Response Output stream to be re-read multiple times should your Services need it:

```csharp
appHost.PreRequestFilters.Add((httpReq, httpRes) => {
    httpReq.UseBufferedStream = true;
    httpRes.UseBufferedStream = true;    
});
```