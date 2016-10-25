---
slug: restricting-services
title: Restricting Services
---

### Restrict Services

You can change the Visibility and Access restrictions on any service using the new [Restrict] attribute. This is a class based attribute and should be placed on your Service class.
Visibility affects whether or not the service shows up on the public `/metadata` pages, whilst access restrictions limits the accessibility of your services. 

### Named Configurations

The Restrict attribute includes a number of Named configurations for common use-cases. E.g You can specify a Service should only be available from your local machine with:

```csharp
[Restrict(LocalhostOnly = true)]
public class LocalAdmin { }
```

Which ensures access to this service is only allowed from localhost clients and the details of this service will only be visible on `/metadata` pages that are viewed locally.

This is equivalent to using the underlying granular form of specifying individual `RequestAttributes`, e.g:

```csharp
[Restrict(AccessTo = RequestAttributes.Localhost, VisibilityTo = RequestAttributes.Localhost)]
public class LocalAdmin { }
```

There are many more named configurations available. You can use **VisibleInternalOnly** to only have a service listed on internally viewed `/metadata` pages with:

```csharp
[Restrict(VisibleInternalOnly = true)]
public class InternalAdmin { }
```

Services can be restricted on any EndpointAttribute, e.g. to ensure this service is only called by XML clients, do:

```csharp
[Restrict(RequestAttributes.Xml)]
public class XmlOnly { }
```

### Restriction Combinations 

Likewise you can add any combination of Endpoint Attributes together, E.g. this restricts access to service to Internal JSON clients only:

```csharp
[Restrict(RequestAttributes.InternalNetworkAccess | RequestAttributes.Json)]
public class JsonInternalOnly { }
```

### Multiple restriction scenarios

It also supports multiple restriction scenarios, E.g. This service is only accessible by **internal JSON** clients or **External XML** clients:

```csharp
[Restrict(
    RequestAttributes.InternalNetworkAccess | RequestAttributes.Json,
    RequestAttributes.External | RequestAttributes.Xml)]
public class JsonInternalOrXmlExternalOnly { }
```

A popular configuration that takes advantage of this feature would be to only allow HTTP plain-text traffic from Internal Networks and only allow external access via secure HTTPS, which you can enforce with:

```csharp
[Restrict(RequestAttributes.InSecure | RequestAttributes.InternalNetworkAccess,
          RequestAttributes.Secure   | RequestAttributes.External)]
public class InternalHttpAndExternalHttps { }
```
