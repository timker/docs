[Server Sent Events](http://www.html5rocks.com/en/tutorials/eventsource/basics/) (SSE) is an elegant [web technology](http://dev.w3.org/html5/eventsource/) for efficiently receiving push notifications from any HTTP Server. It can be thought of as a mix between long polling and one-way WebSockets and contains many benefits over each:

  - **Simple** - Server Sent Events is just a single long-lived HTTP Request that any HTTP Server can support
  - **Efficient** - Each client uses a single TCP connection and each message avoids the overhead of HTTP Connections and Headers that's [often faster than Web Sockets](http://matthiasnehlsen.com/blog/2013/05/01/server-sent-events-vs-websockets/).
  - **Resilient** - Browsers automatically detect when a connection is broken and automatically reconnects
  - **Interoperable** - As it's just plain-old HTTP, it's introspectable with your favorite HTTP Tools and even works through HTTP proxies (with buffering and [chunked-encoding turned off](http://www.w3.org/TR/2011/WD-eventsource-20111020/#notes)).
  - **Well Supported** - As a Web Standard it's supported in all major browsers except for IE which [can be enabled with polyfills](http://html5doctor.com/server-sent-events/#yaffle) - see [default_ieshim.cshtml](https://github.com/ServiceStackApps/Chat/blob/master/src/Chat/default_ieshim.cshtml) and its [Live Chat Example](http://chat.servicestack.net/default_ieshim).

We've chosen to adopt Server Sent Events for Server Notifications as it's a beautifully simple and elegant [Web Standard](http://dev.w3.org/html5/eventsource/) with better HTTP fidelity than **WebSockets**, that's perfect fit for Server Push Communications that works in both ServiceStack' ASP.NET and SelfHosts without requiring any extra .NET dependencies or [require the host Windows Server have WebSockets support](http://stackoverflow.com/a/12073593/85785) to use. 

### Server Event Clients

  - [JavaScript Client](https://github.com/ServiceStack/ServiceStack/wiki/JavaScript-Server-Events-Client)
  - [C# Client](https://github.com/ServiceStack/ServiceStack/wiki/C%23-Server-Events-Client)

### Server Event Providers

  - Memory Server Events (default)
  - [Redis Server Events](https://github.com/ServiceStack/ServiceStack/wiki/Redis-Server-Events)

## Registering

Like most other [modular functionality](https://github.com/ServiceStack/ServiceStack/wiki/Plugins) in ServiceStack, Server Sent Events is encapsulated in a single Plugin that can be registered in your AppHost with:

```csharp
Plugins.Add(new ServerEventsFeature());
```

The registration above is all that's needed for most use-cases which just uses the defaults below:

```csharp
class ServerEventsFeature
{
    StreamPath = "/event-stream";            // The entry-point for Server Sent Events
    HeartbeatPath = "/event-heartbeat";      // Where to send heartbeat pulses
    UnRegisterPath = "/event-unregister";    // Where to unregister your subscription
    SubscribersPath = "/event-subscribers";  // Where to view public info of channel subscribers 

    LimitToAuthenticatedUsers = false;       // Return `401 Unauthorized` to non-authenticated clients
    IdleTimeout = TimeSpan.FromSeconds(30);      // How long to wait for heartbeat before unsubscribing
    HeartbeatInterval = TimeSpan.FromSeconds(10); // Client Interval for sending heartbeat messages

    NotifyChannelOfSubscriptions = true;          // Send notifications when subscribers join/leave
}
```

> The paths allow you to customize the routes for the built-in Server Events API's, whilst setting either path to `null` disables that feature. 

There are also a number of hooks available providing entry points where custom logic can be added to modify or enhance existing behavior:

```csharp
class ServerEventsFeature
{
    Action<IEventSubscription, Dictionary<string, string>> OnConnect; //Filter OnConnect messages

    Action<IEventSubscription, IRequest> OnCreated;  // Fired when an Subscription is created
    Action<IEventSubscription> OnSubscribe;          // Fired when subscription is registered 
    Action<IEventSubscription> OnUnsubscribe;        // Fired when subscription is unregistered
}
```

## Sending Server Events

The way your Services send notifications is via the `IServerEvents` API which defaults to an in-memory `MemoryServerEvents` implementation which keeps a record of all subscriptions and connections in memory:

> Server Events can also be configured to use a [distributed Redis backend](https://github.com/ServiceStack/ServiceStack/wiki/Redis-Server-Events) which allows Server Events to work across load-balanced app servers.

```csharp
public interface IServerEvents : IDisposable
{
    // External API's
    void NotifyAll(string selector, object message);

    void NotifyChannel(string channel, string selector, object message);

    void NotifySubscription(string subscriptionId, string selector, object message, string channel = null);

    void NotifyUserId(string userId, string selector, object message, string channel = null);

    void NotifyUserName(string userName, string selector, object message, string channel = null);

    void NotifySession(string sspid, string selector, object message, string channel = null);

    SubscriptionInfo GetSubscriptionInfo(string id);

    List<SubscriptionInfo> GetSubscriptionInfosByUserId(string userId);

    // Admin API's
    void Register(IEventSubscription subscription, Dictionary<string, string> connectArgs = null);

    void UnRegister(string subscriptionId);

    long GetNextSequence(string sequenceId);

    // Client API's
    List<Dictionary<string, string>> GetSubscriptionsDetails(string channel = null);

    bool Pulse(string subscriptionId);

    // Clear all Registrations
    void Reset();
    void Start();
    void Stop();
}
```

The API's your Services predominantly deal with are the **External API's** which allow sending of messages at different levels of granularity. As Server Events have deep integration with ServiceStack's [Sessions](https://github.com/ServiceStack/ServiceStack/wiki/Sessions) and [Authentication Providers](https://github.com/ServiceStack/ServiceStack/wiki/Authentication-and-authorization) you're also able to notify specific users by either:

```csharp
NotifyUserId()   // UserAuthId
NotifyUserName() // UserName
NotifySession()  // Permanent Session Id (ss-pid)
```

Whilst these all provide different ways to send a message to a single authenticated user, any user can be connected to multiple subscriptions at any one time (e.g. by having multiple tabs open). Each one of these subscriptions is uniquely identified by a `subscriptionId` which you can send a message with using: 

```csharp
NotifySubscription() // Unique Subscription Id
```

There are also API's to retrieve a users single event subscription as well as all subscriptions for a user:

```csharp
IEventSubscription GetSubscription(string id);

List<IEventSubscription> GetSubscriptionsByUserId(string userId);
```

## Event Subscription

An Event Subscription allows you to inspect different metadata contained on each subscription as well as being able to `Publish()` messages directly to it, manually send a Heartbeat `Pulse()` (to keep the connection active) as well as `Unsubscribe()` to revoke the subscription and terminate the HTTP Connection.

```csharp
public interface IEventSubscription : IMeta, IDisposable
{
    DateTime CreatedAt { get; set; }
    DateTime LastPulseAt { get; set; }

    string Channel { get; }
    string UserId { get; }
    string UserName { get; }
    string DisplayName { get; }
    string SessionId { get; }
    string SubscriptionId { get; }
    bool IsAuthenticated { get; set; }

    Action<IEventSubscription> OnUnsubscribe { get; set; }
    void Unsubscribe();

    void Publish(string selector, object message);

    void Pulse();
}
```

The `IServerEvents` API also offers an API to UnRegister a subscription with:


```csharp
void UnRegister(IEventSubscription subscription);
```

## Channels

Standard Publish / Subscribe patterns include the concept of a **Channel** upon which to subscribe and publish messages to. The channel in Server Events can be any arbitrary string which is declared on the fly when it's first used. 

> As Request DTO names are unique in ServiceStack they also make good channel names which benefit from providing a typed API for free, e.g: `typeof(Request).Name`.

The API to send a message to a specific channel is:

```csharp
void NotifyChannel(string channel, string selector, object message);
```

Which just includes the name of the `channel`, the `selector` you wish the message applies to and the `message` to send which can be any JSON serializable object.

Along with being able to send a message to everyone on a channel, each API also offers an optional `channel` filter which when supplied will limit messages only to that channel:

```csharp
void NotifyUserId(string userId, string selector, object message, string channel = null);
```

## Order of Events

The following Server and Client callbacks are fired when a client first makes a Server Events Connection:

 1. `ServerEventsFeature.OnInit()` - Fired when the server receives the initial HTTP connection. This callback can be used to customize any HTTP Headers that are sent back to the client
 2. `ServerEventsFeature.OnCreated()` - Fired when the server `IEventSubscription` is created but before it becomes Connected.
 3. `ServerEventsFeature.OnConnect()` - Fired when the `IEventSubscription` is about to be connected. This callback can be used to modify the connection info arguments the client receives
 4. **(Client)**  - The Client is then sent an `cmd.onConnect` message with the connection info arguments about the connection. 
 5. `ServerEventsFeature.OnSubscribe()` - Fired after the subscription is registered. This callback can be used to send any custom messages to the client
 6. **(Client)** - If `ServerEventsFeature.NotifyChannelOfSubscriptions = true` every client in the same channel receives a `cmd.onJoin` message to notify them that a new subscription has joined the channel as well as a `cmd.onLeave` message when subscription leaves the channel

> The `cmd.onConnect`, `cmd.onJoin` and `cmd.onLeave` messages can be handled with the [Global Event Handlers](https://github.com/ServiceStack/ServiceStack/wiki/JavaScript-Server-Events-Client#global-event-handlers) on the JavaScript Client and the [Message Event Handlers](https://github.com/ServiceStack/ServiceStack/wiki/C%23-Server-Events-Client#message-event-handlers) or the [Global Receiver](https://github.com/ServiceStack/ServiceStack/wiki/C%23-Server-Events-Client#the-global-receiver) .NET ServerEventClient.

### Heartbeats

Periodically whilst the client maintains a ServerEvents subscription with the server it will send periodic Heartbeats to the Server. This will fire the `ServerEventsFeature.OnHeartbeatInit()` server event which if the client is still connected will be sent a `cmd.onHeartbeat` message. If the subscription no longer exists, heartbeats will return a `404 - Subscription {id} does not exist` HTTP Response.

## Chat Features

The implementation of Chat is a great way to explore different Server Event features which make it easy to develop highly interactive and responsive web apps with very little effort. 

### Active Subscribers

One feature common to chat clients is to get details of all the active subscribers in a channel which we can get from the built-in `/event-subscribers` route, e.g:

```javascript
$.getJSON("/event-subscribers?channel=@channel", function (users) {
    $.map(users, function(user) {
        usersMap[user.userId] = user;
        refCounter[user.userId] = (refCounter[user.userId] || 0) + 1;
    });
    var html = $.map(usersMap, function(user) { return createUser(user); }).join('');
    $("#users").html(html);
});
```

As a single user can have multiple subscriptions (e.g. multiple tabs open) users are merged into a single `usersMap` so each user is only listed once in the users list and a `refCounter` is maintained with the number of subscriptions each user has, so we're able to tell when the user has no more active subscriptions and can remove them from the list.

### Chat box

Chat's text box provides a free-text entry input to try out different Server Event features where each text message is posted to a ServiceStack Service which uses the `IServerEvents` API to send notifications the channels subscribers. When a server event is received on the client, the ss-utils.js client bindings routes the message to the appropriate handler. As all messages go through this same process, the moment the log entry appears in your chat window is also when it appears for everyone else (i.e instant when running localhost).

Normal chat messages (i.e. that don't specify a selector) uses the default `cmd.chat` selector which is sent to the `chat` handler that just echoes the entry into the chat log with:

```javascript
chat: function (m, e) {
    addEntry({ id: m.id, userId: m.fromUserId, userName: m.fromName, msg: m.message, 
               cls: m.private ? ' private' : '' });
}

```

### Specifying a selector

You can specify to use an alternative selector by prefixing the message with a `/{selector}`, e.g: 

    /cmd.announce This is your captain speaking ...

When a selector is specified in Chat it routes the message to the `/channels/{Channel}/raw` Service which passes the raw message through as a string. Normal Chat entries are instead posted to the `/channels/{Channel}/chat` Service, adding additional metadata to the chat message with the user id and name of the sender so it can be displayed in the chat log. The Javascript code that calls both Services is simply: 

```javascript
if (msg[0] == "/") {
    parts = $.ss.splitOnFirst(msg, " ");
    $.post("/channels/@channel/raw", 
        { from: activeSub.id, toUserId: to, message: parts[1], selector: parts[0].substring(1) });
} else {
    $.post("/channels/@channel/chat", 
        { from: activeSub.id, toUserId: to, message: msg, selector: "cmd.chat" });
}
```

### Sending a message to a specific user

Another special syntax supported in Chat is the ability to send messages to other users by prefixing it with `@` followed by the username, e.g:

    @mythz this is a private message
    @mythz /tv.watch http://youtu.be/518XP8prwZo

There's also a special `@me` alias to send a message to yourself, e.g:

    @me /tv.watch http://youtu.be/518XP8prwZo

## Server Event Services

By default ServiceStack doesn't expose any Services that can send notifications to other users by default.  It's left up to your application as to what functionality and level of granularity should be enabled for your Application. Your Services can send notifications via the `IServerEvents` provider.

Below is the annotated implementation for both Web Services used by Chat. The `PostRawToChannel` is a simple implementation that just relays the message sent to all users in the channel or just a specific user if `ToUserId` parameter is specified.

The `PostChatToChannel` Service is used for sending Chat messages which sends a wrapped `ChatMessage` DTO instead that holds additional metadata about the message that the Chat UI requires:

```csharp
[Route("/channels/{Channel}/chat")]
public class PostChatToChannel : IReturn<ChatMessage>
{
    public string From { get; set; }
    public string ToUserId { get; set; }
    public string Channel { get; set; }
    public string Message { get; set; }
    public string Selector { get; set; }
}

[Route("/channels/{Channel}/raw")]
public class PostRawToChannel : IReturnVoid
{
    public string From { get; set; }
    public string ToUserId { get; set; }
    public string Channel { get; set; }
    public string Message { get; set; }
    public string Selector { get; set; }
}

public class ServerEventsServices : Service
{
    public IServerEvents ServerEvents { get; set; }
    public IChatHistory ChatHistory { get; set; }
    public IAppSettings AppSettings { get; set; }

    public void Any(PostRawToChannel request)
    {
        if (!IsAuthenticated && AppSettings.Get("LimitRemoteControlToAuthenticatedUsers", false))
            throw new HttpError(HttpStatusCode.Forbidden, "You must be authenticated to use remote control.");

        // Ensure the subscription sending this notification is still active
        var sub = ServerEvents.GetSubscriptionInfo(request.From);
        if (sub == null)
            throw HttpError.NotFound("Subscription {0} does not exist".Fmt(request.From));

        // Check to see if this is a private message to a specific user
        if (request.ToUserId != null)
        {
            // Only notify that specific user
            ServerEvents.NotifyUserId(request.ToUserId, request.Selector, request.Message);
        }
        else
        {
            // Notify everyone in the channel for public messages
            ServerEvents.NotifyChannel(request.Channel, request.Selector, request.Message);
        }
    }

    public object Any(PostChatToChannel request)
    {
        // Ensure the subscription sending this notification is still active
        var sub = ServerEvents.GetSubscriptionInfo(request.From);
        if (sub == null)
            throw HttpError.NotFound("Subscription {0} does not exist".Fmt(request.From));

        var channel = request.Channel;

        // Create a DTO ChatMessage to hold all required info about this message
        var msg = new ChatMessage
        {
            Id = ChatHistory.GetNextMessageId(channel),
            FromUserId = sub.UserId,
            FromName = sub.DisplayName,
            Message = request.Message,
        };

        // Check to see if this is a private message to a specific user
        if (request.ToUserId != null)
        {
            // Mark the message as private so it can be displayed differently in Chat
            msg.Private = true;
            // Send the message to the specific user Id
            ServerEvents.NotifyUserId(request.ToUserId, request.Selector, msg);

            // Also provide UI feedback to the user sending the private message so they
            // can see what was sent. Relay it to all senders active subscriptions 
            var toSubs = ServerEvents.GetSubscriptionInfosByUserId(request.ToUserId);
            foreach (var toSub in toSubs)
            {
                // Change the message format to contain who the private message was sent to
                msg.Message = "@{0}: {1}".Fmt(toSub.DisplayName, msg.Message);
                ServerEvents.NotifySubscription(request.From, request.Selector, msg);
            }
        }
        else
        {
            // Notify everyone in the channel for public messages
            ServerEvents.NotifyChannel(request.Channel, request.Selector, msg);
        }

        if (!msg.Private)
            ChatHistory.Log(channel, msg);

        return msg;
    }
}
```

## Troubleshooting

### Response Buffering delaying events

If your web server is configured to automatically buffer the response it will delay when the [Server Events get sent to the client](http://stackoverflow.com/a/25983774/85785). In IIS Express you can disable buffering by disabling compression for dynamic requests by adding this to your **Web.config**:

```xml
<system.webServer>
   <urlCompression doStaticCompression="true" doDynamicCompression="false" />
</system.webServer>
```

Alternatively you can switch to use Visual Studio Development Server which doesn't buffer by default.

## ServerEvent Examples

## [Chat](https://github.com/ServiceStackApps/Chat)

> Feature-rich Single Page Chat App, showcasing Server Events support in 170 lines of JavaScript!

[![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/chat.png)](http://chat.servicestack.net)

## [React Chat](https://github.com/ServiceStackApps/Chat-React)

> React.js port of ServerEvents Chat using React, Reflux and new **ReactJS App** VS.NET Template

[![React Chat](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/chat-react-multichannels.png)](http://react-chat.servicestack.net?channels=home,work,play)

## [React Chat Desktop](https://github.com/ServiceStackApps/ReactChatApps)

> Built with [React Desktop Apps](https://github.com/ServiceStackApps/ReactDesktopApps)
VS.NET template and packaged into a native Desktop App for Windows and OSX - showcasing synchronized 
real-time control of multiple Windows Apps:

[![](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/livedemos/react-desktop-apps/dancing-windows.png)](https://youtu.be/-9kVqdPbqOM)

> Downloads for [Windows, OSX, Linux and Web](https://github.com/ServiceStackApps/ReactChatApps#downloads)