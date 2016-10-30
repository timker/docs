---
slug: dump-utils
title: Dump Utils
---

ServiceStack.Text has extension methods which recursively dumps all the public properties of any type into a human readable **pretty formatted** string. The `Dump()` utils are invaluable when explanatory coding or creating tests as you can quickly see what's in an object without having to set breakpoints and navigate nested properties in VS.NET's Watch window.

#### Extension Methods

```csharp
string Dump<T>(this T instance);
string SerializeAndFormat<T>(this T instance); //Wordier Alias

void PrintDump<T>(this T instance);
void Print(this string text, params object[] args);
```

The convenient `PrintDump` and `Print()` extension just writes the output to the Console to provide a wrist-friendly API for a common use-case, e.g:

```
var response = client.Send(request);
response.PrintDump(); // Dumps contents to Console in human-friendly format

"Top Technologies: {0}".Print(response.TopTechnologies.Dump());
```

## Example Usage

After importing the **ServiceStack.Text** namespace you can view the values of all fields as seen in [the following example](https://github.com/ServiceStack/ServiceStack.Text/blob/master/tests/ServiceStack.Text.Tests/Utils/JsvFormatterTests.cs):

```csharp
var model = new TestModel();
model.PrintDump();
```

### Example Output

```csharp
{
    Int: 1,
    String: One,
    DateTime: 2010-04-11,
    Guid: c050437f6fcd46be9b2d0806a0860b3e,
    EmptyIntList: [],
    IntList:
    [
        1,
        2,
        3
    ],
    StringList:
    [
        one,
        two,
        three
    ],
    StringIntMap:
    {
        a: 1,
        b: 2,
        c: 3
    }
}
```

### Circular References

The `T.PrintDump()` and `T.Dump()` extension methods can also be used on objects with cyclical references 
where it will display the first-level `ToString()` value of properties that have circular references.

Whilst our Text Serializers don't support serializing DTOs with cyclical dependencies (which are actively discouraged), 
the APIs below can be used instead to partially serialize objects where it uses the `ToString()` on any properties containing Circular references:

 - `T.ToSafeJson()`
 - `T.ToSafeJsv()`
 - `T.ToSafePartialObjectDictionary()`

The API used to detect whether an object has Circular References is also available for your use: 

```csharp
if (obj.HasCircularReferences()) {
}
```

### Inbuilt into Service Stack JSV web service endpoint

As this feature has come in super useful for debugging, it's also included it as part of the JSV endpoint by simply appending `&debug` anywhere in the request’s query string. 

Even if you don’t use the new JSV endpoint you can still benefit from it by instantly being able to read the data provided by your web service. Here are some live examples showing the same web services called from the XML and JSV endpoint that shows the difference in readability:

  - [GetNorthwindCustomerOrders](http://northwind.servicestack.net/json/reply/Orders?debug)

