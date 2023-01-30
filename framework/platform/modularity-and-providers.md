# Modularity & Providers

## Providers

A provider is a design pattern to integrate and swap a code extension more easily and flexibly. A good example are payment methods. If a developer wants to implement multiple payment methods of one payment company, he can create a single module (representing the payment company), which contains a provider for each payment method. Besides [IPaymentMethod](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Checkout/Payment/Service/IPaymentMethod.cs), there are other providers like [IExportProvider](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/DataExchange/Export/IExportProvider.cs), [IShippingRateComputationMethod](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Checkout/Shipping/Services/IShippingRateComputationMethod.cs), [IExchangeRateProvider](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Common/Services/IExchangeRateProvider.cs), [IExternalAuthenticationMethod](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Identity/Services/IExternalAuthenticationMethod.cs) etc.

Each of these provider interfaces is derived from the marker interface [IProvider](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Engine/Modularity/IProvider.cs) in order to be able to identify it uniformly as a provider. The [generic provider class](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Modularity/Provider.cs) encapsulates the respective interface and enriches it with general properties like [ProviderMetadata](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Modularity/ProviderMetadata.cs). As a result you get an abstract, handy and API friendly construct such as `Provider<IPaymentMethod>`.



<mark style="color:red;">**BEGIN: this is all duplicated docu!? See Compose > Modules.**</mark>&#x20;

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

| Folder name   | Contains                                                                                                                                                                                                                                                                                                                                                                                   |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Client        | HTTP or third party API clients.                                                                                                                                                                                                                                                                                                                                                           |
| Components    | View components.                                                                                                                                                                                                                                                                                                                                                                           |
| Configuration | Configuration files like settings inherited from ISetting.                                                                                                                                                                                                                                                                                                                                 |
| Controllers   | View supporting MVC controllers.                                                                                                                                                                                                                                                                                                                                                           |
| Domain        | Entities and `type` or `enum` definitions.                                                                                                                                                                                                                                                                                                                                                 |
| Extensions    | Classes with extension methods. 1.                                                                                                                                                                                                                                                                                                                                                         |
| Filters       | Any implementation of MVC filters like `TypeFilterAttribute`, `IAsyncActionFilter`, `IResultFilter` etc.                                                                                                                                                                                                                                                                                   |
| Hooks         | [Hook](hooks.md) implementations. 1.                                                                                                                                                                                                                                                                                                                                                       |
| Localization  | XML files with language localizations. At a minimum, a module should include English localizations _resources.en-us.xml_.                                                                                                                                                                                                                                                                  |
| Migrations    | Fluent Migrator database migrations. 1.                                                                                                                                                                                                                                                                                                                                                    |
| Models        | MVC view models and other models.                                                                                                                                                                                                                                                                                                                                                          |
| Providers     | Any provider implementation like `IExternalAuthenticationMethod`, `IPaymentMethod` etc.                                                                                                                                                                                                                                                                                                    |
| Services      | Module specific service classes.                                                                                                                                                                                                                                                                                                                                                           |
| Views         | MVC razor views.                                                                                                                                                                                                                                                                                                                                                                           |
| wwwroot       | <p>Any publicly accessible files (optional). Examples:</p><p><em>public.scss</em>: Sass Cascading Style Sheet (SCSS) definitions.<br><em>icon.png</em>: icon automatically displayed in modules list. 48 x 48 pixel.</p><p><em>favicon.png</em>: small icon, e.g. for order notes. 16 x 16 pixel.</p><p><em>branding.(png|gif|jpg)</em>: any associated brand image, e.g. PayPal logo.</p> |

It is recommended to place some files (optional) in the root directory of the module (besides those mandatory already mentioned).

| File name      | Description                                                                                               |
| -------------- | --------------------------------------------------------------------------------------------------------- |
| AdminMenu.cs   | `AdminMenuProvider` or `IMenuProvider` imlementations of menu item(s) injected to administration menu. 1. |
| Events.cs      | `IConsumer` of _all_ consumed events.                                                                     |
| Permissions.cs | `IPermissionProvider` to add custom permissions. 1.                                                       |
| Startup.cs     | `StarterBase` inherited module startup for middleware related things like IoC registrations. 1.           |

1. Usually these classes have an `internal` access modifier.

### Localization

The folder _Localization_ contains XML files with language localizations of a module. At a minimum, it is recommended to include English localizations _resources.en-us.xml_, except the module has no localizations at all. The file name has the format `resources.<language culture>.xml` in lower case.

{% code title="resources.en-us.xml example" %}
```xml
<Language Name="English" IsDefault="false" IsRightToLeft="false">
    <LocaleResource Name="Plugins.FriendlyName.MyCompany.AbcPay" AppendRootKey="false">
        <Value>Pay with abc</Value>
    </LocaleResource>
    <LocaleResource Name="Plugins.Description.MyCompany.AbcPay" AppendRootKey="false">
        <Value>Provides the payment methods of abc.</Value>
    </LocaleResource>
    <LocaleResource Name="PublicKeyId">
        <Value>Public API key</Value>
    </LocaleResource>
    <LocaleResource Name="PublicKeyId.Hint">
        <Value>Specifies the public key of the abc API.</Value>
    </LocaleResource>
    <!-- TODO more localizations... -->    
</Language>
```
{% endcode %}

HINT: the _ResourceRootKey_ from the _module.json_ file is automatically prefixed to the resources's name if the `AppendRootKey` attribute is missing in `LocaleResource` node.

The localization name for the modules's name and description must follow the following convention:

| Resource name\key                  | Description                                  |
| ---------------------------------- | -------------------------------------------- |
| Plugins.FriendlyName.\<SystemName> | Localized name of the module.                |
| Plugins.Description.\<SystemName>  | A short description of what the module does. |

<mark style="color:red;">**END: this is all duplicated docu!? See Compose > Modules.**</mark>&#x20;
