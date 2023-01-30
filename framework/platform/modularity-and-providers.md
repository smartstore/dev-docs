# Modularity & Providers

## Modules

A module is an extension of Smartstore with one or more functions, such as pay with AmazonPay, authenticate with Facebook or export in Google Merchant Center format. The most important connection to the core is the `Module` class inherited from `ModuleBase`, which is called when installing and uninstalling the module.

{% code title="Simple module class" %}
```csharp
internal class Module : ModuleBase
{
    public override async Task InstallAsync(ModuleInstallationContext context)
    {
        await TrySaveSettingsAsync<MyModuleSettings>();
        await ImportLanguageResourcesAsync();

        await base.InstallAsync(context);
    }

    public override async Task UninstallAsync()
    {
        await DeleteSettingsAsync<MyModuleSettings>();
        await DeleteLanguageResourcesAsync();

        await base.UninstallAsync();
    }
}

public class MyModuleSettings : ISettings
{
    //...
}
```
{% endcode %}

### Metadata

A module must contain a _module.json_ file with JSON formatted metadata in his root directory.

{% code title="module.json example" %}
```json
{
  "$schema": "../module.schema.json",
  "Group": "Payment",
  "SystemName": "MyCompany.MyModule",
  "FriendlyName": "Pay with xyz",
  "Description": "",
  "Version": "5.0.2",
  "MinAppVersion": "5.0.2",
  "Author": "My company",
  "Order": 1,
  "ResourceRootKey": "Modules.MyCompany.MyModule"
}
```
{% endcode %}

| Name                | Type      | Description                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ------------------- | --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| $schema             | string    | Path to schema for module JSON file format, usually _../module.schema.json_.                                                                                                                                                                                                                                                                                                                                                                 |
| **FriendlyName \*** | string    | English (unlocalized) friendly name. Displayed in modules list in administration backend.                                                                                                                                                                                                                                                                                                                                                    |
| Description         | string    | English (unlocalized) description. Displayed in modules list in administration backend.                                                                                                                                                                                                                                                                                                                                                      |
| **SystemName \***   | string    | Module system name (usually the assembly name without extension). Recommended format `<unique name of company\author>.<module name>`. Please use your own name and avoid names of others like _Smartstore_.                                                                                                                                                                                                                                  |
| Theme               | string    | Theme name (if the module is a theme companion). On build a symbolic link will be created in _/Themes/\[Theme]_ to _/Modules/\[Module]_.                                                                                                                                                                                                                                                                                                     |
| Author              | string    | Name of the author.                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ProjectUrl          | string    | Link to project's or author's homepage.                                                                                                                                                                                                                                                                                                                                                                                                      |
| Tags                | string\[] | Comma-separated tags.                                                                                                                                                                                                                                                                                                                                                                                                                        |
| **Version \***      | string    | Current version, e.g. 5.0.2. Usually corresponds to current Smartstore version.                                                                                                                                                                                                                                                                                                                                                              |
| MinAppVersion       | string    | The minimum compatible application version.                                                                                                                                                                                                                                                                                                                                                                                                  |
| ResourceRootKey     | string    | The root key of module's language resources. Typically the key has the format `Modules.<SystemName>`. For backward compatibility you can also find other formats like _Plugins.Payments.xyz_ in older modules.                                                                                                                                                                                                                               |
| AssemblyName        | string    | Optional assembly name. By default `<SystemName>.dll`. is assumed to be the assembly name.                                                                                                                                                                                                                                                                                                                                                   |
| Order               | int       | Display order in administration backend's modules list.                                                                                                                                                                                                                                                                                                                                                                                      |
| PrivateReferences   | string\[] | Optional array of private dependency package names that the module references. By default, referenced packages are not copied to the module's output directory because it is assumed that the application core references them already. Any module private package should be listed here (including the complete dependency chain). Example: AmazonPay is dependent on its API SDK for .NET. So _Amazon.Pay.API.SDK_ must be specified here. |
| DependsOn           | string\[] | Optional array of module system names that this module depends on.                                                                                                                                                                                                                                                                                                                                                                           |
| Group               | string    | Conceptual group name. Used to visually categorize modules in module list in administration backend. For known groups see `ModuleDescriptor.KnownGroups`.                                                                                                                                                                                                                                                                                    |

### Folders and files

The folder structure of the module should be similar to that of other core projects. The following table shows the most common folders of a module.

| Folder name   | Contains                                                                                                                                                                                                                                                                                                                                                                         |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Client        | HTTP or third party API clients.                                                                                                                                                                                                                                                                                                                                                 |
| Components    | View components.                                                                                                                                                                                                                                                                                                                                                                 |
| Configuration | Configuration files like settings inherited from ISetting.                                                                                                                                                                                                                                                                                                                       |
| Controllers   | View supporting MVC controllers.                                                                                                                                                                                                                                                                                                                                                 |
| Domain        | Entities and `type` or `enum` definitions.                                                                                                                                                                                                                                                                                                                                       |
| Extensions    | Classes with extension methods. 1.                                                                                                                                                                                                                                                                                                                                               |
| Filters       | Any implementation of MVC filters like `TypeFilterAttribute`, `IAsyncActionFilter`, `IResultFilter` etc.                                                                                                                                                                                                                                                                         |
| Hooks         | [Hook](hooks.md) implementations. 1.                                                                                                                                                                                                                                                                                                                                             |
| Localization  | XML files with language localizations. At a minimum, a module should include English resources _resources.en-us.xml_.                                                                                                                                                                                                                                                            |
| Migrations    | Fluent Migrator database migrations. 1.                                                                                                                                                                                                                                                                                                                                          |
| Models        | MVC view models and other models.                                                                                                                                                                                                                                                                                                                                                |
| Providers     | Any provider implementation like `IExternalAuthenticationMethod`, `IPaymentMethod` etc.                                                                                                                                                                                                                                                                                          |
| Services      | Module specific service classes.                                                                                                                                                                                                                                                                                                                                                 |
| Views         | MVC razor views.                                                                                                                                                                                                                                                                                                                                                                 |
| wwwroot       | <p>Any publicly accessible files (optional). Examples:</p><p><em>public.scss</em>: Sass Cascading Style Sheet (SCSS) definitions.<br><em>icon.png</em>: icon automatically displayed in modules list. 48 x 48 pixel.</p><p><em>favicon.png</em>: small icon, e.g. for order notes. 16 x 16 pixel.</p><p><em>branding.png</em>: any associated brand image, e.g. PayPal logo.</p> |

Some optional files are placed in the root directory of the module (besides those already mentioned).

| File name    |                                                                                             |
| ------------ | ------------------------------------------------------------------------------------------- |
| AdminMenu.cs | `AdminMenuProvider` or `IMenuProvider` imlementations of menu item(s) injected to menus. 1. |
|              |                                                                                             |
|              |                                                                                             |

1. Usually classes with `internal` access modifier.
