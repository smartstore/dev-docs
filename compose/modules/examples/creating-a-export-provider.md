# ✔ Creating an Export provider

Using export providers you can export your shop data into many different formats. Smartstore primarily uses CSV and XML. In this tutorial, you will write an export provider for the product catalog.

{% hint style="info" %}
To learn more about export, refer to [Export](../../../framework/platform/export.md).
{% endhint %}

## Create a configuration

If you want to make your export provider customizable, you will need a configuration. This step is optional. The configuration is made up of three things:

1. The profile configuration model `ProfileConfigurationModel` that describes the configurable data.
2. The view component `HelloWorldConfigurationViewComponent` that converts saved data into usable formats for your view.
3. The view that is displayed to the user. This acts like a widget view and must abide to the same directory structure.

In this configuration, you will be able to limit the number of rows that are exported.

### The Model

Create a class `ProfileConfigurationModel.cs` and add it to the _Models_ folder.

```csharp
namespace MyOrg.HelloWorld.Models
{
    [Serializable, CustomModelPart]
    [LocalizedDisplay("Plugins.MyOrg.HelloWorld.")]
    public class ProfileConfigurationModel
    {
        [LocalizedDisplay("*NumberOfExportedRows")]
        public string NumberOfExportedRows { get; set; } = 10;
    }
}
```

Add the resources to your localization file.

```xml
<LocaleResource Name="NumberOfExportedRows">
    <Value>Number of rows</Value>
</LocaleResource>
<LocaleResource Name="NumberOfExportedRows.Hint">
    <Value>Number of rows to be exported.</Value>
</LocaleResource>
```

### The ViewComponent

Create a class `HelloWorldConfigurationViewComponent.cs` and add it to the _Components_ folder.

```csharp
namespace MyOrg.HelloWorld.Components
{
    public class HelloWorldConfigurationViewComponent : SmartViewComponent
    {
        public IViewComponentResult Invoke(object data)
        {
            var model = data as ProfileConfigurationModel;
            return View(model);
        }
    }
}
```

### The View

Create a razor view `Default.cshtml` and add it to _Views / Shared / Components / HelloWorldConfiguration_.

```cshtml
@model ProfileConfigurationModel

@{
    Layout = null;
}

<div class="adminContent">
    <div class="adminRow">
        <div class="adminTitle">
            <smart-label asp-for="NumberOfExportedRows" />
        </div>
        <div class="adminData">
            <input asp-for="NumberOfExportedRows" />
            <span asp-validation-for="NumberOfExportedRows"></span>
        </div>
    </div>
</div>
```

## Add export providers

### The CSV export provider

Create a class `HelloWorldCsvExportProvider.cs` and add it to the new folder _Providers_.

```csharp
namespace MyOrg.HelloWorld.Providers
{
    public class HelloWorldCsvExportProvider : ExportProviderBase
    {
        protected override Task ExportAsync(ExportExecuteContext context, CancellationToken cancelToken)
        {
        }
    }
}
```

This class implements the `ExportProviderBase` interface, which requires you to override the `ExportAsync` method.

#### Add Attributes

Smartstore uses the following attributes to integrate the providers correctly: `SystemName`, `FriendlyName` and `ExportFeatures` (optional).

Add these attributes to your class definition.

```csharp
[SystemName("MyOrg.HelloWorld.ProductCsv")]
[FriendlyName("Hello world CSV product feed")]
[ExportFeatures(Features =
    ExportFeatures.CreatesInitialPublicDeployment |
    ExportFeatures.OffersBrandFallback)]
```

{% hint style="info" %}
`SystemName` is the system name of the provider, not the system name of the module.
{% endhint %}

| Export feature                  | Description                                                                                                                                                                              |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| CreatesInitialPublicDeployment  | Automatically create a file based public deployment when an export profile is created.                                                                                                   |
| CanOmitGroupedProducts          | Offer option to include/exclude grouped products.                                                                                                                                        |
| CanProjectAttributeCombinations | Offer option to export attribute combinations as products.                                                                                                                               |
| CanProjectDescription           | Offer further options to manipulate the product description.                                                                                                                             |
| OffersBrandFallback             | Offer option to enter a brand fallback.                                                                                                                                                  |
| CanIncludeMainPicture           | Offer option to set a picture size and to get the URL of the main image.                                                                                                                 |
| UsesSkuAsMpnFallback            | Use SKU as manufacturer part number if MPN is empty.                                                                                                                                     |
| OffersShippingTimeFallback      | Offer option to enter a shipping time fallback.                                                                                                                                          |
| OffersShippingCostsFallback     | Offer option to enter a shipping costs fallback and a free shipping threshold.                                                                                                           |
| CanOmitCompletionMail           | Automatically send a completion email.                                                                                                                                                   |
| UsesAttributeCombination        | Provide additional data of attribute combinations.                                                                                                                                       |
| UsesAttributeCombinationParent  | <p>Export attribute combinations as products including parent product.</p><p>This is only effective in combination with the <em>CanProjectAttributeCombinations</em> export feature.</p> |
| UsesRelatedDataUnits            | Provide extra data units for related data.                                                                                                                                               |

