# Creating an Export provider

Using export providers you can export your shop data into many different formats. Smartstore primarily uses CSV and XML.

In this tutorial, you will write an export provider for the product catalog.

## Create a configuration

If you want to make your export provider customisable, you will need to a configuration. This step is optional.

The configuration is made up of three things:

1. The profile configuration model `ProfileConfigurationModel` that describes the used data.
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
            <smart-editor asp-for="NumberOfExportedRows"></smart-editor>
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

Smartstore uses the following attributes to integrate the providers correctly: SystemName, FriendlyName and ExportFeatures.

Add these attributes to your class definition.

```csharp
[SystemName("MyOrg.HelloWorld.ProductXml")]
[FriendlyName("Hello world XML product feed")]
[ExportFeatures(Features =
    ExportFeatures.CreatesInitialPublicDeployment |
    ExportFeatures.OffersBrandFallback]
```

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

### The XML export provider

## Delete export profiles

Finally you just need to clean up any existing profiles on `UnistallAsync` in `Module.cs`.

Add a `SmartDbContext` and an `IExportProfileService` via _Dependency Injection_.

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
var profiles = await _db.ExportProfiles
    .Include(x => x.Deployments)
    .Include(x => x.Task)
    .Where(x => x.ProviderSystemName == HelloWorldCsvExportProvider.SystemName)
    .ToListAsync();

await profiles.EachAsync(x => _exportProfileService.DeleteExportProfileAsync(x, true));
```

This does two things:

1. Fetch all export profiles that are associated with your export provider.
2. Force delete each profile.

## Conclusion

Blablabla

\<Insert File here>
