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
