In order to access physical files in view engines from multiple sources, ServiceStack includes its own pluggable virtual file system API that lets it support multiple filesystem backends. 

The virtual file system (VFS) is what allows ServiceStack to support view engines in a standard ASP.NET websites (e.g. serving directories from the root directory) as well in self-hosting stand-alone HttpListener websites and Windows Services serving from the output `/bin` directory as well as embedded resources inside .dlls, any combination of both and in future from remote datastores.

## Embedded Resources

To enable ServiceStack to serve embedded resources in your Website's compiled `.dll` Assembly you'll need to register either an `Assembly` or a `Type` in an Assembly that contains embedded resources, e.g:

```csharp
SetConfig(new HostConfig {
   EmbeddedResourceSources = { typeof(TypeInDllWithEmbeddedResources).Assembly },
   EmbeddedResourceBaseTypes = { typeof(TypeInDllWithEmbeddedResources) } 
});
```

By default ServiceStack automatically includes the Assembly where your `AppHost` is defined which since it's typically the same top-level assembly where all your Website assets are maintained, no configuration is required to serve any embedded resources which are accessible from the same path as it's defined in your VS.NET project. E.g. if you have an embedded resource in your project at `/dir/file.js` it would be available from the same path where ServiceStack is mounted, e.g `http://localhost:1337/dir/file.js`.

### [ServiceStack.Gap](https://github.com/ServiceStack/ServiceStack.Gap)

See the ServiceStack.Gap project for different examples of how to create single **.exe** ILMerged applications with Embedded Resources and Compiled Razor Views.

### Registering additional Virtual Path Providers

An easy way to register additional VirtualPath Providers in ServiceStack is to override the `GetVirtualFileSources()` method in your AppHost where you can add, remove, or re-order existing providers to change their priority. E.g. we can use this to provide an elegant solution for minifying static `.html`, `.css` and `.js` resources by simply pre-loading a new **InMemory Virtual FileSystem** with minified versions of existing files and giving the Memory FS a higher precedence so any matching requests serve up the minified version first with:

```csharp
public override List<IVirtualPathProvider> GetVirtualFileSources()
{
    var existingProviders = base.GetVirtualFileSources();
    var memFs = new InMemoryVirtualPathProvider(this);

    //Get FileSystem Provider
    var fs = existingProviders.First(x => x is FileSystemVirtualPathProvider);

    //Process all .html files:
    foreach (var file in fs.GetAllMatchingFiles("*.html"))
    {
        var contents = Minifiers.HtmlAdvanced.Compress(file.ReadAllText());
        memFs.WriteFile(file.VirtualPath, contents);
    }

    //Process all .css files:
    foreach (var file in fs.GetAllMatchingFiles("*.css")
        .Where(file => !file.VirtualPath.EndsWith(".min.css")))
    {
        var contents = Minifiers.Css.Compress(file.ReadAllText());
        memFs.WriteFile(file.VirtualPath, contents);
    }

    //Process all .js files
    foreach (var file in fs.GetAllMatchingFiles("*.js")
        .Where(file => !file.VirtualPath.EndsWith(".min.js")))
    {
        try
        {
            var js = file.ReadAllText();
            var contents = Minifiers.JavaScript.Compress(js);
            memFs.WriteFile(file.VirtualPath, contents);
        }
        catch (Exception ex)
        {
            //Report any errors in StartUpErrors collection on ?debug=requestinfo
            base.OnStartupException(new Exception(
                "JSMin Error in {0}: {1}".Fmt(file.VirtualPath, ex.Message)));
        }
    }

    //Give new Memory FS highest priority
    existingProviders.Insert(0, memFs);
    return existingProviders;
}
```

### Using a different Virtual Path Provider

You can also globally replace the VFS used by setting it in your AppHost, e.g. If you only want to use an InMemory File System:

```csharp
base.VirtualPathProvider = new InMemoryVirtualPathProvider(this);
```

