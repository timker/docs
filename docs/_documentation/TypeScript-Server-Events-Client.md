---
slug: typescript-server-events-client
title: TypeScript ServerEvents Client
---

[![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/release-notes/servicestack-heart-typescript.png)](https://github.com/ServiceStack/ServiceStack/wiki/TypeScript-Add-ServiceStack-Reference)

The TypeScript `ServerEventClient` is an idiomatic port of ServiceStack's 
[C# Server Events Client](/csharp-server-events-client) in native TypeScript providing a productive 
client to consume ServiceStack's [real-time Server Events](/server-events) that can be used in both 
TypeScript Web and node.js server applications.

## Install

`ServerEventClient` is available in the 
[servicestack-client npm package](https://github.com/ServiceStack/servicestack-client) 
which is preconfigured in all [ServiceStackVS TypeScript VS.NET Templates](https://github.com/ServiceStack/ServiceStackVS)

Other TypeScript or ES6 projects can install `servicestack-client` with:

    jspm install servicestack-client

node server projects can instead install it with:

    npm install servicestack-client --save

The Type Definitions are contained in the above `servicestack-client` npm package, if using jspm it 
can be installed with:

    npm install servicestack-client --save-dev

To configure Server Sent Events on the client create a new instance of `ServerEventsClient` with the
**baseUrl** and the **channels** you want to connect to, e.g:

```ts
const channels = ["home"];
const client = new ServerEventsClient("/", channels, {
    handlers: {
        onConnect: (sub:ServerEventConnect) => {  // Successful SSE connection
            console.log("You've connected! welcome " + sub.displayName);
        },
        onJoin: (msg:ServerEventJoin) => {        // User has joined subscribed channel
            console.log("Welcome, " + msg.displayName);
        },
        onLeave: (msg:ServerEventLeave) => {      // User has left subscribed channel
            console.log(user.displayName + " has left the building");
        },
        onUpdate: (msg:ServerEventUpdate) => {    // User
            console.log(user.displayName + " channels subscription were updated");
        },        
        onMessage: (msg:ServerEventMessage) => {}  // Invoked for each other message
        //... Register custom handlers
        CustomMessage: (msg:CustomMessage) = {}    // Handle CustomMessage Request DTO
    },
    receivers: { 
        //... Register any receivers
        tv: {
            watch: function (id) {                 // Handle 'tv.watch {url}' messages 
                var el = document.querySelector("#tv");
                if (id.indexOf('youtu.be') >= 0) {
                    var v = splitOnLast(id, '/')[1];
                    el.innerHTML = templates.youtube.replace("{id}", v);
                } else {
                    el.innerHTML = templates.generic.replace("{id}", id);
                }
                el.style.display = 'block'; 
            },
            off: function () {                     // Handle 'tv.off' messages
                var el = document.querySelector("#tv");
                el.style.display = 'none';
                el.innerHTML = '';
            }
        }
    }
}).start();
```

> If hosted from same ServiceStack Instance, the relative `/` url can be used instead of the absolute `baseUrl` 

ServiceStack Server Events has 4 built-in events sent during a subscriptions life-cycle: 

 - **onConnect** - sent when successfully connected, includes the subscriptions private `subscriptionId` as well as heartbeat and unregister urls that's used to automatically setup periodic heartbeats
 - **onJoin** - sent when a new user joins the channel
 - **onLeave** - sent when a user leaves the channel
 - **onUpdate** - sent when a user's channels subscription was updated

> The onJoin/onLeave/onUpdate events can be turned off with `ServerEventsFeature.NotifyChannelOfSubscriptions=false`.

All other messages can be handled with the catch-all:

 - **onMessage** - fired when any other message is sent

## Selectors

A selector is a string that identifies what should handle the message, it's used by the client to route the message to different handlers. The client bindings supports 4 different handlers out of the box:

### Global Event Handlers

The easiest way to handle a custom event is to define a handler, e.g:

```ts
const client = new ServerEventsClient("/", channels, {
    handlers: {
        paint: (color) => {
            this.style.background = color;
        }
    }
}).start();
```

The selector to invoke a global event handler is:

    cmd.{handler} {message}

Which can be sent in ServiceStack with:

```csharp
ServerEvents.NotifyChannel(channel, "cmd.paint", "green");
```

Where `{handler}` is the name of the handler you want to invoke, e.g `cmd.paint`. When invoked from a server event the message (deserialized from JSON) is the first argument, the Server Sent DOM Event is the 2nd argument and `this` by default is assigned to `document.body`.

```ts
function paint(msg /* JSON object msg */, e /*SSE Event*/){
    this // HTML Element or document.body
    this.style.background = "green";
},
```

#### Handling Messages with the Default Selector

All `IServerEvents` Notify API's includes [overloads for sending messages without a selector](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack/ServerEventsFeature.cs#L743-L771) that by convention will take the format `cmd.{TypeName}`. 

As they're prefixed with `cmd.*` these events can be handled with a handler **based on Message type name**, e.g:

```ts
const client = new ServerEventsClient("/", channels, {
    handlers: {
        CustomType: (msg:CustomType, e) => { ... },
        SetterType: (msg:SetterType, e) => { ... }
    }
}).start();
```

Which will be called when messages are sent without a selector, e.g:

```csharp
public class MyServices : Service
{
    public IServerEvents ServerEvents { get; set; }

    public void Any(Request request)
    {
        ServerEvents.NotifyChannel("home", new CustomType { ... });
        ServerEvents.NotifyChannel("home", new SetterType { ... });
    }
}
```

### Postfix CSS selector 

All server event handler options also support a postfix CSS selector for specifying what each handler should be bound to with a `$` followed by the CSS selector, e.g:

    cmd.{handler}${cssSelector}

A concrete example for calling the above API would be:

    cmd.paintGreen$#btnPaint

Which will bind `this` to the `#btnSubmit` HTML Element.

> Note: Spaces in CSS selectors need to be encoded with `%20`

### Modifying CSS

As it's a popular use-case Server Events also has native support for modifying CSS properties with:

    css.{propertyName}${cssSelector} {propertyValue}

Where the message is the property value, which roughly translates to:

    document.querySelectorAll({cssSelector}).style[{propertyName}] = {propertyValue}

When no CSS selector is specified it falls back to `document.body` by default.

    css.background #eceff1

Some other examples include:

    css.background$#top #673ab7   // $('#top').css('background','#673ab7')
    css.font$li bold 12px verdana // $('li').css('font','bold 12px verdana')
    css.visibility$a,img hidden   // $('a,img').css('visibility','#673ab7')
    css.visibility$a%20img hidden // $('a img').css('visibility','hidden')

### Receivers

In programming languages based on message-passing like Smalltalk and Objective-C invoking a method is done by sending a message to a receiver. This is conceptually equivalent to invoking a method on an instance in C# where both these statements are roughly equivalent:

```objc
// Objective-C
[receiver method:argument]
```

```csharp
// C#
receiver.method(argument)
```

Support for receivers is available in the following format:

    {receiver}.{target} {msg}

### Registering Receivers

Registering a receiver can be done with a map of the object instance and the name you want it to be exported as. E.g. The `window` and `document` global objects can be setup to receive messages with:

```ts
const client = new ServerEventsClient("/", channels, {
    receivers: {
        "window": window, 
        "document": document 
    }
}).start();
```

Alternatively you can add receivers at runtime with:

```ts
const client = new ServerEventsClient("/", channels)
    .registerNamedReceiver("window", window);
    .registerNamedReceiver("document", document)
    .start();
```

Once registered you can set any property or call any method on a receiver with:

    document.title New Window Title
    window.location http://google.com

Where if `{target}` was a function it will be invoked with the message, otherwise its property will be set.
By default when no `{cssSelector}` is defined, `this` is bound to the **receiver** instance.

Example of setting up a custom receiver:

```ts
const client = new ServerEventsClient("/", channels, {
    receivers: {
        tv: {
            watch: function (id) {
                var el = document.querySelector("#tv");
                if (id.indexOf('youtu.be') >= 0) {
                    var v = splitOnLast(id, '/')[1];
                    el.innerHTML = templates.youtube.replace("{id}", v);
                } else {
                    el.innerHTML = templates.generic.replace("{id}", id);
                }
                el.style.display = 'block'; 
            },
            off: function () {
                var el = document.querySelector("#tv");
                el.style.display = 'none';
                el.innerHTML = '';
            }
        }
    }
}).start();
```

This registers a custom `tv` receiver that can now be called with:

    tv.watch http://youtu.be/518XP8prwZo
    tv.watch https://servicestack.net/img/logo-220.png
    tv.off

### Receiver Constructors

In addition to registering an object instance as a receiver, you can also specify a `class` 
(aka constructor Function) instead, e.g:

```ts
class CssReceiver {
    backgroundImage(url:string) {
        document.body.style.backgroundImage = url;
    }
}

const client = new ServerEventsClient("/", channels, {
    receivers: {
        css: CssReceiver
    }
}).start();
```

Which will invoke `backgroundImage` method off a new instance of the `CssReceiver` class that's triggered with:

    css.background-image url(https://bit.ly/1yIJOBH)

and can be sent to all subscriptions on the **home** channel in ServiceStack with:

```csharp
ServerEvents.NotifyChannel("home", "css.background-image", "url(https://bit.ly/1yIJOBH)");
```

This works the same with Global Receivers, in addition Receivers can extend `ServerEventReceiver`:

```ts
class ServerEventReceiver implements IReceiver {
    public client: ServerEventsClient;
    public request: ServerEventMessage;
    noSuchMethod(selector: string, message:any) {}
}
```

To access additional built-in functionality where it will allow receivers to access the `ServerEventsClient` 
client dependency, the `ServerEventMessage` that was received, it also lets you handle any unhandled messages 
sent by implementing `noSuchMethod()`, e.g:

```ts
class JavaScriptReceiver extends ServerEventReceiver {

    chatMessage(msg: ChatMessage) {
        let request = new LogMessage();
        request.channel = msg.channel;
        request.message = msg.message;

        this.client.serviceClient.post(request)
            .then(r => {
                console.log(`logged ${msg.message} sent by ${msg.fromName} to ${msg.channel}`);
            });
    }
    
    announce(message:string) {
        alert(message);
    }

    toggle() {
        let el = document.querySelector(this.request.cssSelector);
        el.style.display = el.style.display != "none" ? "none" : "block";
    }

    noSuchMethod(selector:string, message:ServerEventMessage) {
        console.log(`Unhandled ${selector} was sent message: `, message);
    }
}

client.registerReceiver(JavaScriptReceiver); //register Global Receiver
```

These can triggered with:

```csharp
ServerEvents.NotifyChannel(channel, new ChatMessage { ... });
ServerEvents.NotifyChannel(channel, "cmd.announce", "Hello, World!");
ServerEvents.NotifyChannel(channel, "cmd.toggle$#sidebar");
ServerEvents.NotifyChannel(channel, "cmd.UnknownSelector", new Message { ... });
```

### Dependency Resolvers

You can control the lifetime of the receivers by injecting a custom resolver which will let you reuse the same
Receiver instance by using a `SingletonInstanceResolver`, e.g:

```ts
const client = new ServerEventsClient("/", channels)
    .setResolver(new SingletonInstanceResolver())
    .registerReceiver(JavaScriptReceiver)
    .start();
```

Which is implemented with:

```ts
export class SingletonInstanceResolver implements IResolver {
    tryResolve(ctor:ObjectConstructor): any {
        return (ctor as any).instance 
            || ((ctor as any).instance = new ctor());
    }
}
```

### Un Registering a Receiver

As receivers are maintained in a simple map, they can be disabled at anytime with:

```ts
client.unregisterReceiver("window");
```

and re-enabled with:

```ts
client.registerNamedReceiver("window", window);
```

Whilst Named Receivers are used to handle messages sent to a specific namespaced selector, the client also supports registering a **Global Receiver** for handling messages sent with the special `cmd.*` selector.

### Triggers

You can subscribe to triggers in the same way as handlers and receivers, e.g:

```ts
const client = new ServerEventsClient("/", channels, {
    triggers: {
        customEvent(msg, e) { 
            var target = this;
        }
    }
}).start();
```

The selector to trigger this event is:

    trigger.customEvent arg
    trigger.customEvent$#btnPaint arg

Where if no CSS selector is specified it defaults to `document`. 

### Channel Subscriber APIs

You can use any of the APIs below to update an active Subscriptions Channels:

```ts
client.subscribeToChannels("chan3","chan4");
client.unsubscribeFromChannels("chan1","chan2");

//Alternatively subscribe/unsubscribe to channels in the same request with:
var request = new UpdateEventSubscriber();
request.subscribeChannels = ["chan3","chan4"];
request.unsubscribeChannels = ["chan1","chan2"];
client.updateSubscriber(request);
```

### Get Channel Subscribers

Once connected, you can get a list of channel subscribers the `ServerEventsClient` is currently connected
to with:

```csharp
client.getChannelSubscribers()
    .then(users => users.forEach(x => 
        console.log(`#${x.userId} @${x.displayName} ${x.profileUrl} ${x.channels}`)));
```

### Accessing ServiceClient

Alternatively you can access the channel subscribers using the built-in `JsonServiceClient`, e.g:

```csharp
let request = new GetEventSubscribers();
request.channels = ["chan1","chan2"];

client.serviceClient.get(request)
    .then(r => ...);
```

Which you can also use to call your own Services:

```ts
client.serviceClient.post(new MyRequest())
    .then(r => ...)
```

### Integration Test Examples

More examples of the `ServerEventClient` can be found in the [TypeScript ServerEventClient Test Suite](https://github.com/ServiceStack/servicestack-client/blob/master/tests/serverevents.spec.ts).

# ServerEvent JavaScript Examples

## [Gistlyn](https://github.com/ServiceStack/Gistlyn)

Gistlyn is a C# Gist IDE for creating, running and sharing stand-alone, executable C# snippets.

[![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/gistlyn/home-screenshot.png)](http://gistlyn.com)

> Live Demo: http://gistlyn.com

## [React Chat](https://github.com/ServiceStackApps/ReactChat)

React Chat is a port of [ServiceStack Chat](https://github.com/ServiceStackApps/Chat) ES5, jQuery Server Events 
demo into a [TypeScript](http://www.typescriptlang.org/), [React](http://facebook.github.io/react/) and 
[Redux](https://github.com/reactjs/redux) App:

[![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/chat-react/screenshot.png)](http://react-chat.servicestack.net/)

## [Networked Time Traveller Shape Creator](https://github.com/ServiceStackApps/typescript-redux#example-9---real-time-networked-time-traveller)

A network-enhanced version of the
[stand-alone Time Traveller Shape Creator](https://github.com/ServiceStackApps/typescript-redux#example-8---time-travelling-using-state-snapshots)
that allows users to **connect to** and **watch** other users using the App in real-time similar 
to how users can use Remote Desktop to watch another computer's screen: 

[![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/redux-chrome-safari.png)](http://redux.servicestack.net)

> Live demo: http://redux.servicestack.net

## [Chat](https://github.com/ServiceStackApps/Chat)

> Feature-rich Single Page Chat App, showcasing Server Events support in 170 lines of JavaScript!

[![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/chat.png)](http://chat.servicestack.net)

## [React Chat Desktop](https://github.com/ServiceStackApps/ReactChatApps)

> Built with [React Desktop Apps](https://github.com/ServiceStackApps/ReactDesktopApps)
VS.NET template and packaged into a native Desktop App for Windows and OSX - showcasing synchronized 
real-time control of multiple Windows Apps:

[![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/react-desktop-apps/dancing-windows.png)](https://youtu.be/-9kVqdPbqOM)

> Downloads for [Windows, OSX, Linux and Web](https://github.com/ServiceStackApps/ReactChatApps#downloads)

