# 👍 Caching

## Overview

* Cache is used to store application data to speed things up
* Crucial for better performance
* Static/Singleton cache is used for durable objects that should live as long as the app lives (or for a specified custom duration)
* Request cache is used for objects that should be removed when the request ends
* Static cache
  * [ICacheManager](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Caching/ICacheManager.cs): a composite multi-level cache manager
  * By default, accesses `IMemoryCache` under the hood
  * But provides a unified API for both memory and distributed cache
  * If a distributed cache provider is installed (like REDIS), it is accessed in the same way as the memory cache
    * This is different from how .NET Core handles cache access, because it exposes two different APIs: `IMemoryCache` and `IDistributedCache`.
    * But Smartstore unifies the API because of the multi-level character (read further below)
* Request cache
  * [IRequestCache](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Caching/IRequestCache.cs): By default, accesses `HttpContext.Items` dictionary under the hood.
  * If no `HttpContext` is present, a local dictionary is created instead.

## Application cache

### Multi-level cache

* `ICacheManager` is a container for multiple cache stores represented by [ICacheStore](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Caching/ICacheStore.cs).
* Every CRUD method traverses all registered stores to find, update or delete items (memory stores come first, then distributed).
* By default, only one store exists: [MemoryCacheStore](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Caching/MemoryCacheStore.cs).
* But modules may provide new stores. Like the REDIS module which provides the `RedisCacheStore`.
* The multi-level cache design guarantees:
  * Unified API for both memory and distributed store
  * **Performance**! A memory store is MUCH faster than a distributed store. `ICacheManager` will always query memory store first and will only fallback to distributed store if an item does not exist in memory. After retrieval from distributed store, the item is also put in memory store, so subsequent reads return the object which is in memory.
  * `IMessageBus` is responsible for syncing all memory stores across web-farm nodes:
    * If server B caused the deletion of an item in the distributed store, a notification will be sent to server A to delete the item from its memory store as well.
* WARN: Because of the unified API, you must be careful when it comes to object types. Your cached objects must be serializable by JSON (_Newtonsoft_, not _System.Text.Json_). You must ensure that your object:
  * Has a public parameterless constructor
  * Does not have circular references
  * Does not produce a large object graph
  * Does not contain property types that are not serializable
  * Is **not** an entity type deriving from `BaseEntity`. Never do that, really... it's dangerous.
* To resolve the primary memory or distributed store explicitly (without relying on the composite multi-level manager): use [ICacheFactory](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Caching/ICacheFactory.cs):&#x20;
  * `GetMemoryCache()`: resolves an `ICacheManager` instance that only interacts with the current `IMemoryCacheStore` implementation
  * `GetDistributedCache()`: resolves an `ICacheManager` instance that only interacts with the current `IDistributedCacheStore` implementation. If no distributed store exists, a memory store manager will be returned instead.

#### Implementing a custom cache store

* To implement a custom cache store create a class that implements [ICacheStore](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Caching/ICacheStore.cs).
* Follow the contract and implement all members
* Register your store impl in service container:

```csharp
internal class Startup : StarterBase
{
    public override void ConfigureContainer(ContainerBuilder builder, IApplicationContext appContext)
    {
        builder.RegisterType<MyDistributedCacheStore>()
            .As<ICacheStore>()
            .As<IDistributedCacheStore>() // OR as IMemoryCacheStore
            .SingleInstance();
    }
}
```

### Accessing the cache

* Accessing the static application cache in Smartstore is fairly easy.
* Just get yourself the `ICacheManager` dependency (which is a singleton):

```csharp
ICacheManager cache;

return cache.GetAsync("MyCacheItemKey", async options =>
{
    // Set the entry absolute expiration relative to now
    options.ExpiresIn(TimeSpan.FromHours(2));
    
    // ... prepare some model
    var model = await PrepareSomeModelAsync();
    
    return model;
});
```

* All calls are thread-safe and may be used concurrently from multiple threads.
* `ICacheManager` provides several methods for reading, adding or updating cache items, as described in the following table.

| To do this                                                                                                        | Use this method                                    |
| ----------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| Get an item if it exists                                                                                          | `Get<T>(string, bool)`                             |
| Try to get an item if it exists                                                                                   | `TryGet(string, out T)`                            |
| Get an item, adding it by calling the passed acquirer function if doesn't exist                                   | `Get<T>(string, Func<CacheEntryOptions, T>, bool)` |
| Get or create a provider specific hashset. If key does not exist, create a new set and put to cache automatically | `GetHashSet(string, Func<IEnumerable>)`            |
| Add an item, or update if it exists                                                                               | `Put(string, object, CacheEntryOptions)`           |
| Remove an item it if exists                                                                                       | `Remove(string)`                                   |
| Remove many items by key pattern                                                                                  | `RemoveByPattern(string)`                          |
| Enumerate all existing item keys by optionally passing a key matching glob pattern                                | `Keys(string)`                                     |
| Set an item's absolute expiration                                                                                 | `SetTimeToLive(string, TimeSpan?)`                 |
| Get a lock object for the given key used to synchronize access to the underlying cache storage                    | `GetLock(string)`                                  |

* NOTE: Nearly all cache access methods have async overloads.

### Glob patterns

* Some cache methods support glob patterns in order to match keys. Supported patterns are:
  * **h?llo** matches _hello_, hallo and _hxllo_
  * **h\*llo** matches _hllo_ and _heeeello_
  * **h\[ae]llo** matches _hello_ and _hallo_, but not _hillo_
  * **h\[^e]llo** matches _hallo_, _hbllo_, ... but not _hello_
  * **h\[a-b]llo** matches _hallo_ and _hbllo_

### Expiration policy

* By default, every item put to cache has infinite lifetime. To limit the lifetime you can configure _absolute_ or _sliding_ expiration per item.

```csharp
ICacheManager cache;

return cache.Get("A", o =>
{
    // Set the entry absolute expiration relative to now
    o.ExpiresIn(TimeSpan.FromHours(2));
    
    // How long an entry can be inactive (e.g. not accessed) before
    // it will be removed. This will not extend the entry lifetime 
    // beyond the absolute expiration (if set)
    o.SetSlidingExpiration(TimeSpan.FromHours(1));
    
    // ... prepare some data and return
    return model;
});

// - OR, alternative to "CacheEntryOptions.ExpiresIn()" -
cache.SetTimeToLive(TimeSpan.FromHours(2));
```

### Item dependencies

* By default, every item that is removed from cache automatically invalidates items from parent closures. Example:

```csharp
ICacheManager cache;

return cache.Get("A", () =>
{
    // If "B" is removed from cache or updated at some point in the future,
    // "A" will also be removed, because "A" depends on "B"
    var model = cache.Get("B", o =>
    {
        // ... prepare some data
    });
    
    return model;
});
```

* To disable this hierarchy dependency chain, pass `independent` parameter to the `Get` method and set its value to `true.` In this case no attempt will be made to invalidate parent cache entries.
* To specify custom dependency keys for an item, use `CacheEntryOptions.DependsOn(string[])` method:

```csharp
ICacheManager cache;

return cache.Get("A", o =>
{
    // Remove "A" from cache whenever "B", "C" or "D" are removed or updated.
    o.DependsOn("B", "C", "D");
    
    // ... prepare some data and return
    return model;
});
```

* Lock recursion (?) --> _maybe too complicated to explain_

## Request cache

* Just get yourself the `IRequestCache` dependency (which is request scoped):

```csharp
IRequestCache requestCache;

return requestCache.Get("A", () =>
{
    // ... prepare some data and return
    return model;
});
```