To tell the provider that you want to export a CSV file, the `FileExtension` property has to be overriden. The `Localizer` is used for localizing messages. `CsvConfiguration` specifies details of the CSV format.

```csharp
public override string FileExtension => "CSV";

public Localizer T { get; set; } = NullLocalizer.Instance;

private CsvConfiguration _csvConfiguration;

private CsvConfiguration CsvConfiguration
{
    get
    {
        _csvConfiguration ??= new CsvConfiguration
        {
            Delimiter = ';',
            SupportsMultiline = false
        };

        return _csvConfiguration;
    }
}
```

#### Add configuration

Next you need to tell the provider how it should be configured (_ViewComponent_ and _Model_). For this the `ConfigurationInfo` method is used.

```csharp
public override async ExportConfigurationInfo ConfigurationInfo => new()
{
    ConfigurationWidget = new ComponentWidget<HelloWorldConfigurationViewComponent>(),
    ModelType = typeof(ProfileConfigurationModel)
};
```

#### Export data

Now you can start exporting your data. Start off by fetching the profile configuration data.

```csharp
var config = (context.ConfigurationData as ProfileConfigurationModel) ?? new ProfileConfigurationModel();
```

Get a `CsvWriter` and write the first row with column names.

```csharp
using var writer = new CsvWriter(new StreamWriter(context.DataStream, Encoding.UTF8, 1024, true));

writer.WriteFields(new string[]
{
    "ProductName",
    "SKU",
    "Price",
    "Savings",
    "Description"
});
writer.NextRow();
```

Now iterate over the data segments.

```csharp
while (context.Abort == DataExchangeAbortion.None && await context.DataSegmenter.ReadNextSegmentAsync())
{
    var segment = await context.DataSegmenter.GetCurrentSegmentAsync();
}
```

Inside the while loop we fetch the product and it's entity.

```csharp
foreach (dynamic product in segment)
{
    if (context.Abort != DataExchangeAbortion.None)
    {
        break;
    }

    Product entity = product.Entity;
}
```

{% hint style="info" %}
The difference between `entity` and `product` is the following:

* `entity`: Represents the original entity read from the database.
* `product`: A dynamic object that encapsulates and enriches the entity with some computed data.
{% endhint %}

Add a try-catch block for error handling.

```csharp
try
{
    // Export Product data
}
catch (OutOfMemoryException ex)
{
    context.RecordOutOfMemoryException(ex, entity.Id, T);
    context.Abort = DataExchangeAbortion.Hard;
    throw;
}
catch (Exception ex)
{
    context.RecordException(ex, entity.Id);
}
```

Now you calculate the savings inside the try-block.

```csharp
var calculatedPrice = (CalculatedPrice)product._Price;
var saving = calculatedPrice.Saving;
```

Next we write the fields in order of our columns. After that we increment the row count.

```csharp
writer.WriteFields(new string[]
{
    product.Name,
    product.Sku,
    ((decimal)product.Price).FormatInvariant(),
    saving.HasSaving ? saving.SavingPrice.Amount.FormatInvariant() : string.Empty,
    ((string)product.FullDescription).Truncate(5000)
});

writer.NextRow();
context.RecordsSucceeded++;
```

And finally you want to limit your row exports to `NumberOfExportedRows` from the profile configuration data.

```csharp
if (context.RecordsSucceeded >= config.NumberOfExportedRows)
{
    context.Abort = DataExchangeAbortion.Soft;
}
```

Your code may look something like this:

