# 🍪 Getting started with modules

## Overview

Smartstore modules are designed to extend Smartstore's functionality in any way you can imagine. There are no limits to what features they can add.

Here are a few examples of what modules can do:

* alter the way the app operates
* change workflows
* modify / extend UI
* overwrite services

Modules are sets of extensions that compile into a single assembly in order to be re-used in other Smartstore shops. Eventhough it may use Smartstore APIs, they are no necessity. The only to requirements for a module project are:

* `module.json`: A manifest file describing the metadata of a module.
* `Module.cs`: A class that implements IModule and contains (un-) installation routines.

Some special, mostly commerce related, features are encapsulated as [_providers_ ](../../framework/platform/modularity-and-providers.md)in the Smartstore ecosystem. A module can expose as many of these providers needed.

Represented by their interfaces, provider types are:

* `IPaymentMethod`: Payment providers (PayPal, AmazonPay, offline payment etc.)
* `ITaxProvider`: Tax calculation
* `IShippingRateComputationMethod`: Shipping fee calculation
* `IExportProvider`: Data export (Shops, products, orders etc.)
* `IMediaStorageProvider`: Storage for media file blobs
* `IOutputCacheProvider`: Storage for ouput cache items
* `IWidget`: Content rendering in UI
* `IExternalAuthenticationMethod`: External authenticators (Google, Facebook etc.)
* `IExchangeRateProvider`: Live curreny rates

## Module structure

