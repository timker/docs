---
slug: formats
title: Content Types
---

ServiceStack supports the following formats:

- [JSON](https://github.com/ServiceStack/ServiceStack.Text)
- XML
- [SOAP 1.1/1.2](/soap-support)
- [JSV](https://github.com/ServiceStack/ServiceStack.Text#servicestacktypeserializer-and-the-jsv-format) _(hybrid CSV-style escaping + JSON format that is optimized for both size and speed)_
- [CSV](/csv-format)
- HTML
    - [Razor](http://razor.servicestack.net)
    - [Markdown Razor](/markdown-razor) _(Razor combined with markdown)_
    - [HTML5 Report](/html5reportformat) _(human-friendly HTML auto-layout letting you quickly visualize data returned by your services)_
- [Message Pack](/messagepack-format)
- [Protocol Buffers](/protobuf-format)
- [Wire](/wire-format)

## HTTP/REST Endpoints

You can define which format should be used by adding a `.{format}` extension:

 - `.json`
 - `.xml`
 - `.jsv`
 - `.csv`
 - `.html`

Or by appending `?format={format}` to the end of the URL:

- `?format=json`
- `?format=xml`
- `?format=jsv`
- `?format=csv`
- `?format=html`

> Example: http://test.servicestack.net/hello/World!?format=json

Alternatively ServiceStack also recognizes which format should be used with the `Accept` [http header](http://en.wikipedia.org/wiki/List_of_HTTP_header_fields):

- `Accept: application/json`
- `Accept: application/xml`

As you can see, this approach only works with `json` and `xml`.

## Pre-defined Routes

    /[xml|json|html|jsv|csv]/[reply|oneway]/[servicename]

Examples:

 - /json/reply/Hello (JSON)
 - /xml/oneway/SendEmail (XML)

## [SOAP Endpoint](/soap-support)

Consume ServiceStack Services via [SOAP](/soap-support) using WCF Add Service Reference or [ServiceStack generic SOAP Clients](/csharp-client#httpwebrequest-service-clients).

## [MQ Endpoint](/messaging)

Consume ServiceStack Services via [Message Queue](/messaging).
