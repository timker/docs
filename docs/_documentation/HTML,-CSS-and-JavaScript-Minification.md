---
slug: html-css-and-javascript-minification
title: HTML, CSS and JavaScript Minification
---

As part of our quest to provide a complete and productive solution for developing highly responsive Web and Single Page Apps, ServiceStack now includes minifiers for compressing HTML, CSS and JavaScript available from the new `Minifiers` class: 

```csharp
var minifiedJs = Minifiers.JavaScript.Compress(js);
var minifiedCss = Minifiers.Css.Compress(css);
var minifiedHtml = Minifiers.Html.Compress(html);

// Also minify in-line CSS and JavaScript
var advancedMinifiedHtml = Minifiers.HtmlAdvanced.Compress(html);
```

> Each minifier implements the lightweight [ICompressor](https://github.com/ServiceStack/ServiceStack/blob/master/src/ServiceStack.Interfaces/ICompressor.cs) interface making it trivial to mock or subtitute with a custom implementation.

#### JS Minifier
For the JavaScript minifier we're using [Ext.Net's C# port](https://github.com/extnet/Ext.NET.Utilities/blob/master/Ext.Net.Utilities/JavaScript/JSMin.cs) of Douglas Crockford's venerable [JSMin](http://crockford.com/javascript/jsmin). 

#### CSS Minifier
The CSS Minifer uses Mads Kristensen simple [CSS Minifer](http://madskristensen.net/post/efficient-stylesheet-minification-in-c). 

#### HTML Compressor
For compressing HTML we're using a [C# Port](http://blog.magerquark.de/c-port-of-googles-htmlcompressor-library/) of Google's excellent [HTML Compressor](https://code.google.com/p/htmlcompressor/) which we've further modified to remove the public API's ugly Java-esque idioms and replaced them with C# properties.

The `HtmlCompressor` also includes a number of well-documented options which can be customized by configuring the available properties on its concrete type, e.g:

```csharp
var htmlCompressor = (HtmlCompressor)Minifier.Html;
htmlCompressor.RemoveComments = false;
```

### Easy win for server generated websites

If your project is not based off one of our optimized Gulp/Grunt.js powered [React JS and AngularJS Single Page App templates](https://github.com/ServiceStack/ServiceStackVS) or configured to use our eariler [node.js-powered Bundler](https://github.com/ServiceStack/Bundler) Web Optimization solution, these built-in minifiers now offers the easiest solution to effortlessly optimize your existing website which is able to work transparently with your existing Razor Views and static `.js`, `.css` and `.html` files without requiring adding any additional external tooling or build steps to your existing development workflow.

### Minify dynamic Razor Views

Minification of Razor Views is easily enabled by specifying `MinifyHtml=true` when registering the `RazorFormat` plugin:

```csharp
Plugins.Add(new RazorFormat {
    MinifyHtml = true,
    UseAdvancedCompression = true,
});
```

Use the `UseAdvancedCompression=true` option if you also want to minify inline js/css, although as this requires a bit more processing you'll want to benchmark it to see if it's providing an overall performance benefit to end users. It's a recommended option if you're caching Razor Pages. Another solution is to minimize the use of in-line js/css and move them to static files to avoid needing in-line js/css compression.

### Minify static `.js`, `.css` and `.html` files

With nothing other than the new minifiers, we can leverage the flexibility in ServiceStack's [Virtual File System](?id=Virtual-file-system) to provide an elegant solution for minifying static `.html`, `.css` and `.js` resources by simply pre-loading a new InMemory Virtual FileSystem with minified versions of existing files and giving the Memory FS a higher precedence so any matching requests serve up the minified version first. We only need to pre-load the minified versions once on StartUp by overriding `GetVirtualFileSources()` in the AppHost:

```csharp
public override List<IVirtualPathProvider> GetVirtualFileSources()
{
    var existingProviders = base.GetVirtualFileSources();
    var memFs = new InMemoryVirtualPathProvider(this);

    //Get existing Local FileSystem Provider
    var fs = existingProviders.First(x => x is FileSystemVirtualPathProvider);

    //Process all .html files:
    foreach (var file in fs.GetAllMatchingFiles("*.html"))
    {
        var contents = Minifiers.HtmlAdvanced.Compress(file.ReadAllText());
        memFs.AddFile(file.VirtualPath, contents);
    }

    //Process all .css files:
    foreach (var file in fs.GetAllMatchingFiles("*.css")
      .Where(file => !file.VirtualPath.EndsWith(".min.css"))) //ignore pre-minified .css
    {
        var contents = Minifiers.Css.Compress(file.ReadAllText());
        memFs.AddFile(file.VirtualPath, contents);
    }

    //Process all .js files
    foreach (var file in fs.GetAllMatchingFiles("*.js")
      .Where(file => !file.VirtualPath.EndsWith(".min.js"))) //ignore pre-minified .js
    {
        try
        {
            var js = file.ReadAllText();
            var contents = Minifiers.JavaScript.Compress(js);
            memFs.AddFile(file.VirtualPath, contents);
        }
        catch (Exception ex)
        {
            //As JSMin is a strict subset of JavaScript, this can fail on valid JS.
            //We can report exceptions in StartUpErrors so they're visible in ?debug=requestinfo
            base.OnStartupException(new Exception("JSMin Error {0}: {1}".Fmt(file.VirtualPath, ex.Message)));
        }
    }

    //Give new Memory FS the highest priority
    existingProviders.Insert(0, memFs);
    return existingProviders;
}
```

A nice benefit of this approach is that it doesn't pollute your project with minified build artifacts, has excellent runtime performance with the minfied contents being served from Memory and as the file names remain the same, the links in HTML don't need to be rewritten to reference the minified versions. i.e. When a request is made it just looks through the registered virtual path providers and returns the first match, which given the Memory FS was inserted at the start of the list, returns the minified version.

### Enabled in [servicestack.net](https://servicestack.net)

As this was an quick and non-invasive feature to add, we've enabled it on all [servicestack.net](https://servicestack.net) Razor views and static files. You can `view-source:https://servicestack.net/` (as url in Chrome, Firefox or Opera) to see an example of the resulting minified output. 