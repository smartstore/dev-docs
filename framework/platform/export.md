# Export

## Overview

* The data exporter provides the data in segments that are written in a specific format into a stream by an export provider.
* Export profiles are entities that are binding the export to an export provider and storing all settings and configurations of the export.
* When an export is executed, a task associated with the export profile is started, which performs the actual export via data exporter and export provider.
* Export deployments are entities that can optionally be assigned to an export profile to specify how to proceed with the export data (e.g. files), for example to send them to an e-mail address.

## Export provider

* An export provider specifies the data format (e.g. CSV or XML) and if it is a file based or on-the-fly in-memory export. It always writes the data into stream, so it never comes in contact with files at any time.
* The provider implements [IExportProvider](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/DataExchange/Export/IExportProvider.cs) or it inherits from `ExportProviderBase` and declares `SystemName`, `FriendlyName`, `Order` and `ExportFeatures` using attributes.
* If additional files are required independently of the actual export files, they can be requested via `ExportExecuteContext.ExtraDataUnits`.
* Depending on the configuration, the provider is called several times by the data exporter during an export. Typically once per export file. Use `ExportExecuteContext.CustomProperties` for any custom data required across the entire export.

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

### Provider specific configuration

Override `ExportProviderBase.ConfigurationInfo` and provide a `ComponentWidget` and a model type:

```csharp
protected override async Task ExportAsync(ExportExecuteContext context, CancellationToken cancelToken)
{
	var config = (context.ConfigurationData as MyProfileConfigurationModel) ?? new MyProfileConfigurationModel();
	// ...
}
```

### Data access

Export data is provided in segments with dynamic objects which contains all properties of the entity plus extra data generally prefixed with an underscore, e.g. `dynObject._BasePriceInfo`.

Dynamic objects has projection and configuration of the export profile applied. For example if a certain language is selected in projection tab, the dynamic object contains the localized property values (e.g. a localized product name).

The actual entity is accessibly via `dynObject.Entity`.

## Export profile

* Combines an export provider, an export task, deployments (optional), configuration and settings to a profile.&#x20;
* Two types: built-in system profiles and user profiles that have been added subsequently by the user. System profiles are used, for example, when exporting orders via the order list in the backend.
* Partition:
* Filter:
* &#x20;Projection:
* Configuration:

## Deployment

* ....
