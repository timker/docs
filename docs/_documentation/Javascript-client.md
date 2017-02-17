---
slug: javascript-client
title: JavaScript Client
---

Of course, you're able to call your ServiceStack webservice from your 
ajax and JavaScript clients, too. 

### [Using TypeScript](/typescript-add-servicestack-reference#typescript-serviceclient)

The best tooling available for Ajax clients is to use ServiceStack's 
[integrated TypeScript support](/typescript-add-servicestack-reference) where
you can use the TypeScript `JsonServiceClient` with 
TypeScript Add ServiceStack Reference DTO's to get the same productive end-to-end
Typed APIs available in ServiceStack's Typed .NET Clients, e.g:

```ts
var client = new JsonServiceClient(baseUrl);

var request = new Hello();
request.Name = "World";

client.get(request)
  .then(r => console.log(r.Result));
```

### Using JavaScript JsonServiceClient

We also provide our own JsonServiceClients which mimics the [.NET Clients](/clients-overview) in functionality that we make use of in our [Redis Admin UI](http://www.servicestack.net/RedisAdminUI/AjaxClient/):

  - [JsonServiceClient.js](https://github.com/ServiceStack/ServiceStack/tree/master/lib/js/JsonServiceClient.js) - Pure JavaScript client
  - [JsonServiceClient.closure.js](https://github.com/ServiceStack/ServiceStack/tree/master/lib/js/JsonServiceClient.closure.js) - a [Google Closure](https://developers.google.com/closure/) enabled version of the client allowing compilation and bundling within a Closure project

Although most dynamic languages like JavaScript already include support for HTTP and JSON where in most cases it's easier to just use the libraries already provided. Here are a couple of examples from [Backbones Todos](http://todos.servicestack.net) and [Redis StackOverflow](http://redisstackoverflow.servicestack.net) that uses jQuery to talk to back-end ServiceStack JSON services:

### Using jQuery:

```javascript
$.getJSON("http://localhost/Backbone.Todo/todos", function(todos) {
    alert(todos.length == 1);
});

$.post("questions", { 
  UserId: authUser.Id, Title: title, Content: body, Tags: dtoTags 
}, refresh);
```

## JSV Service Client

In our pursuit to provide the fastest end-to-end communication we've also developed a JsvServiceClient in JavaScript that uses the [fast JSV Format](https://github.com/ServiceStackV3/mythz_blog/blob/master/pages/176.md):  

  - [JsvServiceClient.js](https://github.com/ServiceStack/ServiceStack/tree/master/lib/js/JSV.js)

JSV is marginally faster than **safe JSON** in modern browsers (marginally slower than Eval) but because of the poor JS and String Performance in IE7/8 it performs over **20x** slower than IE's native `eval()`.