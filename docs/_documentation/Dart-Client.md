---
slug: dart-client
title: Dart Client
---

The [Dart project](http://www.dartlang.org/) is an exciting new initiative from Google that helps you build and maintain large structured modern web apps. It comes with all batteries included, including a comprehensive library, rich Eclipse and JetBrains IDE's, built-in debugging in Dartium and Chrome (with source maps) and is being developed by many of the top talent behind Googles world-leading [V8 JavaScript engine](http://code.google.com/p/v8/) and the comprehensive [GWT toolkit](https://developers.google.com/web-toolkit/).

As we expect Dart to prove to be a popular web platform target in future, we've jumped in early and have developed a flexible [Dart JsonClient](https://github.com/Dartist/JsonClient) that allows you to effortlessly consume ServiceStack JSON services in idiomatic Dart. The client takes advantage of some of Dart's features like `noSuchMethod` and `Future<T>` to provide a natural and easy to use API, E.g:

```dart
var client = new JsonClient("http://www.servicestack.net/Backbone.Todos");

client.todos()    //GET /todos

client.todos(1)   //GET /todos/1

//POST /todos ...
client.todos({'content':'Add a new TODO!', 'order':1})

//PUT /todos/1 ...
client.put('todos/1', {"content":"Learn Dart","done":true})  

//DELETE /todos/1
client.delete('todos/1')

//GET /files/services/FilesService.cs.txt
client.files("services/FilesService.cs.txt") 
  .then( (fileInfo) => ... )

//Handling responses with Futures
client.todos(1)
  .then( (todo) => ... )       
```

### Using callbacks

The JavaScript idiom of using callbacks for handling async callbacks is still supported:

```dart
client.todos(1, (todo) => ...)  //Handling responses with callbacks
```

But this is discouraged in Dart, whose async APIs all return Futures.