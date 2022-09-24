---
description: >-
  Where are modules located and how are they deployed?, Starters, IModule,
  module.json
---

# Getting started with modules

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

* Lorem ipsum
* TODO: move comments to table

{% code title="module.json" %}
```json
{
  "$schema": "../module.schema.json",
  // Author name.
  "Author": "MyOrg",
  // Conceptual group name. Used to visually categorize module in module listings.
  // Possible values: 
  // Admin, Marketing, Payment, Shipping, Tax, Analytics, CMS
  // Media, SEO, Data, Globalization, Api, Mobile, Social, Security,
  // Developer, Sales, Design, Performance, B2B, Storefront, Law
  "Group": "Payment",
  // Required. Module system name (usually the assembly name without extension).
  "SystemName": "MyOrg.MyGreatModule",
  // Optional assembly name. By default '{SystemName}.dll' is assumed to be the assembly name
  "AssemblyName": "MyOrg.MyGreatModule.dll",
  // Required. English friendly name.
  "FriendlyName": "A great module for Smartstore",
  // English description.
  "Description": "Lorem ipsum",
  // Required. The current version of module, e.g. '5.0'.
  "Version": "5.0",
  // The minimum compatible application version, e.g. '5.0.0'.
  "MinAppVersion": "5.0",
  // The display order in the module manager group.
  "Order": 1,
  // Root key for language resources.
  "ResourceRootKey": "Plugins.Payments.AmazonPay",
  // Link to project or author homepage.
  "ProjectUrl": "http://myorg.com",
  // Comma-separated tags.
  "Tags": "tag1, tag2",
  // Optional array of private dependency package names that this module references. 
  // By default, referenced packages are not copied to the module's output directory 
  // because it is assumed that the application core references them already. 
  // Any module private package should be listed here
  // (including the complete dependency chain).
  "PrivateReferences": [ "Amazon.Pay.API.SDK" ],
  // Optional array of module system names that this module depends on.
  "DependsOn": [ "Smartstore.Tax" ]
}
```
{% endcode %}

#### Schema reference

| Name                  | Type    | Description                                                                                                                                                                                                                                                                                                                       |
| --------------------- | ------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **AssemblyName**      | string  | Optional assembly name. By default '{SystemName}.dll' is assumed to be the assembly name.                                                                                                                                                                                                                                         |
| **Author**            | string  | Author name.                                                                                                                                                                                                                                                                                                                      |
| **FriendlyName \***   | string  | English (unlocalized) friendly name.                                                                                                                                                                                                                                                                                              |
| **Group**             | enum    | Conceptual group name. Used to visually categorize module in module listings.  Possible values: _Admin, Marketing, Payment, Shipping, Tax, Analytics, CMS, Media, SEO, Data, Globalization, Api, Mobile, Social, Security, Developer, Sales, Design, Performance, B2B, Storefront, Law_.                                          |
| **DependsOn**         | array   | Optional array of module system names that a module depends on.                                                                                                                                                                                                                                                                   |
| **Description**       | string  | English (unlocalized) description.                                                                                                                                                                                                                                                                                                |
| **MinAppVersion**     | string  | The minimum compatible application version, e.g. '5.0.0'.                                                                                                                                                                                                                                                                         |
| **Order**             | integer | The display order in the module manager group.                                                                                                                                                                                                                                                                                    |
| **PrivateReferences** | array   | Optional array of private dependency package names that a module references. By default, referenced packages are not copied to the module's output directory because it is assumed that the application core references them already. Any module private package should be listed here (including the complete dependency chain). |
| **ProjectUrl**        | string  | Link to project's or author's homepage.                                                                                                                                                                                                                                                                                           |
| **ResourceRootKey**   | string  | Root key for language resources.                                                                                                                                                                                                                                                                                                  |
| **SystemName \***     | string  | Module system name (usually the assembly name without extension).                                                                                                                                                                                                                                                                 |
| **Tags**              | string  | Comma-separated tags.                                                                                                                                                                                                                                                                                                             |
| **Version \***        | string  | The current version of module, e.g. '5.1.0'.                                                                                                                                                                                                                                                                                      |

### Module entry class

* Lorem ipsum