{% code title="HelloWorldCsvExportProvider.cs" %}
```csharp
namespace MyOrg.HelloWorld.Providers
{
    [SystemName("MyOrg.HelloWorld.ProductCsv")]
    [FriendlyName("Hello world CSV product feed")]
    [ExportFeatures(Features =
        ExportFeatures.CreatesInitialPublicDeployment |
        ExportFeatures.OffersBrandFallback)]
    public class HelloWorldCsvExportProvider : ExportProviderBase
    {
        public override string FileExtension => "CSV";
        
        public Localizer T { get; set; } = NullLocalizer.Instance;

        private CsvConfiguration _csvConfiguration;

        private CsvConfiguration CsvConfiguration
        {
            get
            {
                _csvConfiguration ??= new CsvConfiguration
                {
                    Delimiter = ';',
                    SupportsMultiline = false
                };

                return _csvConfiguration;
            }
        }

        public override ExportConfigurationInfo ConfigurationInfo => new()
        {
            ConfigurationWidget = new ComponentWidget<HelloWorldConfigurationViewComponent>(),
            ModelType = typeof(ProfileConfigurationModel)
        };

        protected override async Task ExportAsync(ExportExecuteContext context, CancellationToken cancelToken)
        {
            var config = (context.ConfigurationData as ProfileConfigurationModel) ?? new ProfileConfigurationModel();

            using var writer = new CsvWriter(new StreamWriter(context.DataStream, Encoding.UTF8, 1024, true));

            writer.WriteFields(new string[]
            {
                "ProductName",
                "SKU",
                "Price",
                "Savings",
                "Description"
            });
            writer.NextRow();
            
            while (context.Abort == DataExchangeAbortion.None && await context.DataSegmenter.ReadNextSegmentAsync())
            {
                var segment = await context.DataSegmenter.GetCurrentSegmentAsync();

                foreach (dynamic product in segment)
                {
                    if (context.Abort != DataExchangeAbortion.None)
                    {
                        break;
                    }

                    Product entity = product.Entity;

                    try
                    {
                        var calculatedPrice = (CalculatedPrice)product._Price;
                        var saving = calculatedPrice.Saving;

                        writer.WriteFields(new string[]
                        {
                            product.Name,
                            product.Sku,
                            ((decimal)product.Price).FormatInvariant(),
                            saving.HasSaving ? saving.SavingPrice.Amount.FormatInvariant() : string.Empty,
                            ((string)product.FullDescription).Truncate(5000)
                        });
                        writer.NextRow();
                        ++context.RecordsSucceeded;

                        if (context.RecordsSucceeded >= config.NumberOfExportedRows)
                        {
                            context.Abort = DataExchangeAbortion.Soft;
                        }
                    }
                    catch (OutOfMemoryException ex)
                    {
                        context.RecordOutOfMemoryException(ex, entity.Id, T);
                        context.Abort = DataExchangeAbortion.Hard;
                        throw;
                    }
                    catch (Exception ex)
                    {
                        context.RecordException(ex, entity.Id);
                    }
                }
            }
        }
    }
}
```
{% endcode %}

### The XML export provider

XML export is very similar to CSV. First you need to change the _file extension_ to `XML`.

```csharp
public override string FileExtension => "XML";
```

The `Localizer` and `ConfigurationInfo` stay the same. Now you only need to modify `ExportAsync`.

For XML the writer comes from an `ExportXmlHelper`.

```csharp
using var helper = new ExportXmlHelper(context.DataStream);
var writer = helper.Writer;
```

Next you start the document and write the grouping tag.

```csharp
writer.WriteStartDocument();
writer.WriteStartElement("products");
```

Same as with CSV provider, you fetch the next data segment, iterate through the products.

```csharp
while (context.Abort == DataExchangeAbortion.None && await context.DataSegmenter.ReadNextSegmentAsync())
{
    var segment = await context.DataSegmenter.GetCurrentSegmentAsync();

    foreach (dynamic product in segment)
    {
        if (context.Abort != DataExchangeAbortion.None)
        {
            break;
        }

        Product entity = product.Entity;
    }
}
```

Write a new product XML node including the values to be exported.

```csharp
writer.WriteStartElement("product");

try
{
    var calculatedPrice = (CalculatedPrice)product._Price;
    var saving = calculatedPrice.Saving;

    writer.WriteElementString("product-name", (string)product.Name);
    writer.WriteElementString("sku", (string)product.Sku);
    writer.WriteElementString("price", ((decimal)product.Price).FormatInvariant());

    if (saving.HasSaving)
    {
        writer.WriteElementString("savings", saving.SavingPrice.Amount.FormatInvariant());
    }

    writer.WriteCData("desc", ((string)product.FullDescription).Truncate(5000));

    context.RecordsSucceeded++;
    
    // Row limitation and catch block left out for brevity
}

writer.WriteEndElement(); // product
```

And when the while-loop is complete, the grouping and the document need to be closed.

```csharp
writer.WriteEndElement(); // products
writer.WriteEndDocument();
```

You can find the source code in `HelloWorldXmlExportProvider.cs`.

## Delete export profiles

Finally you just need to clean up any existing profiles on `UnistallAsync` in `Module.cs`.

Pass `SmartDbContext` and `IExportProfileService` to the module class constructor.

```csharp
private readonly SmartDbContext _db;
private readonly IExportProfileService _exportProfileService;

public Module(SmartDbContext db, IExportProfileService exportProfileService)
{
    _db = db;
    _exportProfileService = exportProfileService;
}
```

Then add the following lines to the beginning of your `UninstallAsync` method.

```csharp
// Read the export profile entities associated with your export provider
var profiles = await _db.ExportProfiles
    .Include(x => x.Deployments)
    .Include(x => x.Task)
    .Where(x => x.ProviderSystemName == "MyOrg.HelloWorld.ProductCsv" || x.ProviderSystemName == "MyOrg.HelloWorld.ProductXml")
    .ToListAsync();

// Now delete the entities and any related file
await profiles.EachAsync(x => _exportProfileService.DeleteExportProfileAsync(x, true));
```

## Conclusion

In this tutorial you built an export provider. You created a configuration profile and a CSV export provider.

The code for this module can be downloaded here:

{% file src="../../../.gitbook/assets/MyOrg.HelloWorldExport.zip" %}
