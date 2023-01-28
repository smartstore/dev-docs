# 🥚 Export

## Overview

* The [data exporter](export.md#data-exporter) provides the data in segments that are written in a specific format into a stream by an [export provider](export.md#export-provider).
* [Export profiles](export.md#export-profile) are entities that are binding the export to an export provider and combining and storing all aspects of an export like settings and configurations making it configurable by the user.
* When an export is executed, a task associated with the export profile is started, which performs the actual export via data exporter and export provider. The task can be triggered manually or scheduled.
* [Export deployments](export.md#deployment) are entities that can optionally be assigned to an export profile to specify how to proceed with the export files, for example to send them to an e-mail address.

## Data exporter

The data exporter is an [IDataExporter](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/DataExchange/Export/IDataExporter.cs) implementation and the main core component of the export infrastructure. Its purpose is to provide the export data to [export providers](export.md#export-provider) in a high-performance way and to manage general tasks such as file management or data preview.

### Events

The [RowExportingEvent](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/DataExchange/Events/RowExportingEvent.cs) is fired before an entity is exported. It can be used, for example, to attach and export additional data.

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

Your provider is displayed in the provider select box when adding a new export profile. For another example, see the [Google Merchant Center export provider](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Modules/Smartstore.Google.MerchantCenter/Providers/GmcXmlExportProvider.cs).

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
    ConfigurationWidget = new ComponentWidget<MyConfigurationViewComponent>(),
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

Your configuration is displayed in the **Configuration** tab on the profile edit page.&#x20;

### Export data

The export data is provided in segments by the data exporter. A data item is a dynamic object of type [DynamicEntity](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/DataExchange/Export/Internal/DynamicEntity.cs) which wraps the actual entity and to which extra properties are attached. The entity is accessibly via `dynamicObject.Entity`. Additional data is generally prefixed with an underscore, e.g. `dynamicObject._BasePriceInfo`. See [appendix](export.md#properties-of-dynamicentity) for complete list.

`DynamicEntity` has projection and configuration of the export profile applied. For example if a certain language is selected in the profile's projection tab, the `DynamicEntity` contains the localized property values (e.g. a localized product name) instead of the actual entity property value.

### Export related data

Export profile has option `ExportRelatedData`. If activated and if provider supports `ExportFeatures.UsesRelatedDataUnits` then data exporter adds additional data units to `ExportExecuteContext.ExtraDataUnits`. The provider can use them to export related data into a separate file, for example to additionally export tier prices through a product export provider. At the moment only tier prices, variant attribute values and variant attribute combinations are supported when exporting products, see `RelatedEntityType`. This mechanism is intended mainly for updating prices via flat data formats such as CSV, which can be edited by end users.

TIP: related data files are automatically imported together with the main data file(s) using file naming convention. If the name of the related data file ends with a `RelatedEntityType` value (e.g. `TierPrice` or `ProductVariantAttributeCombination`) then the product importer uses its data to update tier prices, variant attribute values or variant attribute combinations.

## Export profile

Export profiles combine all aspects of an export to make it configurable by the user: provider, task, partition, filters, projections, configuration and deployments. An export provider can be assigned to several profiles with different configurations and settings. Two types: built-in system profiles and user profiles that have been added subsequently by the user. System profiles are used, for example, when exporting orders via the order grid in the backend.

Use [IExportProfileService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/DataExchange/Export/IExportProfileService.cs) to manage export profiles. For instance if you want to delete all profiles that have your export provider assigned when your module is unistalled. Use `ExportProfileInfoViewComponent` if you want display a list of all profiles that have your provider assigned. It renders a link to the profile and task, an info about last execution and a button to start the export:

```csharp
@await Component.InvokeAsync("ExportProfileInfo", new { providerSystemName = "Exports.MyCompanyProductCsv" })
```

### Deployment

Publishing profiles are implementations of [IFilePublisher](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/DataExchange/Export/Deployment/IFilePublisher.cs) and define how to further proceed with the export files. Built-in are publishing via the file system, email, HTTP POST, FTP or public folder. Any number of publishing profiles can be assigned to an export profile.

After a successful export the data exporter instantiates every publisher associated with the export profile and calls its `PublishAsync` method together with the [ExportDeployment](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/DataExchange/Domain/ExportDeployment.cs) entity containing all details of the deployment.

## Data grid and exports

Sometimes it's usefull to show a button or menu above a data grid, for example to let the user directly export all or selected orders via the orders grid. Some steps needed to implement this. First you need to add an `IResultFilter` to register your `PartialViewWidget` or alternatively a view component:

```csharp
internal class Startup : StarterBase
{
    public override void ConfigureServices(IServiceCollection services, IApplicationContext appContext)
    {
        services.Configure<MvcOptions>(o =>
        {
            o.Filters.AddConditional<MyOrdersGridToolbarFilter>(
                context => context.ControllerIs<OrderController>(x => x.List()) && !context.HttpContext.Request.IsAjax());
        });
    }
}

public class MyOrdersGridToolbarFilter : IResultFilter
{
    private readonly IPermissionService _permissionService;
    private readonly Lazy<IWidgetProvider> _widgetProvider;

    public MyOrdersGridToolbarFilter(
        IPermissionService permissionService,
        Lazy<IWidgetProvider> widgetProvider)
    {
        _permissionService = permissionService;
        _widgetProvider = widgetProvider;
    }

    public void OnResultExecuting(ResultExecutingContext context)
    {
        if (context.Result.IsHtmlViewResult() 
            && _permissionService.Authorize(Permissions.Configuration.Export.Execute))
        {
            _widgetProvider.Value.RegisterWidget("datagrid_toolbar_omega",
                new PartialViewWidget("_MyOrdersGridExportMenu", null, "MyCompany.MyModule"));
        }
    }

    public void OnResultExecuted(ResultExecutedContext context) { }
}
```

Your partial view with the toobar menu may look like this:

```cshtml
<div class="dg-toolbar-group">
    <div class="dropdown">
        <button type="button" class="btn btn-light btn-flat dropdown-toggle" 
            data-toggle="dropdown" data-boundary="window">
            @T("Plugins.MyCompany.MyModule.Export")
        </button>
        <div class="dropdown-menu">
            <a href="javascript:;" class="dropdown-item mycompany-order-export 
               export-selected-data"
               v-bind:class="{ disabled: !grid.hasSelection }"
               data-grid-id="orders-grid"
               data-url="@Url.Action("Export", "MyController", new { all=false })">
                <i class="fa fa-fw fa-code"></i>
                <span>@T("Plugins.MyCompany.MyModule.ExportSelected")</span>
            </a>

            <a href="javascript:;" class="dropdown-item mycompany-order-export"
               data-url="@Url.Action("Export", "MyController", new { all=true })">
                <i class="fa fa-fw fa-code"></i>
                <span>@T("Plugins.MyCompany.MyModule.ExportAll")</span>
            </a>
        </div>
    </div>
</div>

<script sm-target-zone="scripts" data-origin="mycompany-order-export">
    $(function () {
        $('.mycompany-order-export').on('click', function (e) {
            e.preventDefault();

            var link = $(this);
            var selectedIds = '';

            if (!link.hasClass('disabled')) {
                if (link.hasClass('export-selected-data')) {
                    var elGrid = $('#' + link.data('grid-id'));

                    if (!elGrid.length) {
                        alert2('Cannot find order grid.');
                        return false;
                    }

                    selectedIds = elGrid.parent().data('datagrid').selectedRowKeys;
                }

                $({}).postData({
                    url: link.data('url'),
                    data: selectedIds != "" ? { selectedIds } : null,
                    ask: @T("Admin.Common.AskToProceed").JsValue,
                });
            }

            return false;
        });
    });
</script>
```

## Appendix

### Properties of DynamicEntity

#### All entity types

| Property name (type)         | Description                                                                                                                                                                                                                                                                                                                                                   |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| \_Localized (List\<dynamic>) | <p>List of all localized values of all languages for an entity. <code>null</code> if the entity has no localized properties. Properties of an item are</p><p><em>Culture</em>: the language culture.</p><p><em>LocaleKey</em>: the key of the localized value (usually the property name of the entity).</p><p><em>LocaleValue</em>: the localized value.</p> |

#### Products

| Property name (type)                                                | Description                                                                                                                                  |
| ------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| \_UniqueId (string)                                                 | A unique ID consisting of product ID and attribute combination ID if an attribute combination is exported as a product.                      |
| \_IsParent (bool)                                                   | A value indicating whether the product is a "parent" where attribute combinations are exported as products and are considered as "children". |
| \_AttributeCombinationId (int)                                      | The attribute combination ID if it's exported as a product.                                                                                  |
| \_AttributeCombinationValues (IList\<ProductVariantAttributeValue>) | A list of all attribute values if an attribute combination is exported as a product.                                                         |
| \_Brand (string)                                                    | Namen of the first assigned manufacturer.                                                                                                    |
| \_CategoryName (string)                                             | The name of the first assigned category.                                                                                                     |
| \_CategoryPath (string)                                             | The breadcrumb (path) of the first assigned category.                                                                                        |
| \_DetailUrl (string)                                                | URL to product detail page.                                                                                                                  |
| \_Price (CalculatedPrice)                                           | The calculated product price.                                                                                                                |
| \_BasePriceInfo (string)                                            | Base price info of the product.                                                                                                              |
| \_MainPictureUrl (string)                                           | Absolute URL of the product's main picture.                                                                                                  |
| \_MainPictureRelativeUrl (string)                                   | Relative URL of the product's main picture.                                                                                                  |
| \_FreeShippingThreshold (decimal?)                                  | Free shipping threshold from projection settings.                                                                                            |
| \_ShippingCosts (decimal?)                                          | Shipping costs or the projected costs if the product price is greater than or equal to the free shipping threshold.                          |
| \_ShippingTime (string)                                             | Delivery time name or projected time if none is specified for the product.                                                                   |
| \_ProductTemplateViewPath (string)                                  | The view path of the assigned product template.                                                                                              |

#### Customers

| Property name (type)                 | Description                                                                                                                                      |
| ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| \_FullName (string)                  | Full customer name.                                                                                                                              |
| \_AvatarPictureUrl (string)          | Absolute URL of the avatar (if any).                                                                                                             |
| \_RewardPointsBalance (int)          | Current reward points balance.                                                                                                                   |
| \_HasNewsletterSubscription (bool)   | A value  indicating whether the customer is subscribed to the newsletter.                                                                        |
| \_GenericAttributes (List\<dynamic>) | List of associated generic attributes. Only VatNumber and ImpersonatedCustomerId are exported. The dynamic items are of type `GenericAttribute`. |

#### MediaFile

| Property name (type)        | Description                                                                                                 |
| --------------------------- | ----------------------------------------------------------------------------------------------------------- |
| \_FileName (string)         | Name of the file.                                                                                           |
| \_RelativeUrl (string)      | Relative URL.                                                                                               |
| \_ThumbImageUrl (string)    | Absolute URL of the thumbnail.                                                                              |
| \_ImageUrl (string)         | Absolute URL of the image in the size shown on a detail page. If not available, then the thumbnail is used. |
| \_FullSizeImageUrl (string) | Absolute URL of the full size image.                                                                        |
