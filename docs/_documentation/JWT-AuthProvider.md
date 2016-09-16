---
slug: jwt-authprovider
---
The `JwtAuthProvider` is our integrated Auth solution for the popular [JSON Web Tokens](https://jwt.io/) (JWT) industry standard which is easily enabled by registering the `JwtAuthProvider` with the `AuthFeature` plugin:

```csharp
Plugins.Add(new AuthFeature(...,
    new IAuthProvider[] {
        new JwtAuthProvider(AppSettings) { AuthKey = AesUtils.CreateKey() },
        new CredentialsAuthProvider(AppSettings),
        //...
    }));
```

At a minimum you'll need to specify the `AuthKey` that will be used to Sign and Verify JWT tokens.
Whilst creating a new one in memory as above will work, a new Auth Key will be created every time the 
AppDomain recycles which will invalidate all existing JWT Tokens created with the previous key. 

So you'll typically want to generate the AuthKey out-of-band and configure it with the `JwtAuthProvider` at
registration which you can do in code using any of the [AppSettings providers](?id=AppSettings):

```csharp
new JwtAuthProvider(AppSettings) { 
    AuthKeyBase64 = AppSettings.GetString("AuthKeyBase64") 
}
```

Or alternatively you can configure most `JwtAuthProvider` properties in your **Web.config** `<appSettings/>` 
(default AppSettings Provider) following the `jwt.{PropertyName}` format:

```xml
<add key="jwt.AuthKeyBase64" value="{Base64AuthKey}" />
```

As with all crypto keys you'll want to keep them confidential as if anyone gets a hold of your AuthKey 
they'll be able to forge and sign their own JWT tokens letting them be able to impersonate any user, 
roles or permissions!

#### RequireSecureConnection

The JWT Auth Provider defaults to `RequireSecureConnection=true` which mandates for Authentication via either Provider to happen over a secure (HTTPS) connection as both bearer tokens should be kept highly confidential. You can specify `RequireSecureConnection=false` to disable this requirement for testing or within controlled internal environments.

### Sending JWT with Service Clients

JWT Tokens can be sent using the Bearer Token support in all HTTP and Service Clients:

```csharp
var client = new JsonServiceClient(baseUrl) {
    BearerToken = jwtToken
};

var response = await "https://example.org/secured".GetJsonFromUrlAsync(
    requestFilter: req => req.AddBearerToken(jwtToken));
```

The Service Clients offer additional high-level functionality where it's able to transparently request 
a new JWT Token after it expires by handling when the configured JWT Token becomes invalidated in the 
`OnAuthenticationRequired` callback. Here we can retrieve a new JWT Token that we can fetch using a 
different Service Client accessing a centralized and independent Auth Microservice that's configured with 
both API Key and JWT Token Auth Providers. We can fetch a new JWT Token by calling ServiceStack's 
built-in `Authenticate` Service with our **secret** API Key (that by default never invalidates unless revoked).

If authenticated, sending an empty `Authenticate()` DTO will return the currently Authenticated User Info 
that also generates a new JWT Token from the User's Authenticated Session and returns it in the `BearerToken` 
Response DTO property which we can use to update our invalidated JWT Token.

All together we can configure our Service Client to transparently refresh expired JWT Tokens with just:

```csharp
var authClient = JsonServiceClient(centralAuthBaseUrl) {
    Credentials = new NetworkCredential(apiKey, "")
};

var client = new JsonServiceClient(baseUrl);
client.OnAuthenticationRequired = () => {
    client.BearerToken = authClient.Send(new Authenticate()).BearerToken;
};
```

#### Sending JWT using Cookies

To [improve accessibility with Ajax clients](https://stormpath.com/blog/where-to-store-your-jwts-cookies-vs-html5-web-storage) JWT Tokens can also be sent using the `ss-tok` Cookie, e.g:

```csharp
var client = new JsonServiceClient(baseUrl);
client.SetCookie("ss-tok", jwtToken);

//Equivalent to: 
client.SetTokenCookie(jwtToken);
```

We'll walk through an example of how you can access JWT Tokens as well as how you can convert Authenticated
Sessions into JWT Tokens and assign it to use a **Secure** and **HttpOnly** Cookie later on.

### JWT Overview

A nice property of JWT tokens is that they allow for truly stateless authentication where API Keys and user 
credentials can be maintained in a decentralized Auth Service that's kept isolated from the rest of your 
System, making them optimal for use in Microservice architectures.

Being self-contained lends JWT tokens to more scalable, performant and flexible architectures as they don't 
require any I/O or any state to be accessed from App Servers to validate the JWT Tokens, this is unlike all 
other Auth Providers which requires at least a DB, Cache or Network hit to authenticate the user.

A good introduction into JWT is available from the JWT website: https://jwt.io/introduction/ whilst [JWT vs Sessions](https://float-middle.com/json-web-tokens-jwt-vs-sessions/) is a good article on advantages of using JWT instead of Sessions.

### JWT Format

Essentially JWT's consist of 3 parts separated by `.` with each part encoded in 
[Base64url Encoding](https://tools.ietf.org/html/rfc4648#section-5) making it safe to encode both text and
binary using only URL-safe (i.e. non-escaping required) chars in the following format:

    Base64UrlHeader.Base64UrlPayload.Base64UrlSignature 

Where just like the API Key, JWT's can be sent as a Bearer Token in the `Authorization` HTTP Request Header.

### JWT Header

The header typically consists of two parts: the type of the token and the hashing algorithm being used 
which is typically just:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

We also send the "kid" [Key Id](http://self-issued.info/docs/draft-jones-json-web-token-01.html#rfc.section.5.1)
used to identify which key should be used to validate the signature to help with seamless key rotations in 
future. If not specified the KeyId defaults to the **first 3 chars** of the Base64 HMAC or RSA Public Key Modulus.

### JWT Payload

The Payload contains the essential information of a JWT Token which is made up of "claims", i.e. 
statements and metadata about a user which are categorized into 3 groups: 

 - [Registered Claim Names](https://tools.ietf.org/html/rfc7519#section-4.1) - containing a known set of 
reserved names predefined in the JWT Standard
 - [Public Claim Names](https://tools.ietf.org/html/rfc7519#section-4.2) - additional common names that are
 registered in the [IANA "JSON Web Token Claims" registry](http://www.iana.org/assignments/jwt/jwt.xhtml) or
 otherwise adopt a Collision-Resistant name, e.g. prefixed by a namespace
 - [Private Claim Names](https://tools.ietf.org/html/rfc7519#section-4.3) - any other metadata you wish to
 include about an entity

We use the Payload to store essential information about the user which we use to validate the token and 
populate the session. Which typically contains:

 - **iss** ([Issuer](https://tools.ietf.org/html/rfc7519#section-4.1.1)) - the principal that issued the
 JWT. Can be set with `JwtAuthProvider.Issuer`, defaults to **ssjwt**
 - **sub** ([Subject](https://tools.ietf.org/html/rfc7519#section-4.1.2)) - identifies the subject of 
 the JWT, used to store the User's **UserAuthId**
 - **iat** ([Issued At](https://tools.ietf.org/html/rfc7519#section-4.1.6)) - when JWT Token was issued.
 Can use `InvalidateTokensIssuedBefore` to invalidate tokens issued before a specific date
 - **exp** ([Expiration Time](https://tools.ietf.org/html/rfc7519#section-4.1.4)) - when the JWT expires. 
 Initialized with `JwtAuthProvider.ExpireTokensIn` from date of issue (default **14 days**)
 - **aud** ([Audience](https://tools.ietf.org/html/rfc7519#section-4.1.3)) - identifies the recipient of
 the JWT. Can be set with `JwtAuthProvider.Audience`, defaults to `null` (Optional)

The remaining information in the JWT Payload is used to populate the Users Session, to maximize interoperability
we've used the most appropriate [Public Claim Names](http://www.iana.org/assignments/jwt/jwt.xhtml) 
where possible:

 - **email** <- `session.Email`
 - **given_name** <- `session.FirstName`
 - **family_name** <- `session.LastName`
 - **name** <- `session.DisplayName`
 - **preferred_username** <- `session.UserName`
 - **picture** <- `session.ProfileUrl`

We also need to capture Users Roles and Permissions but as there's no Public Claim Name for this yet we're using 
[Azure's Active Directory Conventions](https://azure.microsoft.com/en-us/documentation/articles/active-directory-token-and-claims/) where User Roles are stored in **roles** as a JSON Array and similarly, Permissions are stored in **perms**.

To keep the JWT Token small we're only storing the essential User Info above in the Token, which means when
the Token is restored it will only be partially populated. You can detect when a Session was partially 
populated from a JWT Token with the new `FromToken` boolean property.

#### Modifying the Payload

Whilst only limited info is embedded in the payload by default, all matching 
[AuthUserSession](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack/AuthUserSession.cs)
properties embedded in the token will also be populated on the Session, which you can add to the payload 
using the `CreatePayloadFilter` delegate. So if you also want to have access to when the user was registered 
you can add it to the payload with:

```csharp
new JwtAuthProvider(AppSettings) 
{
    CreatePayloadFilter = (payload,session) => 
        payload["CreatedAt"] = session.CreatedAt.ToUnixTime().ToString()
}
```

You can also use the filter to modify any existing property which you can use to change the behavior of the
JWT Token, e.g. we can add a special exception extending the JWT Expiration to all Users from Acme Inc with:

```csharp
new JwtAuthProvider(AppSettings) 
{
    CreatePayloadFilter = (payload,session) => {
        if (session.Email.EndsWith("@acme.com")) 
            payload["exp"] = DateTime.UtcNow.AddYears(1).ToUnixTime().ToString();
    }
}
```

Likewise you can modify JWT Headers with the `CreateHeaderFilter` delegate and modify how the Users Session
is populated with the `PopulateSessionFilter`.

### JWT Signature

JWT Tokens are possible courtesy of the cryptographic signature added to the end of the message that's used 
to Authenticate and Verify that a Message hasn't been tampered with. As long as the message signature 
validates with our `AuthKey` we can be certain the contents of the message haven't changed from when it was 
created by either ourselves or someone else with access to our AuthKey.

JWT standard allows for a number of different Hashing Algorithms although requires at least the **HM256** 
HMAC SHA-256 to be supported which is the default. The full list of Symmetric HMAC and Asymmetric RSA
Algorithms `JwtAuthProvider` supports include:

 - **HM256** - Symmetric HMAC SHA-256 algorithm
 - **HS384** - Symmetric HMAC SHA-384 algorithm
 - **HS512** - Symmetric HMAC SHA-512 algorithm
 - **RS256** - Asymmetric RSA with PKCS#1 padding with SHA-256
 - **RS384** - Asymmetric RSA with PKCS#1 padding with SHA-384
 - **RS512** - Asymmetric RSA with PKCS#1 padding with SHA-512

HMAC is the simplest to use as it lets you use the same AuthKey to Sign and Verify the message. 

But if preferred you can use a RSA Keys to sign and verify tokens by changing the `HashAlgorithm` and 
specifying a RSA Private Key:

```csharp
new JwtAuthProvider(AppSettings) { 
    HashAlgorithm = "RS256",
    PrivateKeyXml = AppSettings.GetString("PrivateKeyXml") 
}
```

If you don't have a RSA Private Key, one can be created with:

```csharp
var privateKey = RsaUtils.CreatePrivateKeyParams(RsaKeyLengths.Bit2048);
```

And its public key can be extracted using `ToPublicRsaParameters()` extension method, e.g:

```csharp
var publicKey = privateKey.ToPublicRsaParameters();
```

Then to serialize RSA Keys, you can then export them to XML with:

```csharp
var privateKeyXml = privateKey.ToPrivateKeyXml()
var publicKeyXml = privateKey.ToPublicKeyXml();
```

The behavior of using RSA to sign the JWT Tokens is mostly transparent but instead of using the AuthKey to 
both Sign and Verify the JWT Payload, it's signed with the Private Key and verified using the Public Key.
New tokens will also have the **alg** JWT Header set to **RS256** to reflect the new HashAlgorithm used.

### Encrypted JWE Tokens

Something that's not immediately obvious is that while JWT Tokens are signed to prevent tampering and 
verify authenticity, they're not encrypted and can easily be read by decoding the URL-safe Base64 string.
This is a feature of JWT where it allows Client Apps to inspect the User's claims and hide functionality
they don't have access to, it also means that JWT Tokens are debuggable and can be inspected for whenever 
you need to track down unexpected behavior.

But there may be times when you want to embed sensitive information in your JWT Tokens in which case you'll
want to enable Encryption, which can be done with:

```csharp
new JwtAuthProvider(AppSettings) { 
    PrivateKeyXml = AppSettings.GetString("PrivateKeyXml"),
    EncryptPayload = true
}
```

When turning on encryption, tokens are instead created following the
[JSON Web Encryption (JWE)](https://tools.ietf.org/html/rfc7516#section-3) standard where they'll be
encoded in the 5-part [JWE Compact Serialization](https://tools.ietf.org/html/rfc7516#section-3.1) format:

    BASE64URL(UTF8(JWE Protected Header)) || '.' ||
    BASE64URL(JWE Encrypted Key)          || '.' ||
    BASE64URL(JWE Initialization Vector)  || '.' ||
    BASE64URL(JWE Ciphertext)             || '.' ||
    BASE64URL(JWE Authentication Tag)

JwtAuthProvider's JWE implementation uses RSAES OAEP for Key Encryption and AES/128/CBC HMAC SHA256 for
Content Encryption, closely following 
[JWE's AES_128_CBC_HMAC_SHA_256 Example](https://tools.ietf.org/html/rfc7516#appendix-A.2)
where a new MAC Auth and AES Crypt Key and IV are created for each Token. The Content Encryption Key (CEK) 
used to Encrypt and Authenticate the payload is encrypted using the Public Key and decrypted with the 
Private Key so only Systems with access to the Private Key will be able to Decrypt, Validate and Read 
the Token's payload.

### Stateless Auth Microservices

One of JWT's most appealing features is its ability to decouple the System that provides User Authentication 
Services and issues tokens from all the other Systems but are still able provide protected Services although 
no longer needs access to a User database or Session data store to facilitate it, as sessions can now be 
embedded in Tokens and its state maintained and sent by clients instead of accessed from each App Server. 
This is ideal for Microservice architectures where Auth Services can be isolated into a single 
externalized System.

With this use-case in mind we've decoupled `JwtAuthProvider` in 2 classes:

 - [JwtAuthProviderReader](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack/Auth/JwtAuthProviderReader.cs) - 
Responsible for validating and creating Authenticated User Sessions from tokens
 - [JwtAuthProvider](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack/Auth/JwtAuthProvider.cs) -
Inherits `JwtAuthProviderReader` to also be able to Issue, Encrypt and provide access to tokens

#### Services only Validating Tokens

This lets us configure our Microservices that we want to enable Authentication via JWT Tokens down to just:

```csharp
public override void Configure(Container container)
{
    Plugins.Add(new AuthFeature(() => new AuthUserSession(),
        new IAuthProvider[] {
            new JwtAuthProviderReader(AppSettings) {
                HashAlgorithm = "RS256",
                PublicKeyXml = AppSettings.GetString("PublicKeyXml")
            },
        }));
}
```

Or if you want to just use a single AuthKey for both Issuing and Validating tokens:

```csharp
Plugins.Add(new AuthFeature(() => new AuthUserSession(),
    new [] { new JwtAuthProviderReader(AppSettings) }));
```

Which can be configured in [AppSettings](?id=AppSettings):

```xml
<add key="jwt.AuthKeyBase64" value="{Base64AuthKey}" />
```

Which no longer needs access to a 
[IUserAuthRepository](?id=Authentication-and-authorization#userauth-persistence---the-iuserauthrepository)
or [Sessions](?id=Sessions) since they're populated entirely
from JWT Tokens. Whilst you can use the default **HS256** HashAlgorithm, RSA is ideal for this use-case
as you can limit access to the **PrivateKey** to only the central Auth Service issuing the tokens and then 
only distribute the **PublicKey** to each Service which needs to validate them.

#### Service Issuing Tokens

As we can now contain all our Systems Auth Functionality to a single System we can open it up to support 
multiple Auth Providers as it only needs to be maintained in a central location but is still able to 
benefit all our Microservices that are only configured to validate JWT Tokens.

Here's a popular Auth Server configuration example which stores all User Auth information as well as
User Sessions in SQL Server and is configured to support many of ServiceStack's Auth and OAuth providers:

```csharp
public override void Configure(Container container)
{
    //Store UserAuth in SQL Server
    var dbFactory = new OrmLiteConnectionFactory(
        AppSettings.GetString("LiveDb"), SqlServerDialect.Provider);
        
    container.Register<IDbConnectionFactory>(dbFactory);
    container.Register<IAuthRepository>(c =>
        new OrmLiteAuthRepository(dbFactory) { UseDistinctRoleTables = true });

    //Create UserAuth RDBMS Tables
    container.Resolve<IAuthRepository>().InitSchema();

    //Also store User Sessions in SQL Server
    container.RegisterAs<OrmLiteCacheClient, ICacheClient>();
    container.Resolve<ICacheClient>().InitSchema();
    
    //Add Support for 
    Plugins.Add(new AuthFeature(() => new AuthUserSession(),
        new IAuthProvider[] {
            new JwtAuthProvider(AppSettings) {
                HashAlgorithm = "RS256",
                PrivateKeyXml = AppSettings.GetString("PrivateKeyXml")
            },
            new ApiKeyAuthProvider(AppSettings),        //Sign-in with API Key
            new CredentialsAuthProvider(),              //Sign-in with UserName/Password credentials
            new BasicAuthProvider(),                    //Sign-in with HTTP Basic Auth
            new DigestAuthProvider(AppSettings),        //Sign-in with HTTP Digest Auth
            new TwitterAuthProvider(AppSettings),       //Sign-in with Twitter
            new FacebookAuthProvider(AppSettings),      //Sign-in with Facebook
            new YahooOpenIdOAuthProvider(AppSettings),  //Sign-in with Yahoo OpenId
            new OpenIdOAuthProvider(AppSettings),       //Sign-in with Custom OpenId
            new GoogleOAuth2Provider(AppSettings),      //Sign-in with Google OAuth2 Provider
            new LinkedInOAuth2Provider(AppSettings),    //Sign-in with LinkedIn OAuth2 Provider
            new GithubAuthProvider(AppSettings),        //Sign-in with GitHub OAuth Provider
            new YandexAuthProvider(AppSettings),        //Sign-in with Yandex OAuth Provider        
            new VkAuthProvider(AppSettings),            //Sign-in with VK.com OAuth Provider 
        }));
}
```

With this setup we can Authenticate using any of the supported Auth Providers with our central Auth Server,
retrieve the generated Token and use it to communicate with any our Microservices configured to 
validate tokens:

#### Retrieve Token with API Key 

```csharp
var authClient = JsonServiceClient(centralAuthBaseUrl) {
    Credentials = new NetworkCredential(apiKey, "")
};

var jwtToken = authClient.Send(new Authenticate()).BearerToken;

var client = new JsonServiceClient(service1BaseUrl) { BearerToken = jwtToken };
var response = client.Get(new Secured { ... });
```

#### Retrieve Token with HTTP Basic Auth

```csharp
var authClient = JsonServiceClient(centralAuthBaseUrl) {
    Credentials = new NetworkCredential(username, password)
};

var jwtToken = authClient.Send(new Authenticate()).BearerToken;
```

#### Retrieve Token with Credentials Auth

```csharp
var authClient = JsonServiceClient(centralAuthBaseUrl);

var jwtToken = authClient.Send(new Authenticate {
    provider = "credentials",
    UserName = username,
    Password = password
}).BearerToken;
```

### Convert Sessions to Tokens

Another useful Service that `JwtAuthProvider` provides is being able to Convert your current Authenticated
Session into a Token. Authenticating via Credentials Auth establishes an **Authenticated Session** with the
server which is captured in the 
[Session Cookies](?id=Sessions#cookie-session-ids)
that gets populated on the HTTP Client. This lets us access protected Services immediately after we've
successfully Authenticated, e.g:

```csharp
var authResponse = client.Send(new Authenticate {
    provider = "credentials",
    UserName = username,
    Password = password
});

var response = client.Get(new Secured { ... });
```

### Request JWT Cookie is set on Authentication

However this only establishes an **Authenticated Session** to a single Server that only lasts until the
session stored on the Server is valid. The easiest way to tell ServiceStack to convert the Session into a
stateless JWT Cookie instead is to set the `UseTokenCookie` option when authenticating, e.g:

```csharp
var authResponse = client.Send(new Authenticate {
    provider = "credentials",
    UserName = username,
    Password = password,
    UseTokenCookie = true
});

//Uses stateless ss-tok Cookie with our Session encapsulated in JWT Token
var response = client.Get(new Secured { ... }); 
var jwtToken = client.GetTokenCookie(); //From ss-tok Cookie
```

This also removes the our Session from the App Servers Cache as now the Users **Authenticated Session** is
contained solely in the JWT Cookie and is valid until the JWT Cookies Expiration, instead of determined
by Server Session State.

Another way we can access our Token is to call the `ConvertSessionToToken` Service which also converts our 
currently Authenticated Session into a JWT Token which we can use instead to communicate with all our 
independent Services, e.g:

```csharp
var tokenResponse = client.Send(new ConvertSessionToToken());
var jwtToken = client.GetTokenCookie(); //From ss-tok Cookie

var client2 = new JsonServiceClient(service2BaseUrl) { BearerToken = jwtToken };
var response = client2.Get(new Secured2 { ... });

var client3 = new JsonServiceClient(service3BaseUrl) { BearerToken = jwtToken };
var response = client3.Get(new Secured3 { ... });
```

Tokens are returned in the Secure HttpOnly **ss-tok** Cookie, accessible from the `GetTokenCookie()` 
extension method as seen above.

The default behavior of `ConvertSessionToToken` is to remove the Current Session from the Auth Server 
which will prevent access to protected Services using our previously Authenticated Session. 
If you still want to preserve your existing Session you can indicate this with:

```csharp
var tokenResponse = client.Send(new ConvertSessionToToken { PreserveSession = true });
```

### Ajax Clients

Using Cookies is the 
[recommended way for using JWT Tokens in Web Applications](https://stormpath.com/blog/where-to-store-your-jwts-cookies-vs-html5-web-storage)
since the `HttpOnly` Cookie flag will prevent it from being accessible from JavaScript making them immune
to XSS attacks whilst the `Secure` flag will ensure that the JWT Token is only ever transmitted over HTTPS.

You can convert your Session into a Token and set the **ss-tok** Cookie in your web page by sending an Ajax 
request to `/session-to-token`, e.g:

```javascript
$.post("/session-to-token");
```

Likewise this API lets you convert Sessions created by any of the OAuth providers into a stateless JWT Token.

### Switching existing Sites to JWT

Thanks to the flexibility and benefits of using stateless JWT Tokens, we've upgraded both our Single Page App
http://techstacks.io which uses Twitter and Github OAuth to [use JWT with a single Ajax call](https://github.com/ServiceStackApps/TechStacks/blob/78ecd5e390e585c14f616bb27b24e0072b756040/src/TechStacks/TechStacks/js/user/services.js#L30):

```javascript
$.post("/session-to-token");
```

Whilst [Gistlyn uses the new Fetch API](https://github.com/ServiceStack/Gistlyn/blob/770768e7f7f6a7258ea948dbe1cee7f09b47ea8d/src/Gistlyn/src/app.tsx#L69) to convert an existing Github OAuth into a JWT Token Cookie:

```ts
fetch("/session-to-token", { method:"POST", credentials:"include" });
```

We've also upgraded https://servicestack.net which as it uses normal Username/Password Credentials Authentication 
(i.e. instead of redirects in OAuth), it doesn't need any additional network calls as we can add the `UseTokenCookie`
option as a hidden variable in our FORM request:

```html
<form id="form-login" action="/auth/login">
    <input type="hidden" name="UseTokenCookie" value="true" />
    ...
</form>
```

Which just like `ConvertSessionToToken` returns a populated session in the **ss-tok** Cookie so now 
both [techstacks.io](http://techstacks.io) and [servicestack.net](https://servicestack.net) can maintain 
uninterrupted Sessions across multiple redeployments without a persistent Sessions cache.

### Fallback Auth and RSA Keys

You can specify multiple fallback AES Auth Keys and RSA Public Keys to allow for smooth key rotations to newer Auth Keys whilst simultaneously being able to verify JWT Tokens signed with a previous key. 

The fallback keys can be configured in code when registering the `JwtAuthProvider`:

```csharp
new JwtAuthProvider {
    AuthKey = authKey2016,
    FallbackAuthKeys = {
        authKey2015,
        authKey2014,
    },
    PrivateKey = privateKey2016,
    FallbackPublicKeys = {
        publicKey2015,
        publicKey2014,
    },
}
```

Or in [AppSettings](?id=AppSettings):

```xml
<appSettings>
    <add key="jwt.AuthKeyBase64" value="{AuthKey2016Base64}" />
    <add key="jwt.AuthKeyBase64.1" value="{AuthKey2015Base64}" />
    <add key="jwt.AuthKeyBase64.2" value="{AuthKey2014Base64}" />

    <add key="jwt.PrivateKeyXml" value="{PrivateKey2016Xml}" />
    <add key="jwt.PublicKeyXml.1" value="{PublicKeyXml2015Xml}" />
    <add key="jwt.PublicKeyXml.2" value="{PublicKeyXml2014Xml}" />
</appSettings>
```

## JWT Configuration

The JWT Auth Provider provides the following options to customize its behavior: 

```csharp
class JwtAuthProviderReader
{
    // The RSA Bit Key Length to use
    static RsaKeyLengths UseRsaKeyLength = RsaKeyLengths.Bit2048

    // Different HMAC Algorithms supported
    Dictionary<string, Func<byte[], byte[], byte[]>> HmacAlgorithms

    // Different RSA Signing Algorithms supported
    Dictionary<string, Func<RSAParameters, byte[], byte[]>> RsaSignAlgorithms
    Dictionary<string, Func<RSAParameters, byte[], byte[], bool>> RsaVerifyAlgorithms

    // Whether to only allow access via API Key from a secure connection. (default true)
    bool RequireSecureConnection

    // Run custom filter after JWT Header is created
    Action<JsonObject, IAuthSession> CreateHeaderFilter

    // Run custom filter after JWT Payload is created
    Action<JsonObject, IAuthSession> CreatePayloadFilter

    // Run custom filter after session is restored from a JWT Token
    Action<IAuthSession, JsonObject, IRequest> PopulateSessionFilter

    // Whether to encrypt JWE Payload (default false). 
    // Uses RSA-OAEP for Key Encryption and AES/128/CBC HMAC SHA256 for Content Encryption
    bool EncryptPayload

    // Which Hash Algorithm should be used to sign the JWT Token. (default HS256)
    string HashAlgorithm

    // Whether to only allow processing of JWT Tokens using the configured HashAlgorithm.
    bool RequireHashAlgorithm

    // The Issuer to embed in the token. (default ssjwt)
    string Issuer

    // The Audience to embed in the token. (default null)
    string Audience

    // What Id to use to identify the Key used to sign the token. (3 chars of Base64 Key)
    string KeyId

    // The AuthKey used to sign the JWT Token
    byte[] AuthKey
    // Convenient overload to initialize AuthKey with Base64 string
    string AuthKeyBase64

    // The RSA Private Key used to Sign the JWT Token when RSA is used
    RSAParameters? PrivateKey
    // Convenient overload to intialize the Private Key via exported XML
    string PrivateKeyXml

    // The RSA Public Key used to Verify the JWT Token when RSA is used
    RSAParameters? PublicKey

    // Convenient overload to intialize the Public Key via exported XML
    string PublicKeyXml

    // How long should JWT Tokens be valid for. (default 14 days)
    TimeSpan ExpireTokensIn

    // Whether to invalidate all JWT Tokens issued before a specified date.
    DateTime? InvalidateTokensIssuedBefore

    // Whether to populate the Bearer Token in the AuthenticateResponse
    bool SetBearerTokenOnAuthenticateResponse

    // Modify the registration of ConvertSessionToToken Service
    Dictionary<Type, string[]> ServiceRoutes
}
```

### Further Examples

More examples of both the new API Key and JWT Auth Providers are available in 
[StatelessAuthTests](https://github.com/ServiceStack/ServiceStack/blob/master/tests/RazorRockstars.Console.Files/StatelessAuthTests.cs).
