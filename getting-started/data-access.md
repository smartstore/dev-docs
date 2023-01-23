---
description: Getting started to access the application database
---

# 🥚 Data access

## Overview

Every store is represented by a single database, even when using the multi-store option. Each table is represented by a special entity type derived from the `BaseEntity` type. Smartstore currently supports **Microsoft SQL Server** and **MySQL** database systems and uses the [Entity Framework Core (EF)](https://learn.microsoft.com/en-us/ef/core/) Object-relational-Mapper (O/R-Mapper) to access the store database.

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

An instance created by the factory is returned to the pool upon disposal and is reset to its initial state. The pool returns an existing unused instance each time a `SmartDbContext` instance is requested, instead of creating a new one. In case of a depleted pool, a new instance will be created. Therefore, the `SmartDbContext` DI registration is a delegate that leases an instance from this pool, not from the DI container.

Pooling is very beneficial for performance in high-load scenarios, because the factory prevents too many objects from being instantiated. By default, the pool size is set to **1024** instances and can be altered via `appsettings.json` using the `DbContextPoolSize` setting.

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

* Lorem ipsum

## DataProvider

* Lorem ipsum

## Conventions & best practices

* Lorem ipsum

## Deep Dive

To learn more about data access in Smartstore, read [data-access-deep-dive](../framework/advanced/data-access-deep-dive/ "mention").
