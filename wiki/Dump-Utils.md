In ServiceStack.Text's JsvFormatter class are extension methods which recursively dumps all the public properties of any type into a human readable **pretty formatted** string.

```csharp
string Dump<T>(this T instance);
string SerializeAndFormat<T>(this T instance);

void PrintDump<T>(this T instance);
```

The `Dump` and `SerializeAndFormat` methods achieve the same result, where the logically named but lengthier `SerializeAndFormat` describes exactly what it does, although most of the time we don't care and are happy to use the shortened `Dump` to mean the same thing. 

There's also a convenient `PrintDump` extension which just writes the output to the Console to provide a wrist-friendly API for a common use-case.

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

### Inbuilt into Service Stack JSV web service endpoint

As this feature has come in super useful for debugging, it's also included it as part of the JSV endpoint by simply appending `&debug` anywhere in the request’s query string. 

Even if you don’t use the new JSV endpoint you can still benefit from it by instantly being able to read the data provided by your web service. Here are some live examples showing the same web services called from the XML and JSV endpoint that shows the difference in readability:

  - [GetNorthwindCustomerOrders](http://mono.servicestack.net/ServiceStack.Examples.Host.Web/ServiceStack/Jsv/SyncReply/GetNorthwindCustomerOrders?debug)
  - [GetFibonacciNumbers?Skip=5&Take=10](http://mono.servicestack.net/ServiceStack.Examples.Host.Web/ServiceStack/Jsv/SyncReply/GetFibonacciNumbers?Skip=5&Take=10&debug#)

