# 🐣 DbContextScope

An instance of `SmartDbContext` has a short lifetime. It can be retrieved from the pool, when a HTTP request begins, and is returned to the pool, when the request ends. To change the configuration of the request scoped context, you can:

1. Create a custom nested scope.
2. Do several actions within the scope.
3. Dispose the nested scope.

The context’s original state, as it was when you created the scope, is automatically restored when the scope is disposed. Use a custom scope when you want to:

* Disable automatic change detection in long-running operations, which slows down the more entities the change tracker contains.
* Speed up write access by disabling hook invocation, as both hook resolution and hook invocation take some time to process.
* Force the scope to be atomic, which saves all changes when the scope ends or is explicitly committed.

To create a scope where a `DbContext` instance behaves differently, use [DbContextScope](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Data/DbContextScope.cs). The following code shows an excerpt from [MediaService.Folder.cs](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Content/Media/MediaService.Folder.cs).

{% code overflow="wrap" %}
```csharp
using (var scope = new DbContextScope(_db, autoDetectChanges: false,
                            deferCommit: true))
{
    destinationPath += "/" + node.Value.Name;
    var dupeFiles = new List<DuplicateFileInfo>();

    // >>>> Do the heavy stuff
    var folder = await InternalCopyFolder(scope, node, destinationPath,
                            dupeEntryHandling, dupeFiles, cancelToken);

    var result = new FolderOperationResult
    {
        Operation = "copy",
	DuplicateEntryHandling = dupeEntryHandling,
	Folder = folder,
	DuplicateFiles = dupeFiles
    };

    return result;
}
```
{% endcode %}

## Parameter reference

When initializing a `DbContextScope` instance, any non-`null` value will change the corresponding option for the duration of the scope. The `db` parameter is the only one required.

| Parameter             | Description                                                                                                                               |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `db`                  | The data context instance that the scope is uses.                                                                                         |
| `autoDetectChanges`   | Toggles EF's _automatic entity change detection_ feature.                                                                                 |
| `lazyLoading`         | Toggles EF's _lazy loading_ feature.                                                                                                      |
| `forceNoTracking`     | If `true`, queries are not tracked, unless specified otherwise.                                                                           |
| `deferCommit`         | Suppresses the execution of `SaveChanges` until the scope is disposed or Commit is called explicitly. The default is `false`.             |
| `retainConnection`    | Opens a connection and retains it until the scope is closed. May increase load/save performance for large scopes. The default is `false`. |
| `minHookImportance`   | Specifies the minimum hook importance level. Hooks below this level will not be executed.                                                 |
| `cascadeDeleteTiming` | Sets the `CascadeDeleteTiming` option in EF's change tracker.                                                                             |
| `deleteOrphansTiming` | Sets the `DeleteOrphansTiming` option in EF's change tracker.                                                                             |
