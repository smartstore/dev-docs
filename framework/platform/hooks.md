---
description: Special pub/sub system for database save operations
---

# 🥚 Hooks

## Overview

Like database triggers, hooks are subscribers that are automatically executed in response to certain save / commit events on a particular `DbContext` instance.

But unlike triggers, hooks are:

* high-level
* data provider agnostic
* pure managed code

Hooks let you focus on the aspect you want to solve without ever touching the core of the application. They are extremely powerful and flexible when it comes to composable, granular application design and are similar to MVC action filters.

Smartstore relies heavily on hooks. Without them, granular and isolated application design would be nearly impossible. Modules would be much less flexible.

Some examples of what hooks are good for:

* Invalidating cache entries
* Updating computed data
* Validate, fix or enrich an entity before saving
* Removing dependent entities after deleting primary entities
* Perform logging
* Sending notifications
* Updating an index
* Removing orphaned resources

## Concept

A hook is a specialized pub / sub system without the _publishing_ part. This means that you can only subscribe to database events, but not publish them. Publishing is done implicitly during a database save operation (such as `DbContext.SaveChanges()`). This is always the case when the main application context `SmartDbContext` commits data, because it derives from [HookingDbContext](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Data/HookingDbContext.cs), which contains all the hooking logic.

{% hint style="warning" %}
Bypassing EF and accessing the database directly with raw SQL means: no events and no hooking!
{% endhint %}

Each hook has a **PreSave** __ and a **PostSave** handler. They are run for each entity in the EF Change Tracker BEFORE and AFTER saving respectively.

The **PreSave** handler’s purpose is:

* Validating an entity.
* Fix or enrich an entity.
* Change an entity’s state (e.g. to suppress save).
* Check which properties have been modified (which is not possible in **PostSave** handlers).

The **PostSave** handler’s purpose is to perform an action using a _definitely_ saved entity.

{% hint style="warning" %}
Snapshot comparisons aren’t possible in the **PostSave** stage.
{% endhint %}

## Implementing hooks

Create a concrete class that either:

* implements [IDbSaveHook](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Data/Hooks/IDbSaveHook.cs)
* implements `IDbSaveHook<TContext>` to bind it to a particular `DbContext` type.
* derives from `Smartstore.Core.Data.AsyncDbSaveHook<TEntity>` to bind it to the main `SmartDbContext` type and given `TEntity` type.
* derives from [Smartstore.Data.Hooks.AsyncDbSaveHook\<TContext, TEntity>](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Data/Hooks/AsyncDbSaveHook.cs) to bind it to a particular `DbContext` and given `TEntity` type.

{% hint style="info" %}
The abstract base classes are nothing special, they just implement `IDbSaveHook` to make your life easier. There are sync counterparts for the base classes with sync method signatures also.
{% endhint %}

{% hint style="info" %}
When a hook is bound to entity type `TEntity`, it matches all stored entities equal to or subclasses of `TEntity`.
{% endhint %}

There is no need to register a hook in DI, because it is automatically detected and registered as a scoped service when the application starts. This allows a hook type to take any dependency.

{% hint style="info" %}
You can also apply the above interface / base classes to any existing service class..
{% endhint %}

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
    /// "HookResult.Ok": signals the hook handler that it should continue 
    /// to call this method for the current EntityType/State/Stage combination,
    /// "HookResult.Void": signals the hook handler that it should stop 
    /// executing this method for the current EntityType/State/Stage combination.
    /// </returns>
    Task<HookResult> OnBeforeSaveAsync(
        IHookedEntity entry, 
        CancellationToken cancelToken);

    /// <summary>
    /// Called after an entity has been successfully saved.
    /// </summary>
    /// <param name="entry">The entity entry</param>
    /// <returns>
    /// "HookResult.Ok": signals the hook handler that it should continue 
    /// to call this method for the current EntityType/State/Stage combination,
    /// "HookResult.Void": signals the hook handler that it should stop 
    /// executing this method for the current EntityType/State/Stage combination.
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

A hook method always returns a `HookResult` type.

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

#### Optimizing performance

For performance reasons it is essential that you return `HookResult.Void` if the current entity type / state / stage combination is of no interest to the hook and thus will not be handled.

{% hint style="info" %}
Instead of returning `HookResult.Void`, you can also throw `NotImplementedException` or `NotSupportedException`.
{% endhint %}

This way you signal the hooking framework that it should stop executing the hook for the given combination in successive save operations. This is a kind of _filter_ that reduces the number of classes that must be instantiated repeatedly, only to find out that there's nothing to do.

Here is an example to illustrate:

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

`IHookedEntity` is passed to the hook handler method. It represents the entity entry that is being hooked and has the following properties:

| Property          | Description                                                                                                                                                                                                                                                                                                     |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Entry`           | The underlying EF entity entry.                                                                                                                                                                                                                                                                                 |
| `Entity`          | Instance of the hooked entity.                                                                                                                                                                                                                                                                                  |
| `State`           | The **current** entity state.                                                                                                                                                                                                                                                                                   |
| `InitialState`    | The entity state before the save operation. Use this in **PostSave** handlers.                                                                                                                                                                                                                                  |
| `HasStateChanged` | Indicates whether the entity state has been changed in a **PreSave** handler.                                                                                                                                                                                                                                   |
| `IsSoftDeleted`   | Indicates whether the entity is in _soft deleted_ state. This is the case if the entity is an instance of `ISoftDeletable` and the value of its `Deleted` property is `true` **AND** it has changed since being tracked. However, if the entity is not in _modified_ state, the snapshot comparison is omitted. |

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
* This is done by wrapping a [dbcontextscope.md](../advanced/data-access-deep-dive/dbcontextscope.md "mention") around a unit of work
* To customize your hook's importance, decorate your hook class with `ImportantAttribute`:
  * `Normal` (default): Hook can be ignored during long running processes, e.g. imports. Usually simple hooks that invalidate cache entries or clean some resources.
  * `Important`: Hook is important and should also run during long running processes. Not running the hook **may** result in stale or invalid data.
  * `Essential`: Hook instance should run in any case (e.g. _AuditHook_, which is even required during installation). Not running the hook **will definitely** result in stale data or raise exceptions.

### Specify execution order

* Default execution order is 0.
* Decorate your hook class with [OrderAttribute](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Modularity/OrderAttribute.cs) and pass an integer value.

### Mark entities as unhookable

* Some entity types should not be hooked at all, e.g. the [Log](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Logging/Domain/Log.cs) entity.
* Decorate your entity class with `HookableAttribute` and pass _false_.
* The hooking framework will never pass these entities to any hook

## Some Tipps

* Most hooks are just cache invalidators
* Separating cache invalidation from cache access makes code somehow confusing
* Therefore we recommend to combine them in a single class
* _SAMPLE_ (pattern see: [DiscountService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Catalog/Discounts/Services/DiscountService.cs), [ProductTagService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Catalog/Products/Services/ProductTagService.cs), [CurrencyService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Common/Services/CurrencyService.cs), [DeliveryTimeService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Common/Services/DeliveryTimeService.cs) etc.)
