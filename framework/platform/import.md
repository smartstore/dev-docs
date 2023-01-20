# Import

## Overview

* The [data importer](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/DataExchange/Import/DataImporter.cs) provides batches of data that are imported by an [IEntityImporter](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/DataExchange/Import/IDataImporter.cs) implementation.
* [Import profiles](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/DataExchange/Domain/ImportProfile.cs) are entities that are binding the import to an [ImportEntityType](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/DataExchange/Domain/ImportEnums.cs) and combining and storing all aspects of an import like column mapping and settings making it configurable by the user.
* When an import is executed, a task associated with the import profile is started, which performs the actual import via data importer and `IEntityImporter` implementation. The task can be triggered manually or scheduled.

## Data importer

The data importer is an [IDataImporter](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/DataExchange/Import/IDataImporter.cs) implementation with the purpose to provide batches of import data to `IEntityImporter` implementations in a high-performance way.

### Events

The [ImportExecutingEvent](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/DataExchange/Events/ImportExecutingEvent.cs) is fired before a data import. It can be used, for example, to load custom data into the context object, which needs to be available during the entire import.

The [ImportBatchExecutedEvent](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/DataExchange/Events/ImportBatchExecutedEvent.cs) is fired by the `IEntityImporter` after it has imported a batch of data. It can be used, for example, to import the data of additionally attached columns, data of which the `IEntityImporter` has no knowledge.

The [ImportExecutedEvent](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/DataExchange/Events/ImportExecutedEvent.cs) is fired after a data import. It can be used, for example, to remove data from the cache so that the imported data is taken into account the next time it is accessed.

## Entity importer

An import is realised via an [IEntityImporter](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/DataExchange/Import/IDataImporter.cs) implementation or it can inherit from [EntityImporterBase](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/DataExchange/Import/EntityImporterBase.cs) which provides helper methods which can be used by all importers, like importing localized properties. This documentation refers to an importer that inherits from `EntityImporterBase`.

`ProcessBatchAsync` is the main method to import data. It is called several times by the data importer during an import (once for each data batch). The importer is created anew for each batch via its own dependency scope. This ensures that there are no unwanted interactions with the scope of the data importer.

Avoid reloading the same data for each batch. Use `ImportExecuteContext.CustomProperties` for storing extra data which needs to be available during the entire import. You can load them once using the `ImportExecutingEvent` or a helper method like:

```csharp
private async Task<MyImporterCargoData> GetCargoData(ImportExecuteContext context)
{
    const string key = "MyCompany.MyEntityImporter.CargoData";
    if (context.CustomProperties.TryGetValue(key, out object value))
    {
        return (MyImporterCargoData)value;
    }

    var templates = await _db.CategoryTemplates
        .AsNoTracking()
        .OrderBy(x => x.DisplayOrder)
        .ToListAsync(context.CancelToken);

    // Better not pass entities here because of batch scope!
    var result = new MyImporterCargoData
    {
        TemplateViewPaths = templates.ToDictionarySafe(x => x.ViewPath, x => x.Id)
    };

    context.CustomProperties[key] = result;
    return result;
}
```

TIP: often there is data that can only be imported after all others have been imported. To handle this, call `context.DataSegmenter.IsLastSegment` if you want to know whether the current batch is the last one (i.e. no more will follow).

{% code title="A simple entity importer" %}
```csharp
public class MyEntityImporter : EntityImporterBase
{
    public MyEntityImporter(
        ICommonServices services,
        ILocalizedEntityService localizedEntityService,
        IStoreMappingService storeMappingService,
        IUrlService urlService,
        SeoSettings seoSettings)
        : base(services, localizedEntityService, storeMappingService, urlService, seoSettings)
    {
    }

    protected override async Task ProcessBatchAsync(ImportExecuteContext context, CancellationToken cancelToken = default)
    {
        var segmenter = context.DataSegmenter;
        var batch = segmenter.GetCurrentBatch<MyEntity>();

        using (var scope = new DbContextScope(_services.DbContext, autoDetectChanges: false, minHookImportance: HookImportance.Important, deferCommit: true))
        {
            await context.SetProgressAsync(segmenter.CurrentSegmentFirstRowIndex - 1, segmenter.TotalRows);

            try
            {
                // Import batch of entities...
            }
            catch (Exception ex)
            {
                context.Result.AddError(ex, segmenter.CurrentSegment, "My main entity import");
            }

            // Import related data...
        }

        await _services.EventPublisher.PublishAsync(new ImportBatchExecutedEvent<MyEntity>(context, batch), cancelToken);
    }
```
{% endcode %}

For more sample code, see the core's built-in importers such as `ProductImporter`, `CategoryImporter` etc.

## Media importer

The media importer is a helper for importing media files like images. It can and should be used by any other importer. TODO: more....

## Custom import via module

There is no provider mechanism available for data imports like in export infrastructure. So you cannot simply bind your custom importer to an import profile. This would require an extension of the core. In order to realise a custom import via a module, you have to provide an importer from scratch and a task that calls it directly. First, register your importer:

```csharp
internal class Startup : StarterBase
{
    public override void ConfigureServices(IServiceCollection services,
        IApplicationContext appContext)
    {
        if (appContext.IsInstalled)
        {
            services.AddScoped<MyCustomImporter>();
        }
    }
}
```

Then add a task which executes it:

```csharp
public class MyCustomImportTask : ITask
{
    private readonly MyCustomImporter _myCustomImporter;

    public MyCustomImportTask(MyCustomImporter myCustomImporter)
    {
        _myCustomImporter = myCustomImporter;
    }

    public Task Run(TaskExecutionContext context, CancellationToken cancelToken)
        => _myCustomImporter.ExecuteAsync(context, cancelToken);
}
```

Finally, implement your importer:

```csharp
public class MyCustomImporter
{
    public async Task ExecuteAsync(TaskExecutionContext context,
        CancellationToken cancelToken)
    {
        const string directoryName = "MyCustomImporter";
        var root = _appContext.TenantRoot;
        var logPath = PathUtility.Join(directoryName, "import-log.txt");
        var logFile = await root.GetFileAsync(logPath);
        var importContext = await CreateMyImportContext();

        using (var logger = new TraceLogger(logFile, false))
        {
            importContext.Log = logger;
            // Now import data from your data source...
        }
    }
    
    private async Task<MyImportExecuteContext> CreateMyImportContext()
    {
        // Create and init custom import context object...
    }
}
```
