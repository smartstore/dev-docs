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
* similar to MVC filters in the way they behave

Hooks let you focus on the aspect you want to solve without ever touching the core of the app. They are extremely powerful and flexible when it comes to composable, granular application design. Smartstore relies heavily on hooks. Without them:

* Granular and isolated application design would be nearly impossible.
* Modules would be much less flexible.

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

Each hook has a **PreSave** and a **PostSave** handler. They are called for each entity in the EF Change Tracker. **PreSave** is called BEFORE saving, then the actual save operation is performed, after which **PostSave** is called.

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

You can check for modified properties If an entity is in the _Modified_ state. But because the row snapshot is reset after saving, this only happens in a **PreSave** handler.

| Method                                                                        | Return                                                                                                                                                                                                                                 |
| ----------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `IsPropertyModified(string propertyName)`                                     | A value indicating whether the given property has been modified.                                                                                                                                                                       |
| `Entry.GetModifiedProperties()`                                               | A dictionary filled with modified properties for the specified entity. The key is the name of the modified property and the value is its ORIGINAL value, which was tracked when the entity was attached to the context the first time. |
| `Entry.TryGetModifiedProperty(string propertyName, out object originalValue)` | The property value if the given property has been modified, or null if not.                                                                                                                                                            |

### Abstract base class

For more convenience the abstract class [DbSaveHook](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Data/Hooks/DbSaveHook.cs) provides six overridable methods: `OnInserting`, `OnUpdating`, `OnDeleting`, `OnInserted`, `OnUpdated`, `OnDeleted`.

They all return `HookResult.Void` by default and just need to be overridden to opt-in.

* _SAMPLE_ (with some explanatory code comments)

### Batching

Sometimes it may be preferable to hook the entire save batch instead of hooking entities one at a time: for example, if your hook executes some expensive code. Consider the following scenario:

You have an import process that always saves product entities in batches of 100 products each. A **PostSave** product hook handler would be called 100 times for the save operation `OnAfterSaveAsync`. But the batch handler `OnAfterSaveCompletedAsync` would be called only once. All entities that have gone through `OnAfterSaveAsync` before are passed to this method as a collection.

{% hint style="warning" %}
All entities that were handled with `HookResult.Void` in `OnAfterSaveAsync` are excluded from the entries collection, because they are _not of interest_.
{% endhint %}

* _SAMPLE_

### Setting priorities

The `Importance` attribute of a hook specifies its priority of execution. For performance reasons, some callers may reduce the number of hooks executed by specifying the setting `MinHookImportance` for certain units of work.

E.g., the product import task, which is a long-running process, turns off the execution of `Normal` hooks by changing `MinHookImportance` to `Important`.

This is done by wrapping a DbContextScope around a unit of work. To customize your hook's importance, decorate your hook class with `ImportantAttribute`.

| Value              | Description                                                                                                                                                                        |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Normal` (default) | The hook can be ignored during long running processes, such as imports. These usually are simple hooks that invalidate cache entries or clean up some resources.                   |
| `Important`        | The hook is important and should be running, even during long running processes. Not running the hook **may** result in stale or invalid data.                                     |
| `Essential`        | The hook instance should always run (e.g. _AuditHook_, which is even required during installation). Not running the hook will definitely result in stale data or throw exceptions. |

### Specify execution order

The `Order` attribute of a hook determines the order in which hooks appear in the calling queue. Hooks with lower order values are called before hooks with higher ones. To set the order value, decorate your hook class with [OrderAttribute](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Modularity/OrderAttribute.cs) and pass an integer value, the default being `0`.

### Mark entities as unhookable

Some entity types should not be hooked at all, like the [Log](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Logging/Domain/Log.cs) entity. To make sure that the hooking framework will never pass these entities to any hook, decorate your entity class with `HookableAttribute` and pass `false`.

## Some Tipps

Most hooks just invalidate cache. Separating cache invalidation from cache access makes the code a bit confusing. Therefore, we recommend combining them into a single class:

* _SAMPLE_ (pattern see: [DiscountService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Catalog/Discounts/Services/DiscountService.cs), [ProductTagService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Catalog/Products/Services/ProductTagService.cs), [CurrencyService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Common/Services/CurrencyService.cs), [DeliveryTimeService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Common/Services/DeliveryTimeService.cs) etc.)
