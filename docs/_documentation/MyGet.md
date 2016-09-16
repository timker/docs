---
slug: myget
---
## ServiceStack pre-release MyGet Feed

Our interim pre-release NuGet packages first get published to [MyGet](https://www.myget.org/).

Instructions to add ServiceStack's MyGet feed to VS.NET are:

  1. Go to `Tools -> Options -> Package Manager -> Package Sources`
  2. Add the Source `https://www.myget.org/F/servicestack` with the name of your choice, e.g. _ServiceStack MyGet feed_

![NuGet Package Sources](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/wikis/myget/package-sources.png)

After registering the MyGet feed it will show up under NuGet package sources when opening the NuGet package manager dialog:

![NuGet Package Manager](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/wikis/myget/package-manager-ui.png)

Which will allow you to search and install pre-release packages from the selected MyGet feed.

## Redownloading MyGet packages

If you've already packages with the **same version number** from MyGet previously installed, you will need to manually delete the NuGet `/packages` folder for NuGet to pull down the latest packages.

### Clear NuGet Package Cache

To clear NuGet Package Cache go to `Tools -> Options` and click **Clear Package Cache**:

![Clear Packages Cache](https://raw.githubusercontent.com/ServiceStack/Assets/master/img/wikis/myget/clear-package-cache.png)

This option is not available in VS 2015+, instead delete the Cached Nuget packages with:

    del %LOCALAPPDATA%\NuGet\Cache\*.nupkg /q

If the above options do not work, try running the following command:

    nuget locals all -clear