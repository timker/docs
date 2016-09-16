---
slug: authentication-and-authorization
---
Built into ServiceStack is an optional Authentication feature you can use to add Authentication to your services by providing web services to Authenticate existing users, Register new users as well Assign/UnAssign Roles to existing users (if you need them). It's highly pluggable and customizable where you can plug-in your own Auth logic, change the caching and session providers as well as what RDBMS is used to persist UserAuth data.

A minimal configuration needed to get Basic Authentication up and running is the following in `AppHost.Config()` (derived from the [AuthTests unit test](https://github.com/ServiceStack/ServiceStack/blob/master/tests/ServiceStack.WebHost.Endpoints.Tests/AuthTests.cs)):

```csharp 
public override void Configure(Container container)
{
    Plugins.Add(new AuthFeature(() => new AuthUserSession(),
      new IAuthProvider[] { 
        new BasicAuthProvider(), //Sign-in with HTTP Basic Auth
        new CredentialsAuthProvider(), //HTML Form post of UserName/Password credentials
      }));

    Plugins.Add(new RegistrationFeature());

    container.Register<ICacheClient>(new MemoryCacheClient());
    var userRep = new InMemoryAuthRepository();
    container.Register<IUserAuthRepository>(userRep);
    
    //The IUserAuthRepository is used to store the user credentials etc.
    //Implement this interface to adjust it to your app's data storage
}
```

The high-level overview below shows how all the different parts fit together and which parts are customizable:

![Authentication Overview](http://mono.servicestack.net/files/auth-overview.png)

### Auth Providers

From the overview we can see the built-in AuthProviders that are included:

  * **Credentials** - For authenticating with username/password credentials by posting to the `/auth/credentials` service
  * **API Keys** - Allowing users to authenticate with API Keys
  * **JWT Tokens** - Allowing users to authenticate with JWT Tokens
  * **Basic Auth** - Allowing users to authenticate with HTTP Basic Auth
  * **Digest Auth** - Allowing users to authenticate with HTTP Digest Authentication 
  * **Custom Credentials** - By inheriting CredentialsAuthProvider and providing your own Username/Password `TryAuthenticate` implementation
  * **AspNetWindowsAuthProvider** - Allowing users to authenticate with Windows Authentication
  * **Twitter OAuth** - Allow users to Register and Authenticate with Twitter
  * **Facebook OAuth** - Allow users to Register and Authenticate with Facebook 
  * **GitHub OAuth** - Allow users to Register and Authenticate with GitHub 
  * **Yammer OAuth** - Allow users to Register and Authenticate with Yammer
  * **Yandex OAuth** - Allow users to Register and Authenticate with Yandex
  * **VK OAuth** - Allow users to Register and Authenticate with VK
  * **Odnoklassni OAuth** - Allow users to Register and Authenticate with Odnoklassni

### OAuth2 Providers

The [ServiceStack.Authentication.OAuth2](https://nuget.org/packages/ServiceStack.Authentication.OAuth2) NuGet package provides OAuth2 Providers support to ServiceStack that includes:

  * **Instagram OAuth2** - Allow users to Register and Authenticate with Instagram OAuth2
  * **Google OAuth2** - Allow users to Register and Authenticate with Google OAuth2
  * **LinkedIn OAuth2** - Allow users to Register and Authenticate with LinkedIn OAuth2
  * **Microsoft Live OAuth2** - Allow users to Register and Authenticate with Microsoft Live OAuth2

### [OpenId Auth Providers](?id=OpenId)

The [ServiceStack.Authentication.OpenId](https://nuget.org/packages/ServiceStack.Authentication.OpenId) NuGet package provides OpenId Auth Providers support to ServiceStack that includes:

  * **Google OpenId** - Allow users to Register and Authenticate with Google
  * **Yahoo OpenId** - Allow users to Register and Authenticate with Yahoo
  * **MyOpenId** - Allow users to Register and Authenticate with MyOpenId
  * **OpenId** - Allow users to Register and Authenticate with **any** custom OpenId provider

### [API Key AuthProvider](?id=API-Key-AuthProvider)

The `ApiKeyAuthProvider` provides an alternative method for allowing external 3rd Parties access to your protected Services without needing to specify a password. 

### [JWT AuthProvider](?id=JWT-AuthProvider)

The `JwtAuthProvider` is our integrated stateless Auth solution for the popular [JSON Web Tokens](https://jwt.io/) (JWT) industry standard.

### Community Auth Providers

  - [Azure Active Directory](https://github.com/jfoshee/ServiceStack.Authentication.Aad) - Allow Custom App to login with Azure Active Directory
  - [ServiceStack.Authentication.IdentityServer](https://github.com/MacLeanElectrical/servicestack-authentication-identityserver) - Integration with ASP.NET IdentityServer and provides OpenIDConnect / OAuth 2.0 Single Sign-On Authentication

Find more info about [OpenId 2.0 providers on the wiki](?id=OpenId).

### OAuth Configuration

The OAuth providers below require you to register your application with them in order to get the `ConsumerKey` and `ConsumerSecret` required, at the urls below:

  - **Twitter** [dev.twitter.com/apps](https://dev.twitter.com/apps)
  - **Facebook** [developers.facebook.com/apps](https://developers.facebook.com/apps)
  - **Instagram** [instagram.com/developer/authentication](http://instagram.com/developer/authentication/)
  - **Google OAuth2** [code.google.com/apis/console/](https://code.google.com/apis/console/)
  - **LinkedIn OAuth2** [www.linkedin.com/secure/developer](https://www.linkedin.com/secure/developer)
  - **Microsoft Live OAuth2** [account.live.com/developers/applications](https://account.live.com/developers/applications)
  - **Yammer** [www.yammer.com/client_applications](http://www.yammer.com/client_applications)
  - **GitHub** [github.com/settings/applications/new](https://github.com/settings/applications/new)
  - **Yandex** [oauth.yandex.ru/client/new](https://oauth.yandex.ru/client/new)
  - **VK** [vk.com/editapp?act=create](http://vk.com/editapp?act=create)
  - **Odnoklassniki** [www.odnoklassniki.ru/devaccess](http://www.odnoklassniki.ru/devaccess)

[AuthWebTests](https://github.com/ServiceStack/ServiceStack/blob/master/tests/ServiceStack.AuthWeb.Tests/) is a simple project that shows all Auth Providers configured and working in the same app. See the [AppHost](https://github.com/ServiceStack/ServiceStack/blob/master/tests/ServiceStack.AuthWeb.Tests/AppHost.cs) for an example of the code and the [Web.config](https://github.com/ServiceStack/ServiceStack/blob/master/tests/ServiceStack.AuthWeb.Tests/Web.config) for an example of the configuration required to enable each Auth Provider.

Once you have the `ConsumerKey` and `ConsumerSecret` you need to configure it with your ServiceStack host, via [Web.config](https://github.com/ServiceStack/ServiceStack/blob/master/tests/ServiceStack.AuthWeb.Tests/Web.config), e.g:

    <add key="oauth.twitter.RedirectUrl"    value="http://yourhostname.com/"/>
    <add key="oauth.twitter.CallbackUrl"    value="http://yourhostname.com/auth/twitter"/>    
    <add key="oauth.twitter.ConsumerKey"    value="3H1FHjGbA1N0n0aT5yApA"/>
    <add key="oauth.twitter.ConsumerSecret" value="MLrZ0ujK6DwyjlRk2YLp6HwSdoBjtuqwXeHDQLv0Q"/>

> Note: Each OAuth Config option fallbacks to the configuration without the provider name. This is useful for reducing repetitive configuration that's shared by all OAuth providers like the `RedirectUrl` or `CallbackUrl`, e.g:

```xml
<add key="oauth.RedirectUrl"    value="http://yourhostname.com/"/>
<add key="oauth.CallbackUrl"    value="http://yourhostname.com/auth/{0}"/>    
```

Or via configuration in code when you register the `AuthFeature` in your AppHost, e.g:

```csharp
Plugins.Add(new AuthFeature(() => new AuthUserSession(), new IAuthProvider[] {
    new TwitterAuthProvider(appSettings) { 
        RedirectUrl = "http://yourhostname.com/",
        CallbackUrl = "http://yourhostname.com/auth/twitter",
        ConsumerKey = "3H1FHjGbA1N0n0aT5yApA",
        ConsumerSecret = "MLrZ0ujK6DwyjlRk2YLp6HwSdoBjtuqwXeHDQLv0Q",
    },
}));
```

> Note: The Callback URL in each Application should match the CallbackUrl for your application which is typically: http://yourhostname.com/auth/{Provider}, e.g. http://yourhostname.com/auth/twitter for Twitter.

### Built-In Auth Providers

By default the `CredentialsAuthProvider` and `BasicAuthProvider` validate against users stored in the UserAuth repository. The registration service at `/register` allow users to register new users with your service and stores them in your preferred `IUserAuthRepository` provider (below). The [SocialBootstrapApi](https://github.com/ServiceStack/SocialBootstrapApi) uses this to allow new users (without Twitter/Facebook accounts) to register with the website.

A good starting place to create your own Auth provider that relies on username/password validation is to subclass `CredentialsAuthProvider` and override the `bool TryAuthenticate(service, username, password)` hook so you can add in your own implementation. If you want to make this available via BasicAuth as well you will also need to subclass `BasicAuthProvider` with your own custom implementation.

### UserAuth Persistence - the [IUserAuthRepository](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack/Auth/IAuthRepository.cs#L19)

The Authentication module allows you to use your own persistence back-ends but for the most part you should be able to use one of the existing InMemory, Redis, OrmLite or MongoDB adapters. Use the OrmLite adapter if you want to store the Users Authentication information in any of the RDBMS's that [OrmLite](https://github.com/ServiceStack/ServiceStack.OrmLite) supports, which as of this writing includes Sql Server, Sqlite, MySql, PostgreSQL and Firebird. 

  - **OrmLite**: `OrmLiteAuthRepository` in [ServiceStack.Server](https://nuget.org/packages/ServiceStack.Server)
    - [OrmLiteAuthRepositoryMultitenancy](?id=Multitenancy#multitenancy-rdbms-authprovider)
  - **Redis**: `RedisAuthRepository` in [ServiceStack](https://nuget.org/packages/ServiceStack)
  - **In Memory**: `InMemoryAuthRepository` in [ServiceStack](https://nuget.org/packages/ServiceStack)
  - **AWS DynamoDB**: `DynamoDbAuthRepository` in [ServiceStack.Aws](https://nuget.org/packages/ServiceStack.Aws)
  - **Mongo DB**: `MongoDBAuthRepository` in [ServiceStack.Authentication.MongoDB](https://nuget.org/packages/ServiceStack.Authentication.MongoDB)
  - **Raven DB**: `RavenUserAuthRepository` in [ServiceStack.Authentication.RavenDB](https://nuget.org/packages/ServiceStack.Authentication.RavenDB)

### Caching / Sessions - the [ICacheClient](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/Caching/ICacheClient.cs)

Once authenticated the **AuthUserSession** model is populated and stored in the Cache using one of ServiceStack's [supported Caching providers](?id=Caching). ServiceStack's Sessions simply uses the `ICacheClient` API so any new provider added can be used for both Session and Caching options. Which currently include the following implementations:

  - **In Memory**: `MemoryCacheClient` in [ServiceStack](https://nuget.org/packages/ServiceStack)
  - **Redis**: `RedisClient`, `PooledRedisClientManager` or `BasicRedisClientManager` in [ServiceStack.Redis](https://nuget.org/packages/ServiceStack.Redis)
  - **Memcached**: `MemcachedClientCache` in [ServiceStack.Caching.Memcached](https://nuget.org/packages/ServiceStack.Caching.Memcached)
  - **AWS DynamoDB**: `DynamoDbCacheClient` in [ServiceStack.Aws](https://nuget.org/packages/ServiceStack.Aws)
  - **Azure**: `AzureCacheClient` in [ServiceStack.Caching.Azure](https://nuget.org/packages/ServiceStack.Caching.Azure)
The Auth Feature also allows you to specify your own custom `IUserAuthSession` type so you can attach additional metadata to your users session which will also get persisted and hydrated from the cache. 

> Note: If you're using Custom Sessions and have `JsConfig.ExcludeTypeInfo=true`, you need to [explicitly enable it](http://stackoverflow.com/q/18842685/85785) with `JsConfig<TCustomSession>.IncludeTypeInfo=true`.

After authentication the client will receive a cookie with a session id, which is used to fetch the correct session from the `ICacheClient` internally by ServiceStack. Thus, you can access the current session in a service:

```csharp
public class SecuredService : Service
{
    public object Get(Secured request)
    {
        IAuthSession session = this.GetSession();
        return new SecuredResponse() { Test = "You're" + session.FirstName };
    }
}
```

ServiceStack's Authentication, Caching and Session providers are completely new, clean, dependency-free testable APIs that doesn't rely on and is devoid of ASP.NET's existing membership, caching or session provider models. 

## [Live Demos](https://github.com/ServiceStackApps/LiveDemos)

To illustrate Authentication integration with ServiceStack, see the authentication-enabled live demos below:

  - [HttpBenchmarks Application](https://github.com/ServiceStackApps/HttpBenchmarks)
    - [Step-by-Step Authentication Guide](https://github.com/ServiceStackApps/HttpBenchmarks#authentication)
    - Twitter, Facebook, Google, LinkedIn and Credentials Auth
  - [AWS Auth](http://awsapps.servicestack.net/awsauth/) 
    - Twitter, Facebook, GitHub, Google, Yahoo, LinkedIn, and Credentials Auth
  - [MVC and WebForms Example](?id=ServiceStack-Integration) 
    - Twitter, Facebook, GitHub, Google, Yahoo, LinkedIn, VK, Credentials and Windows Auth
  - [Chat](https://github.com/ServiceStackApps/LiveDemos#chat)
    - Twitter and GitHub OAuth
  - [SocialBootstrap Api](https://github.com/ServiceStackApps/LiveDemos#social-bootstrap-api)
    - Twitter, Facebook, Yahoo and Credentials Auth

## Custom authentication and authorization

The classes in ServiceStack have been designed to provide default behavior out the box (convention over configuration). They are also highly customizable. Both the default [BasicAuthProvider](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack/Auth/BasicAuthProvider.cs) and [CredentialsAuthProvider](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack/Auth/CredentialsAuthProvider.cs) (which it extends) can be extended, and their behavior overwritten. An example is below:

```csharp
using ServiceStack;
using ServiceStack.Auth;

public class CustomCredentialsAuthProvider : CredentialsAuthProvider
{
    public override bool TryAuthenticate(IServiceBase authService, 
        string userName, string password)
    {
        //Add here your custom auth logic (database calls etc)
        //Return true if credentials are valid, otherwise false
    }

    public override IHttpResult OnAuthenticated(IServiceBase authService, 
        IAuthSession session, IAuthTokens tokens, 
        Dictionary<string, string> authInfo)
    {
        //Fill IAuthSession with data you want to retrieve in the app eg:
        session.FirstName = "some_firstname_from_db";
        //...

        //Call base method to Save Session and fire Auth/Session callbacks:
        return base.OnAuthenticated(authService, session, tokens, authInfo);

        //Alternatively avoid built-in behavior and explicitly save session with
        //authService.SaveSession(session, SessionExpiry);
        //return null;
    }
}
```
 
Then you need to register your custom credentials auth provider: 
 
```csharp
//Register all Authentication methods you want to enable for this web app.
Plugins.Add(new AuthFeature(() => new AuthUserSession(),
    new IAuthProvider[] {
        new CustomCredentialsAuthProvider(), //HTML Form post of User/Pass
    }
));
```

By default the AuthFeature plugin automatically registers the following (overrideable) Service Routes:
```csharp
new AuthFeature = {
  ServiceRoutes = new Dictionary<Type, string[]> {
    { typeof(AuthService), new[]{"/auth", "/auth/{provider}"} },
    { typeof(AssignRolesService), new[]{"/assignroles"} },
    { typeof(UnAssignRolesService), new[]{"/unassignroles"} },
  }
};
```

## Auth Provider Routes

The `AuthService` is registered at paths `/auth` and `/auth/{provider}` where the Provider maps to the `IAuthProvider.Provider` property of the registered AuthProviders. The urls for clients authenticating against the built-in AuthProviders are:

  * `/auth/credentials` - CredentialsAuthProvider
  * `/auth/apikey` - ApiKeyAuthProvider
  * `/auth/jwt` - JwtAuthProvider
  * `/auth/basic` - BasicAuthProvider
  * `/auth/twitter` - TwitterAuthProvider
  * `/auth/facebook` - FacebookAuthProvider
  * `/auth/instagram` - InstagramOAuth2Provider
  * `/auth/github` - GithubAuthProvider
  * `/auth/microsoftlive` - MicrosoftLiveOAuth2Provider
  * `/auth/yammer` - YammerAuthProvider
  * `/auth/googleopenid` - GoogleOpenIdOAuthProvider
  * `/auth/yahooopenid` - YahooOpenIdOAuthProvider
  * `/auth/myopenid` - MyOpenIdOAuthProvider
  * `/auth/openid` - OpenIdOAuthProvider (Any Custom OpenId provider)
  * `/auth/linkedin` - LinkedInOAuth2Provider
  * `/auth/googleoauth` - GoogleOAuth2Provider
  * `/auth/yandex` - YandexAuthProvider
  * `/auth/vkcom` - VkAuthProvider
  * `/auth/odnoklassniki` - OdnoklassnikiAuthProvider

### Logout

You can do a GET or POST to `/auth/logout` to logout the authenticated user or if you're using C# client you can logout with:

```csharp
client.Post(new Authenticate { provider = "logout" });
```

> Logging out will remove the Users Session from the Server and Session Cookies from the Client and redirect to the url in Authenticate.Continue, session.ReferrerUrl, 'Referer' HTTP Header or AuthProvider.CallbackUrl.

***

### Authenticating with .NET Service Clients

On the client you can use the [C#/.NET Service Clients](?id=CSharp-client) to easily consume your authenticated Services.

To authenticate using your `CustomCredentialsAuthProvider` by POST'ing a `Authenticate` Request, e.g:

    var client = new JsonServiceClient(BaseUrl);

    var authResponse = client.Post(new Authenticate {
        provider = CredentialsAuthProvider.Name, //= credentials
        UserName = "test@gmail.com",
        Password = "p@55w0rd",
        RememberMe = true,
    });

If authentication was successful the Service Client `client` instance will be populated with authenticated session cookies which then allows calling Authenticated services, e.g:

    var response = client.Get(new GetActiveUserId());

If you've also registered the `BasicAuthProvider` it will enable your Services to accept [HTTP Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) which is built-in the Service Clients that you can populate on the Service Client with:

    client.UserName = "test@gmail.com";
    client.Password = "p@55w0rd";

Which will also let you access protected Services, e.g:

    var response = client.Get(new GetActiveUserId());

Although behind-the-scenes it ends up making 2 requests, 1st request sends a normal request which will get rejected with a `401 Unauthorized` and if the Server indicates it has the `BasicAuthProvider` enabled it will resend the request with the HTTP Basic Auth credentials. 

You could instead save the latency of the additional auth challenge request by specifying the client should always send the Basic Auth with every request:

    client.AlwaysSendBasicAuthHeader = true;

### Authenticating with HTTP

To Authenticate with your `CustomCredentialsAuthProvider` (which inherits from CredentialsAuthProvider) you would POST:

`POST` localhost:60339/auth/credentials?format=json

    {
        "UserName": "admin",
        "Password": "test",
        "RememberMe": true
    }

When the client now tries to authenticate with the request above and the auth succeeded, the client will retrieve some cookies with a session id which identify the client on each Web Service call.

### User Sessions Cache

ServiceStack uses the [Cache Provider](?id=Caching) which was registered in the IoC container:

```csharp
//Register to use an In Memory Cache Provider (default)
container.Register<ICacheClient>(new MemoryCacheClient());

//Configure an alt. distributed persisted cache, E.g Redis:
//container.Register<IRedisClientsManager>(c => 
//    new RedisManagerPool("localhost:6379"));
```

> Tip: If you've got multiple servers which run the same ServiceStack service, you can use Redis to share the sessions between these servers.

***

Please look at [SocialBootstrapApi](https://github.com/ServiceStack/SocialBootstrapApi/tree/master/src/SocialBootstrapApi) to get a full example.

> Of course you can also implement your own - custom - authentication mechanism. You aren't forced to use the built-in ServiceStack auth mechanism.

## The `Authenticate` attribute

But how does ServiceStack know, which service needs authentication?
This happens with the [AuthenticateAttribute](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.ServiceInterface/AuthenticateAttribute.cs).

You simply have to mark your request dto with this attribute:

```csharp
//Authentication for all HTTP methods (GET, POST...) required
[Authenticate]
public class Secured
{
    public bool Test { get; set; }
}
```

Now this service can only be accessed if the client is authenticated:

```csharp
public class SecuredService : Service
{
    public object Get(Secured request)
    {
        IOAuthSession session = this.GetSession();
        return new SecuredResponse() { Test = "You're" + session.FirstName };
    }

    public object Put(Secured request)
    {
        return new SecuredResponse() { Test = "Valid!" };
    }

    public object Post(Secured request)
    {
        return new SecuredResponse() { Test = "Valid!" };
    }

    public object Delete(Secured request)
    {
        return new SecuredResponse() { Test = "Valid!" };
    }
}
```

If you want, that authentication is only required for GET and PUT requests for example, you have to provide some extra parameters to the `Authenticate` attribute.

```csharp
[Authenticate(ApplyTo.Get | ApplyTo.Put)] 
```

Of course `Authenticate` can also be placed on top of a service instead on top of a DTO, because it's a normal [filter attribute](?id=Filter-attributes).

## `RequiredRole` and `RequiredPermission` attributes

ServiceStack also includes a built-in permission based authorization mechanism. More details about how Roles and Permissions work is in this [StackOverflow Answer](http://stackoverflow.com/a/12096813).

Your request DTO can require specific permissions:

```csharp
[Authenticate]
//All HTTP (GET, POST...) methods need "CanAccess"
[RequiredRole("Admin")]
[RequiredPermission("CanAccess")]
[RequiredPermission(ApplyTo.Put | ApplyTo.Post, "CanAdd")]
[RequiredPermission(ApplyTo.Delete, "AdminRights", "CanDelete")]
public class Secured
{
    public bool Test { get; set; }
} 
```

Now the client needs the permissions...

- "CanAccess" to make a GET request
- "CanAccess", "CanAdd" to make a PUT/POST request
- "CanAccess", "AdminRights" and "CanDelete" to make a DELETE request

If instead you want to allow access to users in **ANY** Role or Permission use: 

```csharp
[RequiresAnyRole("Admin","Member")]
[RequiresAnyRole(ApplyTo.Post, "Admin","Owner","Member")]
[RequiresAnyPermission(ApplyTo.Delete, "AdminRights", "CanDelete")]
public class Secured
{
    public bool Test { get; set; }
} 
```

Normally ServiceStack calls the method `bool HasPermission(string permission)` in [IAuthSession](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.ServiceInterface/Auth/IAuthSession.cs). This method checks if the list `List<string> Permissions` in [IAuthSession](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.ServiceInterface/Auth/IAuthSession.cs) contains the required permission.

> [IAuthSession](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.ServiceInterface/Auth/IAuthSession.cs) is stored in a cache client as explained above

You can fill this list in the method `OnAuthenticated` you've overriden in the first part of this tutorial.

As with `Authenticate`, you can mark services (instead of DTO) with `RequiredPermission` attribute, too.

## Enabling Authentication at different levels

### Using the [Authenticate] Attribute

You can protect services by adding the `[Authenticate]` attribute on either the Action:

```csharp
class MyService : Service {
    [Authenticate] 
    public object Get(Protected request) { ... }
}
```

The Request DTO

```csharp
[Authenticate] 
class Protected { ... }
```

Or the service implementation

```csharp
[Authenticate] 
class MyService : Service {
    public object Get(Protected request) { ... }
}
```

Or by inheriting from a base class

```csharp
[Authenticate] 
class MyServiceBase : Service { ... }

class MyService : MyServiceBase {
    public object Get(Protected request) { ... }
}
```

### Using a Global Request Filter

Otherwise you can use a [global Request Filter](?id=Request-and-response-filters) if you wanted to restrict all requests any other way, e.g something like:

```csharp
appHost.RequestFilters.Add((httpReq, httpResp, requestDto) =>
{
    if (IsAProtectedPath(httpReq.PathInfo)) {
        new AuthenticateAttribute().Execute(httpReq, httpResp, requestDto);
    }
});
```
## Extending UserAuth tables

Different customization and extension points and strategies to extend the UserAuth tables with your own data are explained in this [StackOverflow answer](http://stackoverflow.com/a/11118747/85785).

## Customizing AuthProviders

### Customizing User Roles and Permissions

The default implementation of User Roles and Permissions on [AuthUserSession](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack/AuthUserSession.cs) shows how ServiceStack's `[RequiredRole]` and `[RequiredPermission]` [Roles and Permission attributes](?id=Authentication-and-authorization#the-requiredrole-and-requiredpermission-attributes) are validated:
 
```csharp
public virtual bool HasPermission(string permission)
{
    var managesRoles = HostContext.TryResolve<IAuthRepository>() as IManageRoles;
    if (managesRoles != null)
    {
        return managesRoles.HasPermission(this.UserAuthId, permission);
    }

    return this.Permissions != null && this.Permissions.Contains(permission);
}

public virtual bool HasRole(string role)
{
    var managesRoles = HostContext.TryResolve<IAuthRepository>() as IManageRoles;
    if (managesRoles != null)
    {
        return managesRoles.HasRole(this.UserAuthId, role);
    }

    return this.Roles != null && this.Roles.Contains(role);
}
```

These APIs are `virtual` so they can be overridden in both your Custom `AuthUserSession`. They default to looking at the `Roles` and `Permissions` collections stored on the Session. These collections are normally sourced from the `AuthUserRepository` when persisting the [UserAuth and UserAuthDetails POCO's](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack/Auth/UserAuth.cs) and are used to populate the `UserAuthSession` on successful Authentication. These collections can be further customized by AuthProviders which is what `AspNetWindowsAuthProvider` does to add [Authenticated WindowsAuth Roles](https://github.com/ServiceStack/ServiceStack/blob/9773b7fccc31ac4d715a02221f396b46cd14d7db/src/ServiceStack/Auth/AspNetWindowsAuthProvider.cs#L126).

As seen above Roles/Permissions can instead be managed by AuthProviders that implement `IManageRoles` API which is what OrmLiteAuthProvider uses to look at distinct RDBMS tables to validate Roles/Permissions:

### IManageRoles API

The [IManageRoles API](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack/Auth/IAuthRepository.cs#L26) 
can be implemented by any `IAuthRepository` to provide an alternative strategy for querying and managing Users Roles and permissions. 

This API is being used in the `OrmLiteAuthRepository` to provide an alternative way to store 
Roles and Permission in their own distinct table rather than being blobbed with the rest of the User Auth data. 
You can enable this new behavior by specifying `UseDistinctRoleTables=true` when registering the OrmLiteAuthRepository, e.g:

```csharp
container.Register<IAuthRepository>(c =>
    new OrmLiteAuthRepository(c.Resolve<IDbConnectionFactory>()) {
        UseDistinctRoleTables = true,
    });
```

When enabled, roles and permissions are persisted in the distinct **UserAuthRole** table instead of being blobbed on the UserAuth. The `IAuthSession.HasRole()` and `IAuthSession.HasPermission()` on the Users Session should be used to check if a User has a specified Role or Permission.
 
More examples of this are in [ManageRolesTests.cs](https://github.com/ServiceStack/ServiceStack/blob/master/tests/ServiceStack.Common.Tests/ManageRolesTests.cs).

#### CustomValidationFilter

The `CustomValidationFilter` on all AuthProviders lets you add post verification logic after a user has signed in with an OAuth provider and their OAuth metadata is retrieved. The filter lets you return a `IHttpResult` to control what error response is returned, e.g: 

```csharp
new FacebookAuthProvider(appSettings) { 
    CustomValidationFilter = authCtx => CustomIsValid(authCtx) 
        ? authCtx.Service.Redirect(authCtx.Session.ReferrerUrl
            .AddHashParam("f","CustomErrorCode"))
        : null,
}
```

Or could be used to redirect a network or users to a "Not Available in your Area" page with:

```csharp
Plugins.Add(new AuthFeature(..., 
    new IAuthProvider[] {
        new CredentialsAuthProvider {
            CustomValidationFilter = authCtx => 
                authCtx.Request.UserHostAddress.StartsWith("175.45.17")
                    ? HttpResult.Redirect("http://host.com/are-not-available")
                    : null
        }   
    }));
```

#### UserName Validation

The UserName validation for all Auth Repositories are configurable at:

```csharp
Plugins.Add(new AuthFeature(...){
    ValidUserNameRegEx = new Regex(@"^(?=.{3,20}$)([A-Za-z0-9][._-]?)*$", RegexOptions.Compiled),
})
```

Instead of RegEx you can choose to validate using a Custom Predicate. The example below ensures UserNames don't include specific chars:

```csharp
Plugins.Add(new AuthFeature(...){
    IsValidUsernameFn = userName => userName.IndexOfAny(new[] { '@', '.', ' ' }) == -1
})
```

#### Saving Extended OAuth Metadata

The new `SaveExtendedUserInfo` property (enabled by default) on all OAuth providers let you control whether to save the extended OAuth metadata available (into `UserAuthDetails.Items`) when logging in via OAuth.

#### MaxLoginAttempts

The `MaxLoginAttempts` feature lets you lock a User Account after multiple invalid login attempts, e.g:
 
```csharp
Plugins.Add(new AuthFeature(...) {
    MaxLoginAttempts = 5   // Lock user after 5 Invalid attempts
});
```

#### IAuthMetadataProvider

An IAuthMetadataProvider provides a way to customize the authInfo in all AuthProviders. It also allows overriding of how extended Auth metadata like profileUrl is returned.

```csharp
public interface IAuthMetadataProvider
{
   void AddMetadata(IAuthTokens tokens, Dictionary<string,string> authInfo);

   string GetProfileUrl(IAuthSession authSession, string defaultUrl = null);
}
```

> To override with a custom implementation, register `IAuthMetadataProvider` in the IOC

#### LoadUserAuthFilter

The LoadUserAuthFilter on `AspNetWindowsAuthProvider` lets you retrieve more detailed information about Windows Authenticated users during Windows Auth Authentication by using the .NET's ActiveDirectory services, e.g:

```csharp
//...
new AspNetWindowsAuthProvider(this) {
    LoadUserAuthFilter = LoadUserAuthInfo
}
//...

public void LoadUserAuthInfo(AuthUserSession userSession, 
    IAuthTokens tokens, Dictionary<string, string> authInfo)
{
    if (userSession == null) return;
    using (PrincipalContext pc = new PrincipalContext(ContextType.Domain))
    {
        var user = UserPrincipal.FindByIdentity(pc, userSession.UserAuthName);
        tokens.DisplayName = user.DisplayName;
        tokens.Email = user.EmailAddress;
        tokens.FirstName = user.GivenName;
        tokens.LastName = user.Surname;
        tokens.FullName = (String.IsNullOrWhiteSpace(user.MiddleName))
            ? "{0} {1}".Fmt(user.GivenName, user.Surname)
            : "{0} {1} {2}".Fmt(user.GivenName, user.MiddleName, user.Surname);
        tokens.PhoneNumber = user.VoiceTelephoneNumber;
    }
}
```

### In Process Authenticated Requests

You can enable the `CredentialsAuthProvider` to allow **In Process** requests to Authenticate without a Password with:

```csharp
new CredentialsAuthProvider {
    SkipPasswordVerificationForInProcessRequests = true,
}
```

When enabled this lets **In Process** Service Requests to login as a specified user without needing to provide their password. 

For example this could be used to create an [Intranet Restricted](?id=Restricting-Services) **Admin-Only** Service that lets you login as another user so you can debug their account without knowing their password with:

```csharp
[RequiredRole("Admin")]
[Restrict(InternalOnly=true)]
public class ImpersonateUser 
{
    public string UserName { get; set; }
}

public object Any(ImpersonateUser request)
{
    using (var service = base.ResolveService<AuthenticateService>()) //In Process
    {
        return service.Post(new Authenticate {
            provider = AuthenticateService.CredentialsProvider,
            UserName = request.UserName,
        });
    }
}
```

> Your Services can use the new `Request.IsInProcessRequest()` to identify Services that were executed in-process.

### Custom Hash Provider

The `IHashProvider` used to generate and verify password hashes and salts in each UserAuth Repository can be overridden with:

```csharp
container.Register<IHashProvider>(c => 
    new SaltedHash(HashAlgorithm:new SHA256Managed(), theSaltLength:4));
```

### Generate New Session Cookies on Authentication 

The AuthFeature also regenerates new Session Cookies each time users login, this behavior can be disabled with:

```csharp
Plugins.Add(new AuthFeature(...) {
    GenerateNewSessionCookiesOnAuthentication = false
});
```

### ClientId and ClientSecret OAuth Config Aliases
 
OAuth Providers can use `ClientId` and `ClientSecret` aliases instead of `ConsumerKey` and `ConsumerSecret`, e.g:

```xml 
<appSettings>
    <add key="oauth.twitter.ClientId" value="..." />
    <add key="oauth.twitter.ClientSecret" value="..." />
</appSettings>
```

<a name="community"></a>
# Community Resources
  - [Using IdentityServer 4 with ServiceStack and Angular](http://estynedwards.com/blog/2016/01/30/ServiceStack-IdentityServer-Angular/) by [@estynedwards](https://twitter.com/estynedwards)
  - [Adding Facebook Authentication using ServiceStack](http://buildclassifieds.com/2016/01/14/facebookauth/) by [@markholdt](https://twitter.com/markholdt)
  - [How to return JSV formatted collection types from SQL Server in OrmLite](http://blog.falafel.com/Blogs/adam-anderson/2013/10/28/how-to-return-jsv-formatted-collection-types-from-sql-server-to-servicestack.ormlite) by [AdamAnderson](http://blog.falafel.com/blogs/AdamAnderson)
  - [How to migrate ASP.NET Membership users to ServiceStack](http://blog.falafel.com/Blogs/adam-anderson/2013/10/23/how-to-migrate-asp.net-membership-users-to-servicestack) by [AdamAnderson](http://blog.falafel.com/blogs/AdamAnderson)
  - [Authentication in Service Stack REST Services](http://www.binaryforge-software.com/wpblog/?p=242) by [@binaryforge](https://twitter.com/binaryforge)
  - [Building a ServiceStack OAuth2 resource server using DotNetOpenAuth](http://dylanbeattie.blogspot.com/2013/08/building-servicestack-based-oauth2.html) by [@dylanbeattie](https://twitter.com/dylanbeattie)
  - [Declarative authorization in REST services in SharePoint with F#](http://sergeytihon.wordpress.com/2013/06/28/declarative-authorization-in-rest-services-in-sharepoint-with-f-and-servicestack/) by [@sergey_tihon](https://twitter.com/sergey_tihon)
  - [Authenticate ServiceStack services against an Umbraco membership provider](http://stackoverflow.com/a/16845317/85785) by [Gavin Faux](http://stackoverflow.com/users/1664508/gavin-faux)
  - [Using OAuth with ArcGIS Online and ServiceStack](http://davetimmins.com/post/2013/april/oauth-with-arcgisonline-servicestack) by [@davetimmins](https://twitter.com/davetimmins)
  - [LinkedIn Provider for ServiceStack Authentication](http://www.binoot.com/2013/03/30/linkedin-provider-for-servicestack-authentication/) by [@binu_thayamkery](https://twitter.com/binu_thayamkery)
  - [A Step by Step guide to create a Custom IAuthProvider](http://enehana.nohea.com/general/customizing-iauthprovider-for-servicestack-net-step-by-step/) by [@rngoodness](https://twitter.com/rngoodness)
  - [Simple API Key Authentication With ServiceStack](http://rossipedia.com/blog/2013/03/simple-api-key-authentication-with-servicestack/) by [@rossipedia](https://twitter.com/rossipedia)
  - [CORS BasicAuth on ServiceStack with custom authentication](http://joeriks.com/2013/01/12/cors-basicauth-on-servicestack-with-custom-authentication/) by [@joeriks](https://twitter.com/joeriks)
  - [Authenticating ServiceStack REST API using HMAC](http://jokecamp.wordpress.com/2012/12/16/authenticating-servicestack-rest-api-using-hmac/) by [@jokecamp](https://twitter.com/jokecamp)
  - ServiceStack Credentials Authentication and EasyHttp: [Part 1](http://blogs.lessthandot.com/index.php/DesktopDev/MSTech/servicestack-credentialsauthentication-and-easyhtpp-of), [Part 2](http://blogs.lessthandot.com/index.php/DesktopDev/MSTech/servicestack-credentialsauthentication-and-easyhtpp-of-1), [Part 3](http://blogs.lessthandot.com/index.php/DesktopDev/MSTech/servicestack-credentialsauthentication-and-easyhtpp-of-2) by [@chrissie1](https://twitter.com/chrissie1)