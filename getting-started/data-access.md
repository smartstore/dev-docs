---
description: Getting started to access the application database
---

# 🥚 Data access

## Overview

Every store is represented by a single database, even when using the multi-store option. Each table is represented by a special entity type derived from the `BaseEntity` type. Smartstore currently supports **Microsoft SQL Server**, **MySQL**, **PostgreSQL** and **SQLite** database systems and uses the [Entity Framework Core (EF)](https://learn.microsoft.com/en-us/ef/core/) Object-relational-Mapper (O/R-Mapper) to access the store database.

* It enables .NET developers to work with a database using .NET objects.
* Most of the data access code that would normally be written can be omitted.
* Write provider-agnostic data access code, regardless of the underlying database provider.

The main gateway to the application database is the `SmartDbContext`, representing the **Unit of Work**. The context keeps track of everything you do during a request / transaction that affects the database. It figures out everything that needs to be done to alter the database as a result of your work. Properties and extension methods that expose **repositories** to interact with specific database tables are also defined by it.

## SmartDbContext

The main EF `DbContext` implementation for the application database is the `SmarDbContext`. It is registered as a scoped dependency in the DI container, and as such its type can easily be passed around as a dependency to get resolved.

```csharp
public class MyService : IMyService
{
    private readonly SmartDbContext _db;

    public MyService(SmartDbContext db)
    {
        _db = db;
    }

    public Task<List<MyEntity>> GetMyEntities()
    {
        return _db.Set<MyEntity>().ToListAsync();
    }
}
```

{% hint style="info" %}
It is good practice to name the data context parameter `db`, or `_db` if used in a field. Keep it simple and short!
{% endhint %}

## Pooled factory

A `SmartDbContext` instance is not directly resolved from DI container, but from the pooled `IDbContextFactory<SmartDbContext>` which is configured and created on app start-up. The factory, registered as a singleton, is responsible for creating and pooling the `SmartDbContext` instances.

An instance created by the factory is returned to the pool upon disposal and is reset to its initial state. Every time a `SmartDbContext` instance is requested, the pool will return an already existing unleased instance instead of creating a new one - or will create a new instance if the pool is depleted. Therefore, the `SmartDbContext` DI registration is actually a delegate that leases an instance from this pool, not from the DI container.

Pooling is very beneficial for performance in high-load scenarios, because the factory prevents too many objects from being instantiated.

{% hint style="info" %}
By default, the pool size is set to **1024** instances and can be altered via `appsettings.json` using the `DbContextPoolSize` setting.
{% endhint %}

In some situations, it may be necessary to manually lease a context instance from the pool. Two scenarios in which this occurs are:

* You cannot pass the `SmartDbContext` as a dependency in singleton objects, because of its [scope](dependency-injection.md).
* You need granular control of your unit of work.

In these cases, you might want to do one of the following:

```csharp
public class MySingletonService : IMySingletonService 
{
    private IDbContextFactory<SmartDbContext> _dbFactory;

    public MySingletonService(IDbContextFactory<SmartDbContext> dbFactory)
    {
        // IDbContextFactory<T> is singleton and can safely 
        // be passed to other singleton dependencies.
        _dbFactory = dbFactory;
    }

    public Task<List<MyEntity>> GetMyEntities()
    {
        // Lease context instance from pool
        using (var db = _dbFactory.CreateDbContext())
        {
            // do something in this "unit of work"
        } // --> Dispose: return context instance to pool
    }
}
```

## Paging

* PagedList
* FastPager

## Second-Level cache

* Where queried entities are cached in memory
* So that subsequent queries to the same entities serve the result from cache without accessing the database
  * Which makes queries faster because: no database workload, no record to class materialization... the already materialized entities are put to cache.
* A cache entry always contains the result of a query, so it either contains a single entity or a list of entities
  * The key of the entry is the unique hash of the query. Even the slightest query variation leads to a different hash, therefore to a new cache entry.
* Whenever an entity that came from cache is updated or deleted, all cache entries that contain the entity are invalidated automatically
  * Next query execution accesses the database then
* But: not all entity types are cacheable... only those that are annotated with the `CacheableEntity` attribute. This attribute also defines the caching policy, like how long to cache the entry (default is 3 hours) and a max rows limit (query results with more items than the given number will not be cached)
* Only those entity types that do not change frequently are marked as cacheable, and those that potentially do not produce large database tables. Like: `Country`, `StateProvince`, `Currency`, `Language`, `Store`, `TaxCategory`, `DeliveryTime`, `QuantityUnit`, `EmailAccount` etc.
* To activate caching on a per query basis:
  * If the queried entity type is annotated with `CacheableEntity` attribute: call `AsNoTracking()` for the query. Because only untracked entities will be cached.
    * _SAMPLE_
  * If the queried entity type is **not** annotated with `CacheableEntity` attribute: call `AsNoTracking()` and `AsCaching()` for the query.
    * _SAMPLE_
* To explicitly deactivate caching on a per query basis:
  * Call `AsNoCaching()` for the query

## DataProvider

* Lorem ipsum

## Conventions & best practices

* Lorem ipsum

## Deep Dive

To learn more about data access in Smartstore, read [data-access-deep-dive](../framework/advanced/data-access-deep-dive/ "mention").