A module is a regular _Class Library_ project in the solution. It should be placed in the **/src/Smartstore.Modules/** directory in the root of your solution.

{% hint style="warning" %}
Do not confuse this with the **/src/Smartstore.Web/Modules/** directory, which is the build target for module. From here the application loads module assemblies into the app-domain dyncamically.
{% endhint %}

If your project directory is located elsewhere, you should create a _symlink_ that links to the actual location.

{% hint style="info" %}
It is good practice to use the **-sym** suffix in symlink sources, because they are git-ignored.
{% endhint %}

For module projects we recommend the naming convention **{Organization}.{ModuleName}**, but you can choose any name you wish. It should also be the _root namespace_ and the _module system name_.

If your module is called **MyOrg.MyGreatModule**, the _.csproj_ project file should look like this:

```xml
<Project Sdk="Microsoft.NET.Sdk.Razor">
    <PropertyGroup>
	<Product>A great module for Smartstore</Product>
	<OutputPath>..\..\Smartstore.Web\Modules\MyOrg.MyGreatModule</OutputPath>
	<OutDir>$(OutputPath)</OutDir>
    </PropertyGroup>
</Project>
```

Each time the solution get's built, your module will be compiled and copied to the `OutputPath` directory specified here.

### Smartstore.Module.props

The file _Smartstore.Build/Smartstore.Module.props_ defines shared properties for modules. It is auto-included into every project located in _Smartstore.Modules/_.

Among other things it specifies files and directories:

* to be copied to the `OutputPath`, if they exist.
  * _module.json_
  * _wwwroot/_
  * _Localization/_
  * _Views/_
* **not** to be copied to the `OutputPath`.

{% hint style="warning" %}
This is important, because the build process would copy the whole dependency graph to the output, which produces too much noise and file redundancy.
{% endhint %}

### Project & Package references

All projects located in the _Smartstore.Modules_ directory reference `Smartstore`, `Smartstore.Core` and `Smartstore.Web.Common` projects by default.

{% hint style="info" %}
You can also reference `Smartstore.Web` to access model types declared there. But add the following lines to the project file, to prevent your project copying dependant files to your `OutputPath`:

```xml
<ItemGroup>
    <ProjectReference Include="..\..\Smartstore.Web\Smartstore.Web.csproj">
        <Private>False</Private>
        <CopyLocal>False</CopyLocal>
        <CopyLocalSatelliteAssemblies>False</CopyLocalSatelliteAssemblies>
    </ProjectReference>
</ItemGroup>
```
{% endhint %}

You can reference any NuGet package you wish, but special consideration is required for private packages. These packages are **not** referenced by the app core.

{% hint style="info" %}
Copying dependencies is completely suppressed. To copy these files anyway you must declare them in `module.json`.
{% endhint %}

### Manifest: module.json

This file describes your module to the system and is used by the _Plugin Manager_ in it's administration screen.

Here is an example of a working `module.json` file.

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

{% hint style="info" %}
The properties `SystemName`, `FriendlyName` and `Version` are required.
{% endhint %}

The following table explains the Schema.

| Name                  | Type    | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| --------------------- | ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **AssemblyName**      | string  | The assembly name. **Default**: '{SystemName}.dll'                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| **Author**            | string  | The author's name.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| **FriendlyName \***   | string  | A readable, easy to understand, english name.                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| **Group**             | enum    | <p>A conceptual group name. Used to visually categorize modules in listings.</p><p><strong>Possible values</strong>: <em>Admin</em>, <em>Marketing</em>, <em>Payment</em>, <em>Shipping</em>, <em>Tax</em>, <em>Analytics</em>, <em>CMS</em>, <em>Media</em>, <em>SEO</em>, <em>Data</em>, <em>Globalization</em>, <em>Api</em>, <em>Mobile</em>, <em>Social</em>, <em>Security</em>, <em>Developer</em>, <em>Sales</em>, <em>Design</em>, <em>Performance</em>, <em>B2B</em>, <em>Storefront</em>, <em>Law</em>.</p> |
| **DependsOn**         | array   | Array of module system names the module depends on.                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| **Description**       | string  | A short english description of the module.                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| **MinAppVersion**     | string  | <p>The minimum compatible application version, e.g. '5.0.2'.</p><p>The module will be unavailable, if the current app version is lower than this value.</p>                                                                                                                                                                                                                                                                                                                                                           |
| **Order**             | integer | The display order in the module manager group.                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| **PrivateReferences** | array   | <p>Optional array of private dependency package names that a module references.</p><p>By default referenced packages are not copied to the <code>OutputPath</code>. It is assumed, that the application core already references them. Any private module package should be listed here, including the complete dependency chain.</p>                                                                                                                                                                                  |
| **ProjectUrl**        | string  | Link to the project's or author's homepage.                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| **ResourceRootKey**   | string  | Root key for language resources (see [Localizing modules](localizing-modules.md)).                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| **SystemName \***     | string  | Module system name. Usually the assembly name without the extension.                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| **Tags**              | string  | Comma-separated tags                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| **Version \***        | string  | The current version of the module e.g. '5.1.0'                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |

### Module entry class

Every module needs an entry class containing the bare minimum of the un- and install methods. To be recognized as such the class must implement the `IModule` interface.

{% hint style="info" %}
It is also recommended to derive from the abstract [ModuleBase](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Modularity/ModuleBase.cs) instead of implementing `IModule`, because it contains some common implementations.
{% endhint %}

The installation method `IModule.InstallAsync()` is called every time the module is installed. Respectively the same goes for the uninstall method `IModule.UninstallAsync()`.

{% hint style="info" %}
It is good practice **not** to delete any custom module data from the database while uninstalling, in case the user wants to re-install the module later.
{% endhint %}

By convention the file named `Module.cs`, is `internal` and is placed in the project's root directory.

{% hint style="info" %}
If your module contains exactly one feature provider, it is recommended to let the entry class implement the provider interface directly.
{% endhint %}

The `IConfigurable` interface is used to expose the route to a module configuration page linked to the _Plugin Manager_ UI.

The following code shows an example of a working `Module.cs` file.

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

### Files & Folders Best Practices

There are some conventions on how to organize the files and directories within a project. Though there is no obligation to comply, it makes things predictable and easier to maintain.

The following is an exhaustive list of files & folders.

| Entry                                                                                  | Description                                                                                                             |
| -------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| :file\_folder: **App\_Data**                                                           | App specific (cargo) data like templates, sample files etc. that needs to be published.                                 |
| :file\_folder: **Blocks**                                                              | Page Builder Block implementations (see [Page Builder and Blocks](../../framework/content/page-builder-and-blocks.md)). |
| :file\_folder: **Bootstrapping**                                                       | Bootstrapping code like _Autofac_ modules, DI extensions etc.                                                           |
| :file\_folder: **Client**                                                              | RESTful API clients                                                                                                     |
| :file\_folder: **Components**                                                          | MVC View Components (see [Controllers and ViewComponents](controllers-and-viewcomponents.md))                           |
| :file\_folder: **Configuration**                                                       | Settings class implementations (see [Configuration](../../framework/platform/configuration.md))                         |
| :file\_folder: **Controllers**                                                         | MVC Controllers (see [Controllers and ViewComponents](controllers-and-viewcomponents.md))                               |
| :file\_folder: **Domain**                                                              | Domain entities (see [Domain](../../getting-started/domain.md))                                                         |
| :file\_folder: **Extensions**                                                          | Static extension method classes                                                                                         |
| :file\_folder: **Filters**                                                             | MVC Filters (see [Filters](filters.md))                                                                                 |
| :file\_folder: **Hooks**                                                               | Hook implementations (see [Hooks](../../framework/platform/hooks.md))                                                   |
| :file\_folder: **Localization**                                                        | Localization files (see [Localizing modules](localizing-modules.md))                                                    |
| :file\_folder: **Media**                                                               | Media system related classes                                                                                            |
| :file\_folder: **Migrations**                                                          | Fluent data migrations (see [Database Migrations](../../framework/platform/database-migrations.md))                     |
| :file\_folder: **Models**                                                              | View model classes (see [Data Modelling](../../framework/platform/data-modelling/))                                     |
| :file\_folder: **Providers**                                                           | Provider implementations                                                                                                |
| :file\_folder: **Security**                                                            | Security related classes                                                                                                |
| :file\_folder: **Services**                                                            | Service classes                                                                                                         |
| :file\_folder: **Utils**                                                               | Utilities                                                                                                               |
| :file\_folder: **Tasks**                                                               | Task scheduler jobs (see [Scheduling](../../framework/platform/scheduling.md))                                          |
| :file\_folder: **TagHelpers**                                                          | Tag Helpers                                                                                                             |
| :file\_folder: **Views**                                                               | Razor view/template files                                                                                               |
| :file\_folder: **wwwroot**                                                             | Static files (including Sass)                                                                                           |
| :file\_cabinet: AdminMenu.cs                                                           | Admin menu hook (see [Menus](../../framework/content/menus.md))                                                         |
| :file\_cabinet: CacheableRoutes.cs                                                     | Route registrar for output cache (see [Output Cache](../../framework/platform/output-cache.md))                         |
| :file\_cabinet: Events.cs                                                              | Event handler methods (see [Events](../../framework/platform/events.md))                                                |
| :file\_cabinet: [Module.cs](getting-started-with-modules.md#module-entry-class) \*     | Required. Module entry class (implements `IModule`).                                                                    |
| :file\_cabinet: [module.json](getting-started-with-modules.md#manifest-module.json) \* | Required. Module metadata manifest.                                                                                     |
| :file\_cabinet: Permissions.cs                                                         | Module permissions (see [Security](../../framework/platform/security.md))                                               |
| :file\_cabinet: Startup.cs                                                             | Module bootstrapper (see [Bootstrapping](../../framework/platform/bootstrapping.md))                                    |