Fine-grained control on which VFS to use can also be specified on any [Plugins](https://github.com/ServiceStack/ServiceStack/wiki/Plugins) requiring access to the FileSystem like ServiceStack's built-in HTML ViewEngines, here's how you could override the VFS used in ServiceStack's Razors support:

```csharp
Plugins.Add(new RazorFormat { 
  VirtualPathProvider = new InMemoryVirtualPathProvider(this) 
});
```

### Changing Physical File Path

You can change the physical root path from where ServiceStack serves your files from by changing `Config.WebHostPhysicalPath`, e.g. the current directory for self-hosts is where the **.exe** is run from, during development this is typically `\bin\Release`. You can change the self-host to serve files from your project folder with:

```csharp
SetConfig(new HostConfig {
    WebHostPhysicalPath = "~/".MapProjectPath()
});
```

> Where `string.MapProjectPath()` is just an extension method that goes back 2 directories `~\..\..` - resolving the project folder from the Debug/Release bin folders.

## Overriding Embedded Resources with Static Files

The VFS supports multiple file source locations where you can override embedded files by including your own custom files in the same location as the embedded files. We can see how this works by overriding the built-in templates used in metadata pages:

### Updating HTML and Metadata Page Templates

The HTML templates for the metadata pages are maintained as [embedded html template resources](https://github.com/ServiceStack/ServiceStack/tree/master/src/ServiceStack/Templates). 

The VFS lets you replace built-in ServiceStack templates with your own by simply copying the metadata or [HtmlFormat Template files](http://bit.ly/164YbrQ) you want to customize and placing them in your Website Directory at:

    /Templates/HtmlFormat.html        // The auto HtmlFormat template
    /Templates/IndexOperations.html   // The /metadata template
    /Templates/OperationControl.html  // Individual operation template

Which you can customize locally that ServiceStack will pick up and use instead.

## Writable Virtual File System

The Virtual File System extended `IVirtualFiles` interface extends the read-only `IVirtualPathProvider` interface to offer a read/write API:

```csharp
public interface IVirtualFiles : IVirtualPathProvider
{
  void WriteFile(string filePath, string textContents);
  void WriteFile(string filePath, Stream stream);
  void WriteFiles(IEnumerable<IVirtualFile> files,Func<IVirtualFile,string> toPath=null);
  void DeleteFile(string filePath);
  void DeleteFiles(IEnumerable<string> filePaths);
  void DeleteFolder(string dirPath);
}
```

> Folders are implicitly created when writing a file to folders that don't exist

The new `IVirtualFiles` API is available in local FileSystem, In Memory and S3 Virtual path providers:

 - FileSystemVirtualPathProvider
 - InMemoryVirtualPathProvider
 - S3VirtualPathProvider

All `IVirtualFiles` providers share the same 
[VirtualPathProviderTests](https://github.com/ServiceStack/ServiceStack.Aws/blob/master/tests/ServiceStack.Aws.Tests/S3/VirtualPathProviderTests.cs)
ensuring a consistent behavior where it's now possible to swap between different file storage backends with simple
configuration as seen in the [Imgur](#imgur) and [REST Files](#restfiles) examples.

### VirtualFiles vs VirtualFileSources

As typically when saving uploaded files you'd only want files written to a single explicit File Storage provider,
ServiceStack keeps a distinction between the existing read-only Virtual File Sources it uses internally whenever a 
static file is requested and the new `IVirtualFiles` which is maintained in a separate `VirtualFiles` property on 
`IAppHost` and `Service` base class for easy accessibility:

```csharp
public class IAppHost
{
    // Read/Write Virtual FileSystem. Defaults to Local FileSystem.
    IVirtualFiles VirtualFiles { get; set; }
    
    // Cascading file sources, inc. Embedded Resources, File System, In Memory, S3.
    IVirtualPathProvider VirtualFileSources { get; set; }
}

public class Service : IService //ServiceStack's convenient concrete base class
{
    //...
    public IVirtualPathProvider VirtualFiles { get; }
    public IVirtualPathProvider VirtualFileSources { get; }
}
```

Internally ServiceStack only uses `VirtualFileSources` itself to serve static file requests. 
The new `IVirtualFiles` is a clean abstraction your Services can bind to when saving uploaded files which can be easily
substituted when you want to change file storage backends. If not specified, `VirtualFiles` defaults to your local 
filesystem at your host project's root directory.

### Examples

 - [AWS RazorRockstars](https://github.com/ServiceStack/ServiceStack.Aws#maintain-website-content-in-s3) - Serving all Razor Views and Markdown Content from a S3 bucket
 - [AWS Imgur and REST Files](https://github.com/ServiceStack/ServiceStack.Aws#aws-imgur) - 1 line configuration switch between saving files to local files or S3 Bucket


## Implementing a new Virtual File System

The VFS is designed to be implementation agnostic so can be changed to use any file repository, e.g. it could easily be made to support a Redis, RDBMS, embedded Sqlite or other NoSQL back-ends.

Like most of ServiceStack's substitutable API's, the interfaces for the VFS lives in the **ServiceStack.Interfaces.dll** under the [ServiceStack.IO](https://github.com/ServiceStack/ServiceStack/tree/master/src/ServiceStack.Interfaces/IO) namespace.

These are the API's that are needed to be implemented in order to create a new VFS:

```csharp
public interface IVirtualPathProvider
{
    IVirtualDirectory RootDirectory { get; }
    string VirtualPathSeparator { get; }
    string RealPathSeparator { get; }

    string CombineVirtualPath(string basePath, string relativePath);

    bool FileExists(string virtualPath);
    bool DirectoryExists(string virtualPath);

    IVirtualFile GetFile(string virtualPath);
    string GetFileHash(string virtualPath);
    string GetFileHash(IVirtualFile virtualFile);

    IVirtualDirectory GetDirectory(string virtualPath);

    IEnumerable<IVirtualFile> GetAllMatchingFiles(
        string globPattern, int maxDepth = Int32.MaxValue);

    bool IsSharedFile(IVirtualFile virtualFile);
    bool IsViewFile(IVirtualFile virtualFile);
}

public interface IVirtualNode
{
    IVirtualDirectory Directory { get; }
    string Name { get; }
    string VirtualPath { get; }
    string RealPath { get; }
    bool IsDirectory { get; }
    DateTime LastModified { get; }
}

public interface IVirtualFile : IVirtualNode
{
    IVirtualPathProvider VirtualPathProvider { get; }
    string Extension { get; }
    string GetFileHash();
    Stream OpenRead();
    StreamReader OpenText();
    string ReadAllText();
}

public interface IVirtualDirectory : IVirtualNode, IEnumerable<IVirtualNode>
{
    bool IsRoot { get; }
    IVirtualDirectory ParentDirectory { get; }

    IEnumerable<IVirtualFile> Files { get; }
    IEnumerable<IVirtualDirectory> Directories { get; }

    IVirtualFile GetFile(string virtualPath);
    IVirtualFile GetFile(Stack<string> virtualPath);

    IVirtualDirectory GetDirectory(string virtualPath);
    IVirtualDirectory GetDirectory(Stack<string> virtualPath);

    IEnumerable<IVirtualFile> GetAllMatchingFiles(
        string globPattern, int maxDepth = Int32.MaxValue);
}
```