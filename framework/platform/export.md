# Export

## Overview

* The data exporter provides the data in segments that are written in a specific format into a stream by an export provider.
* Export profiles are entities that are binding the export to an export provider and combining and storing all aspects of an export like settings and configurations making it configurable by the user.
* When an export is executed, a task associated with the export profile is started, which performs the actual export via data exporter and export provider. The task can be triggered manually or scheduled.
* Export deployments are entities that can optionally be assigned to an export profile to specify how to proceed with the export data (e.g. files), for example to send them to an e-mail address.

## Export provider

The provider implements [IExportProvider](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/DataExchange/Export/IExportProvider.cs) or it inherits from [ExportProviderBase](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/DataExchange/Export/ExportProviderBase.cs). This documentation refers to a provider that inherits from `ExportProviderBase`.

The provider declares `SystemName`, `FriendlyName` and `Order` using attributes. It also specifies the data format (e.g. CSV or XML) and if it is a file based or on-the-fly in-memory export. Set `FileExtension` to _null_ if you do not want to export into files.&#x20;

`ExportAsync` is the main method to export data. The provider always writes the data into stream. It never comes in contact with files at any time. File related aspects are all handled internally by the data exporter.

If additional files are required independently of the actual export files, they can be requested via `ExportExecuteContext.ExtraDataUnits`. The data exporter calls `OnExecutedAsync` for each extra data unit added by a provider.\
HINT: `OnExecutedAsync` is always called after the actual export, even if no extra data unit was requested. You can use it for all kinds of finalization work.

Depending on the configuration, the provider's `ExportAsync` is called several times by the data exporter during an export. Typically once per export file, depending on the partition settings of the profile. Use `ExportExecuteContext.CustomProperties` for any custom data required across the entire export.

`ExportFeatures` flags are used to specify certain data processing and projection items supported by a provider. `CanProjectAttributeCombinations` for example indicates that the provider is able to export attribute combinations as products.

{% code title="A simple export provider" %}
```csharp
[SystemName("Exports.MyCompanyProductCsv")]
[FriendlyName("MyCompany product CSV Export")]
public class MyCompanyProductExportProvider : ExportProviderBase
{
    public override ExportEntityType EntityType => ExportEntityType.Product;
    public override string FileExtension => "CSV";

    protected override async Task ExportAsync(ExportExecuteContext context, CancellationToken cancelToken)
    {
        using var writer = new CsvWriter(new StreamWriter(context.DataStream, Encoding.UTF8, 1024, true), CsvConfiguration.ExcelFriendlyConfiguration);

        writer.WriteField("Id");
        writer.WriteField("Name");
        // Write more columns...
        writer.NextRow();

        while (context.Abort == DataExchangeAbortion.None && await context.DataSegmenter.ReadNextSegmentAsync())
        {
            var segment = await context.DataSegmenter.GetCurrentSegmentAsync();

            foreach (dynamic product in segment)
            {
                if (context.Abort != DataExchangeAbortion.None)
                    break;

                Product entity = product.Entity;

                writer.WriteField(entity.Id.ToString());
                // Get and export localized product name from dynamic object.
                writer.WriteField((string)product.Name);
                // Write more fields...

                writer.NextRow();
                ++context.RecordsSucceeded;
            }
        }
    }
}
```
{% endcode %}

Your provider is displayed in the provider select box when adding a new export profile.

It is recommended to give your provider a friendly localized name and description using string resources and the localization XML files of your module:

| Property to localize -> String resource name                                    | Example                                          |
| ------------------------------------------------------------------------------- | ------------------------------------------------ |
| <p>Provider name -><br>Plugins.FriendlyName.&#x3C;ProviderSystemName></p>       | Plugins.FriendlyName.Exports.MyCompanyProductCsv |
| <p>Provider description -><br>Plugins.Description.&#x3C;ProviderSystemName></p> | Plugins.Description.Exports.MyCompanyProductCsv  |

The description is displayed when adding a new export profile and selecting your provider.

### Provider specific configuration

Override `ExportProviderBase.ConfigurationInfo` and provide a `ComponentWidget` and a model type:

```csharp
public override ExportConfigurationInfo ConfigurationInfo => new()
{
    ConfigurationWidget = new ComponentWidget(typeof(MyConfigurationViewComponent)),
    ModelType = typeof(MyProviderConfigurationModel)
};

protected override async Task ExportAsync(ExportExecuteContext context, CancellationToken cancelToken)
{
    var config = (context.ConfigurationData as MyProviderConfigurationModel) ?? new MyProviderConfigurationModel();
    // ...
}
```

HINT: your configuration model must be decorated with `Serializable` and `CustomModelPart` attributes so that it can be successfully saved with the export profile.

```csharp
[Serializable, CustomModelPart]
[LocalizedDisplay("Plugins.MyCompany.MyModule.")]
public class MyProviderConfigurationModel
{
    [LocalizedDisplay("*MyConfigProperty")]
    public string MyConfigProperty { get; set; }
    //...
}
```

Your configuration is displayed in the configuration tab on the profile edit page.&#x20;

### Data access

Export data is provided in segments with dynamic objects which contains all properties of the entity plus extra data generally prefixed with an underscore, e.g. `dynObject._BasePriceInfo`.

Dynamic objects has projection and configuration of the export profile applied. For example if a certain language is selected in projection tab, the dynamic object contains the localized property values (e.g. a localized product name).

The actual entity is accessibly via `dynObject.Entity`.

### Export related data

Export profile has option `ExportRelatedData`. If activated and if provider supports `ExportFeatures.UsesRelatedDataUnits` then data exporter adds additional data units to `ExportExecuteContext.ExtraDataUnits`. The provider can use them to export related data into a separate file, for example to additionally export tier prices through a product export provider. At the moment only tier prices, variant attribute values and variant attribute combinations are supported when exporting products, see `RelatedEntityType`. This mechanism is intended mainly for updating prices via flat data formats such as CSV, which can be edited by end users.

TIP: related data files are automatically imported together with the main data file(s) using file naming convention. If the name of the related data file ends with a `RelatedEntityType` value (e.g. `TierPrice` or `ProductVariantAttributeCombination`) then the product importer uses its data to update tier prices, variant attribute values or variant attribute combinations.

## Export profile

* Combines all aspects of an export to make it configurable by the user: provider, task, partition, filters, projections, configuration and deployments.&#x20;
* An export provider can be assigned to several profiles with different configurations and settings.
* Two types: built-in system profiles and user profiles that have been added subsequently by the user. System profiles are used, for example, when exporting orders via the order grid in the backend.



## Deployment

* ....
