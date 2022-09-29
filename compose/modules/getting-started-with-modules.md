# 👍 Getting started with modules

## Overview

* Smartstore modules are designed to extend Smartstore's functionality in any way you can imagine.
* Modules can bring in any desired feature, no limits.
* Modules can alter the way the app operates, change workflows, change or extend UI, overwrite services etc.
* Modules are sets of extensions that can be packaged in order to be re-used on other Smartstore shops.
* A module is a regular _Class Library_ project that compiles into a single assembly. It may use Smartstore APIs (but it doesn't necessarily have to).
* The only two requirements for a module project are:
  * `module.json`: a manifest file describing the metadata of a module
  * `Module` class: implements `IModule` and contains (un)installation routines&#x20;
* Some special - mostly commerce related - features are encapsulated as _providers_ in the Smartstore ecosystem_._ A module can expose as many of these providers as you like, or none.
* Provider types are (represented by their interfaces):
  * `IPaymentMethod` (provides payment provider like PayPal, AmazonPay, offline payment etc.)
  * `ITaxProvider` (tax calculation)
  * `IShippingRateComputationMethod` (calculates shipping fees)
  * `IExportProvider` (exports data)
  * `IMediaStorageProvider` (provides storage for media file blobs)
  * `IOutputCacheProvider` (provides storage for output cache items)
  * `IWidget` (provides content to render in UI)
  * `IExternalAuthenticationMethod` (provides an external authenticator like Google, Facebook etc.)
  * `IExchangeRateProvider` (fetches currency live rates)

## Module structure

* A module is a regular _Class Library_ project in the solution
* The project should be placed in the **/src/Smartstore.Modules** directory in the root of your solution
  * WARN: Do not get this confused with the **/src/Smartstore.Web/Modules** directory which is the build target for modules (from where the application dynamically loads module assemblies into the app-domain)
* If your project directory is located elsewhere, you should create a _symlink_ that links to the actual location.
  * INFO: It is good practice to use the **-sym** suffix in symlink sources, because they are git-ignored.
* For module projects we recommend the naming convention **{Organization}.{ModuleName}**, but you can choose any name you wish. This, at the same time, should be the root namespace name and the module system name.
* Assuming your module is called **MyOrg.MyGreatModule**, the _.csproj_ file should look like this:

```xml
<Project Sdk="Microsoft.NET.Sdk.Razor">
    <PropertyGroup>
        <Product>A great module for Smartstore</Product>
        <OutputPath>..\..\Smartstore.Web\Modules\MyOrg.MyGreatModule</OutputPath>
        <OutDir>$(OutputPath)</OutDir>
    </PropertyGroup>
</Project>
```

* Every time the solution gets build, your module will be compiled and copied to the `OutputPath` directory specified in the project file.
* The file _Smartstore.Build/Smartstore.Module.props_ defines some shared props for modules. You don't need to include this file: it is auto-included in every project that is located in the _Smartstore.Modules_ directory.&#x20;
* Among other things it specifies that some known files and directories should always be copied to the output path (if they exist): _module.json_, _wwwroot_, _Localization_, _Views_.
* It also specifies that dependencies should **not** be copied to the output directory. This is important because otherwise the build process would copy the whole dependency graph to the output, which produces way too much noise and file redundancy.

### Project & Package references

* All projects located in the _Smartstore.Modules_ directory reference `Smartstore`, `Smartstore.Core` and `Smartstore.Web.Common` projects by default.
* You can also reference `Smartstore.Web` (e.g. to access model types that are declared there), but in this case you should tell your project not to copy dependant stuff over to your output path by adding following lines to the .csproj file:

```xml
<ItemGroup>
    <ProjectReference Include="..\..\Smartstore.Web\Smartstore.Web.csproj">
        <Private>False</Private>
        <CopyLocal>False</CopyLocal>
        <CopyLocalSatelliteAssemblies>False</CopyLocalSatelliteAssemblies>
    </ProjectReference>
</ItemGroup>
```

* You can reference any NuGet package you wish. But special consideration is required for packages that are private (private = package is not referenced by app core)...
* ...Remember: we completely suppress copying of dependencies. To copy the files anyway, you have to declare them in `module.json` file (read further below).

### module.json manifest

* This file is describing your module to the system
* The information contained in this file will be used for example in the _Plugin Manager_ administration screen
* Here is how a _module.json_ may look like
* `SystemName`, `FriendlyName` and `Version` are required.

{% code title="module.json" %}
```json
{
  "$schema": "../module.schema.json",
  "Author": "MyOrg",
  "Group": "Payment",
  // Required. Module system name.
  "SystemName": "MyOrg.MyGreatModule",
  // Required. English friendly name.
  "FriendlyName": "A great module for Smartstore",
  "Description": "Lorem ipsum",
  // Required. The current version of module.
  "Version": "5.0",
  "MinAppVersion": "5.0",
  "Order": 1,
  "ResourceRootKey": "Plugins.Payments.MyGreatModule",
  "ProjectUrl": "https://myorg.com",
  "PrivateReferences": [
    "MiniProfiler.Shared",
    "MiniProfiler.AspNetCore",
    "MiniProfiler.AspNetCore.Mvc",
    "MiniProfiler.EntityFrameworkCore"
  ]
}
```
{% endcode %}

#### Schema reference

| Name                  | Type    | Description                                                                                                                                                                                                                                                                                                                       |
| --------------------- | ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **AssemblyName**      | string  | Optional assembly name. By default, '{SystemName}.dll' is assumed to be the assembly name.                                                                                                                                                                                                                                        |
| **Author**            | string  | Author name.                                                                                                                                                                                                                                                                                                                      |
| **FriendlyName \***   | string  | English friendly name.                                                                                                                                                                                                                                                                                                            |
| **Group**             | enum    | Conceptual group name. Used to visually categorize module in module listings.  Possible values: _Admin, Marketing, Payment, Shipping, Tax, Analytics, CMS, Media, SEO, Data, Globalization, Api, Mobile, Social, Security, Developer, Sales, Design, Performance, B2B, Storefront, Law_.                                          |
| **DependsOn**         | array   | Optional array of module system names that a module depends on.                                                                                                                                                                                                                                                                   |
| **Description**       | string  | English description.                                                                                                                                                                                                                                                                                                              |
| **MinAppVersion**     | string  | The minimum compatible application version, e.g. '5.0.0'. The module will be unavailable if the current app version is less than this value.                                                                                                                                                                                      |
| **Order**             | integer | The display order in the module manager group.                                                                                                                                                                                                                                                                                    |
| **PrivateReferences** | array   | Optional array of private dependency package names that a module references. By default, referenced packages are not copied to the module's output directory because it is assumed that the application core references them already. Any module private package should be listed here (including the complete dependency chain). |
| **ProjectUrl**        | string  | Link to project's or author's homepage.                                                                                                                                                                                                                                                                                           |
| **ResourceRootKey**   | string  | Root key for language resources (see [localizing-modules.md](localizing-modules.md "mention")).                                                                                                                                                                                                                                   |
| **SystemName \***     | string  | Module system name (usually the assembly name without extension).                                                                                                                                                                                                                                                                 |
| **Tags**              | string  | Comma-separated tags.                                                                                                                                                                                                                                                                                                             |
| **Version \***        | string  | The current version of module, e.g. '5.1.0'.                                                                                                                                                                                                                                                                                      |

### Module entry class

* Every module needs an entry class which (at least) contains methods to install and uninstall the module.
* The class must implement `IModule`
* By convention, we place this file in the root of the project, name it `Module.cs` and make it internal
* We also recommend to derive from abstract [ModuleBase](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Modularity/ModuleBase.cs) instead of implementing `IModule,` because `ModuleBase` contains some common implementation already.
* Your module entry class may also implement any provider interface directly (like e.g. `IPaymentMethod`). This is the recommended approach if your module contains exactly one feature provider.
* It may also implement `IConfigurable` to expose the route to a module configuration page that should be linked in the _Plugin Manager_ UI.
* Every time the module is installed, `IModule.InstallAsync()` method will be called...
* ...and if the module is uninstalled, `IModule.UninstallAsync()` method will be called
* TIPP: It is good practice NOT to delete any custom module data from database during uninstallation in case the user wants to re-install the module later. But it's up to you.

{% code title="Module.cs" %}
```csharp
internal class Module : ModuleBase, IConfigurable
{
    public RouteInfo GetConfigurationRoute()
        => new("Configure", "MyGreatAdmin", new { area = "Admin" });

    public override async Task InstallAsync(ModuleInstallationContext context)
    {
        // Saves the default state of a settings class to the database 
        // without overwriting existing values.
        await TrySaveSettingsAsync<MyGreatModuleSettings>();
        
        // Imports all language resources for the current module from 
        // xml files in "Localization" directory (if any found).
        await ImportLanguageResourcesAsync();
        
        // VERY IMPORTANT! Don't forget to call.
        await base.InstallAsync(context);
    }

    public override async Task UninstallAsync()
    {
        // Deletes all "MyGreatModuleSettings" properties settings from the database.
        await DeleteSettingsAsync<MyGreatModuleSettings>();
        
        // Deletes all language resource for the current module 
        // if "ResourceRootKey" is module.json is not empty.
        await DeleteLanguageResourcesAsync();
        
        // VERY IMPORTANT! Don't forget to call.
        await base.UninstallAsync();
    }
}
```
{% endcode %}

## Files & Folders Best Practices

* There are some conventions how we organize the file structure within a project
* There is no obligation for you to comply to it, but it makes things more - well, organized - and predictable
* The following is a list of files & folders (maxed out)

| Entry                              | Description                                                                                                                          |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| :file\_folder: **App\_Data**       | App specific (cargo) data like templates, sample files etc. that needs to be published.                                              |
| :file\_folder: **Blocks**          | Page Builder Block implementations (see [page-builder-and-blocks.md](../../framework/content/page-builder-and-blocks.md "mention")). |
| :file\_folder: **Bootstrapping**   | Bootstrapping code like _Autofac_ modules, DI extensions etc.                                                                        |
| :file\_folder: **Client**          | RESTful API clients                                                                                                                  |
| ****:file\_folder: **Components**  | MVC View Components (see [controllers-and-viewcomponents.md](controllers-and-viewcomponents.md "mention"))                           |
| :file\_folder: **Configuration**   | Settings class implementations (see [configuration.md](../../framework/platform/configuration.md "mention"))                         |
| :file\_folder: **Controllers**     | MVC Controllers (see [controllers-and-viewcomponents.md](controllers-and-viewcomponents.md "mention"))                               |
| :file\_folder: **Domain**          | Domain entities (see [domain.md](../../getting-started/domain.md "mention"))                                                         |
| :file\_folder: **Extensions**      | Static extension method classes                                                                                                      |
| :file\_folder: **Filters**         | MVC Filters (see [filters.md](filters.md "mention"))                                                                                 |
| :file\_folder: **Hooks**           | Hook implementations (see [hooks.md](../../framework/platform/hooks.md "mention"))                                                   |
| :file\_folder: **Localization**    | Localization files (see [localizing-modules.md](localizing-modules.md "mention"))                                                    |
| :file\_folder: **Media**           | Media system related classes                                                                                                         |
| :file\_folder: **Migrations**      | Fluent data migrations (see [database-migrations.md](../../framework/platform/database-migrations.md "mention"))                     |
| :file\_folder: **Models**          | View model classes (see [data-modelling](../../framework/platform/data-modelling/ "mention"))                                        |
| :file\_folder: **Providers**       | Provider implementations                                                                                                             |
| :file\_folder: **Security**        | Security related classes                                                                                                             |
| :file\_folder: **Services**        | Service classes                                                                                                                      |
| :file\_folder: **Utils**           | Utilities                                                                                                                            |
| :file\_folder: **Tasks**           | Task scheduler jobs (see [scheduling.md](../../framework/platform/scheduling.md "mention"))                                          |
| :file\_folder: **TagHelpers**      | Tag Helpers                                                                                                                          |
| :file\_folder: **Views**           | Razor view/template files                                                                                                            |
| :file\_folder: **wwwroot**         | Static files (including Sass)                                                                                                        |
| :file\_cabinet: AdminMenu.cs       | Admin menu hook (see [menus.md](../../framework/content/menus.md "mention"))                                                         |
| :file\_cabinet: CacheableRoutes.cs | Route registrar for output cache (see [output-cache.md](../../framework/platform/output-cache.md "mention"))                         |
| :file\_cabinet: Events.cs          | Event handler methods (see [events.md](../../framework/platform/events.md "mention"))                                                |
| :file\_cabinet: Module.cs \*       | Required. Module entry class (implements `IModule`).                                                                                 |
| :file\_cabinet: module.json \*     | Required. Module metadata manifest.                                                                                                  |
| :file\_cabinet: Permissions.cs     | Module permissions (see [security.md](../../framework/platform/security.md "mention"))                                               |
| :file\_cabinet: Startup.cs         | Module bootstrapper (see [bootstrapping.md](../../framework/platform/bootstrapping.md "mention"))                                    |
