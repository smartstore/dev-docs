---
description: Special pub/sub system for database save operations
---

# Hooks

* .Like database _triggers_, hooks are subscribers that are automatically executed in response to certain save/commit events on a particular `DbContext` instance.
* But unlike triggers, hooks are high-level, data provider agnostic, and pure managed code.
* Extremely powerful and very flexible when it comes to composable, granular application design.
* They are similar to MVC action filters
* Hooks lets you concentrate on the aspect you want to solve without ever touching the application core
* Smartstore heavily relies on hooks. Without them, granular and isolated application design is merely impossible. Modules would be much less flexible.
* Some examples what hooks are good for:
  * Invalidating cache entries
  * Perform some logging
  * Sending notifications
  * Removing dependant entities after some primary entities has been deleted
  * Update an index
  * Removing orphaned resources
  * Validate, fix or enrich an entity before save
  * Update some computed data
  * and many more...

## Concept

* It is a specialized pub/sub system without the _pub_ part.
* You only can subscribe to database events, but not publish them.
* Publishing is performed implicitly during a database save operation... when `DbContext.SaveChanges()` is executed. This is always the case when `SmartDbContext` - the application main context - commits data.... because `SmartDbContext` derives from [`HookingDbContext`](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Data/HookingDbContext.cs), which contains all the hooking logic.
* WARN: bypassing EF and accessing the database directly (with raw SQL) --> No events, no hooking!
* Every hook has a _PreSave_ and a _PostSave_ handler.
  * _PreSave_ handler is executed for every entity in the EF change tracker right BEFORE a save operation
  * The actual save operation is performed, then
  * _PostSave_ handler is executed for every entity in the EF change tracker
  * _PreSave_ handler purpose:
    * Validate entity
    * Fix or enrich entity
    * Change entity state (e.g. to suppress save)
    * Check which properties have been modified (what you can't do in _PostSave_ handlers)
    * etc.
  * _PostSave_ handler purpose:
    * To do something after **making sure** that an entity has actually been saved
    * But: snapshot comparison is not possible in this stage anymore

## Implementing hooks

* Create a concrete class...:
  * that implements [`IDbSaveHook`](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Data/Hooks/IDbSaveHook.cs), or
  * that implements `IDbSaveHook<TContext>` (to bind the hook to a particular `DbContext` type), or
  * that derives from [`Smartstore.Data.Hooks.AsyncDbSaveHook<TContext, TEntity`](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Data/Hooks/AsyncDbSaveHook.cs)`>` (to bind the hook to a particular `DbContext` and given `TEntity` type), or
  * that derives from `Smartstore.Core.Data.AsyncDbSaveHook<TEntity>` (to bind the hook to the main `SmartDbContext` type and given `TEntity` type)
  * INFO: The abstract base classes are nothing special, they just implement `IDbSaveHook` to make your life easier. Also: there are sync counterparts for the base classes with sync method signatures.
  * INFO: If bound to entity type `TEntity`: matches all saved entities that are equal to or are subclasses of `TEntity`.
* No need to register in DI, it is auto-discovered and registered as a scoped service on app startup. So: hook types can take any dependency.
* TIPP: you can also apply the above interface/base class to any existing service class.

### Interface definition

```csharp
/// <summary>
/// A hook that is executed before and after a database save operation.
/// Raising <see cref="NotSupportedException"/> or
/// <see cref="NotImplementedException"/> will be treated just 
/// like <see cref="HookResult.Void"/>.
/// </summary>
public interface IDbSaveHook
{
    /// <summary>
    /// Called before an entity is about to be saved.
    /// </summary>
    /// <param name="entry">The entity entry</param>
    /// <returns>
    ///     <see cref="HookResult.Ok"/>: signals the hook handler that it should continue 
    ///        to call this method for the current EntityType/State/Stage combination,
    ///     <see cref="HookResult.Void"/>: signals the hook handler that it should stop 
    ///        executing this method for the current EntityType/State/Stage combination.
    /// </returns>
    Task<HookResult> OnBeforeSaveAsync(
        IHookedEntity entry, 
        CancellationToken cancelToken);

    /// <summary>
    /// Called after an entity has been successfully saved.
    /// </summary>
    /// <param name="entry">The entity entry</param>
    /// <returns>
    ///     <see cref="HookResult.Ok"/>: signals the hook handler that it should continue 
    ///        to call this method for the current EntityType/State/Stage combination,
    ///     <see cref="HookResult.Void"/>: signals the hook handler that it should stop
    ///        executing this method for the current EntityType/State/Stage combination.
    /// </returns>
    Task<HookResult> OnAfterSaveAsync(
        IHookedEntity entry, 
        CancellationToken cancelToken);

    /// <summary>
    /// Called after all entities in the current unit of work have been handled 
    /// right before saving changes to the database.
    /// All entities that were handled with <see cref="HookResult.Void"/> 
    /// result in <see cref="OnBeforeSaveAsync(IHookedEntity, CancellationToken)"/>
    /// will be excluded from <paramref name="entries"/>.
    /// </summary>
    Task OnBeforeSaveCompletedAsync(
        IEnumerable<IHookedEntity> entries, 
        CancellationToken cancelToken);

    /// <summary>
    /// Called after all entities in the current unit of work have been handled 
    /// after saving changes to the database.
    /// All entities that were handled with <see cref="HookResult.Void"/> 
    /// result in <see cref="OnAfterSaveAsync(IHookedEntity, CancellationToken)"/>
    /// will be excluded from <paramref name="entries"/>.
    /// </summary>
    Task OnAfterSaveCompletedAsync(
        IEnumerable<IHookedEntity> entries, 
        CancellationToken cancelToken);
}
```

### Hook Result

{% code title="Definition" %}
```csharp
/// <summary>
/// The result of a database save hook operation.
/// </summary>
public enum HookResult
{
    /// <summary>
    /// Signals the hook handler that it never should process the hook
    /// again for the current EntityType/State/Stage combination.
    /// </summary>
    Void = -1,

    /// <summary>
    /// Operation was handled but completed with errors.
    /// Failed hooks will be absent from 
    /// <see cref="IDbSaveHook.OnBeforeSaveCompletedAsync(IEnumerable{IHookedEntity}, CancellationToken)"/>
    /// or <see cref="IDbSaveHook.OnAfterSaveCompletedAsync(IEnumerable{IHookedEntity}, CancellationToken)"/>
    /// </summary>
    Failed,

    /// <summary>
    /// Operation was handled and completed without errors.
    /// </summary>
    Ok
}
```
{% endcode %}

* A hook method returns a result of type `HookResult`.
* For performance reasons it is absolutely essential that you return `HookResult.Void` if the current entity type/state/stage combination is of no interest for the hook and thus will not be handled.&#x20;
* INFO: Instead of returning `HookResult.Void` you can also throw `NotImplementedException` or `NotSupportedException`.
* This way you signal the hooking framework that it should stop executing the hook for the given combination in successive save operations.&#x20;
* This is some kind of a _filter_ which reduces the amount of classes that have to be instantiated repeatedly, only to find out that there's nothing to do.

An example for clarification:

```csharp
/// <summary>
/// This hook handles all entity types that derive from "BaseEntity".
/// Because every entity type derive from "BaseEntity", this hook
/// will potentially be instantiated and executed too many times.
/// </summary>
internal class MyCacheInvalidatorHook : AsyncDbSaveHook<BaseEntity>
{
    public override Task<HookResult> OnAfterSaveAsync(IHookedEntity entry, CancellationToken cancelToken)
    {
        var e = entry.Entity;

        // Because the given entity types below have no shared interface, 
        // we were forced to specify "BaseEntity" as generic 
        // type argument for this class.
        var shouldHandle = e is Product || e is Topic || e is Category || e is Manufacturer || e is StoreMapping;
        if (!shouldHandle)
        {
            // Not of interest
            return Task.FromResult(HookResult.Void);
        }
        else
        {
            if (entry.InitialState != Smartstore.Data.EntityState.Added)
            {
                // Entity type matches, but state is not of interest.
                // We gonna handle only new entities in this hook.
                return Task.FromResult(HookResult.Void);
            }
            else
            {
                // ...Do something useful...

                return Task.FromResult(HookResult.Ok);
            }
    }
}
```

### IHookedEntity

* Passed to the hook handler method
* Represents the entity entry that is being hooked
* `Entry`:  the underlying EF entity entry
* `Entity`:  instance of hooked entity
* `State`:  the **current** entity state
* `InitialState`:  the entity state before the save operation. Use this in _PostSave_ handlers.
* `HasStateChanged`:  whether the entity state was changed in a _PreSave_ handler
* `IsSoftDeleted`:  whether the entity is in soft deleted state. This is the case when the entity is an instance of `ISoftDeletable` and the value of its `Deleted` property is true AND has changed since tracking. But when the entity is not in modified state the snapshot comparison is omitted.

#### Check for modified properties

* If an entity is in _Modified_ state, you can check for modified properties
* But only in a _PreSave_ handler. Because the row snapshot is resetted after save.
* `IsPropertyModified(string propertyName)`:  gets a value indicating whether the given property has been modified
* `Entry.GetModifiedProperties()`: gets a dictionary with modified properties for the specified entity. The key is the name of the modified property and the value is its ORIGINAL value (which was tracked when the entity was attached to the context the first time). Returns an empty dictionary if no modification could be detected.
* `Entry.`TryGetModifiedProperty`()`: _Describe_

### Abstract base class

* For more convenience
* Provides six overridable methods: `OnInserting`, `OnUpdating`, `OnDeleting`, `OnInserted`, `OnUpdated`, `OnDeleted`
* Each method returns `HookResult.Void` by default
* You just need to opt-in by overriding
* _SAMPLE_ (with some explanatory code comments)

### Batching

* Sometimes it may be preferable to hook the whole save batch instead of hooking entities one by one: e.g. when your hook executes some expensive code.
* Imagine:&#x20;
  * an import process always saves product entities batch-wise, 100 products per batch
  * a _PostSave_ product hook handler would be called 100x for this save operation --> `OnAfterSaveAsync`
  * But the batch handler is called only once --> `OnAfterSaveCompletedAsync`
  * All entities that ran through `OnAfterSaveAsync` before are passed to this method as a collection
  * WARN: All entities that were handled with `HookResult.Void` in `OnAfterSaveAsync` will be excluded from the entries collection, because they are _not of interest_.
* _SAMPLE_

### Importance

* By default, hook importance is specified as `Normal`.
* For performance reasons, some callers may reduce the amount of executed hooks by specifying the so-called `MinHookImportance` for particular unit of works.&#x20;
* E.g., the product import task, which is a long-running process, turns off the execution of `Normal` hooks by setting `MinHookImportance` to `Important`.
* This is done by wrapping a [`DbContextScope`](../advanced/dbcontextscope.md) around a unit of work
* To customize your hook's importance, decorate your hook class with `ImportantAttribute`:
  * `Normal` (default): Hook can be ignored during long running processes, e.g. imports. Usually simple hooks that invalidate cache entries or clean some resources.
  * `Important`: Hook is important and should also run during long running processes. Not running the hook **may** result in stale or invalid data.
  * `Essential`: Hook instance should run in any case (e.g. _AuditHook_, which is even required during installation). Not running the hook **will definitely** result in stale data or raise exceptions.

### Specify execution order

* Default execution order is 0.
* Decorate your hook class with [`OrderAttribute`](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Modularity/OrderAttribute.cs) and pass an integer value.

### Mark entities as unhookable

* Some entity types should not be hooked at all, e.g. the [`Log`](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Logging/Domain/Log.cs) entity.
* Decorate your entity class with `HookableAttribute` and pass _false_.
* The hooking framework will never pass these entities to any hook

## Some Tipps

* Most hooks are just cache invalidators
* Separating cache invalidation from cache access makes code somehow confusing
* Therefore we recommend to combine them in a single class
* _SAMPLE_ (pattern see: [DiscountService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Catalog/Discounts/Services/DiscountService.cs), [ProductTagService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Catalog/Products/Services/ProductTagService.cs), [CurrencyService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Common/Services/CurrencyService.cs), [DeliveryTimeService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Common/Services/DeliveryTimeService.cs) etc.)
