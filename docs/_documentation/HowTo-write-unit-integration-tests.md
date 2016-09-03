---
#File header for Jekyll to pick up 
---
## Integration test example

```csharp
[Test]
public void Test_all_REST_methods()
{
    IRestClient client = new JsonServiceClient("http://localhost:5000");
    List<Customer> allCustomers = client.Get(new AllCustomers());
    Assert.That(allCustomers.Count, Is.EqualTo(0));

    var newCustomer = client.Post(
        new Customer { Name = "test", Age = 1, Email = "as@if.com" });

    Assert.That(newCustomer.Id, Is.EqualTo(1));
    Assert.That(newCustomer.Name, Is.EqualTo("test"));

    allCustomers = client.Get(new AllCustomers());
    Assert.That(allCustomers.Count, Is.EqualTo(1));

    var singleCustomer = client.Get(new Customer { Id = 1 });
    Assert.That(singleCustomer.Name, Is.EqualTo("test"));

    singleCustomer.Name = "Update Name";    

    client.Put(singleCustomer);
    singleCustomer = client.Get(new Customer { Id = 1 });

    client.Delete(new Customer { Id = 1 });

    allCustomers = client.Get(new AllCustomers());
    Assert.That(allCustomers.Count, Is.EqualTo(0));
}
```