---
description: Getting started to access the application database
---

# 👎 Data access

## Overview

* Every store is represented by a single database
* Smartstore currently supports **Microsoft SQL Server** and **MySQL** database systems and uses the [Entity Framework Core (EF)](https://learn.microsoft.com/en-us/ef/core/) O/R-Mapper to access the store database:
  * It enables .NET developers to work with a database using .NET objects.
  * It eliminates the need for most of the data-access code that typically needs to be written.
  * Because EF abstracts the way a database is accessed, we can write provider-agnostic data access code that just works, regardless of the underlying database provider
* The main gateway to the application database is `SmartDbContext`:
  * It represents the database **Unit of Work**: The context keeps track of everything you do during a request or transaction that can affect the database. When you're done, it figures out everything that needs to be done to alter the database as a result of your work.
  * It defines properties and extensions methods that expose the **Repositories** to interact with particular database tables. Each table is represented by a special entity type that derives from the `BaseEntity` type.

## SmartDbContext

* Is the main EF `DbContext` implementation for the application database
* `SmartDbContext` is registered as a scoped dependency in DI container.
* To resolve an instance of `SmartDbContext` just pass its type around as dependency:

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

* TIPP: It is good practice to name the data context parameter `db`**,** or `_db` if field. Simple and short!

## Pooled factory

* Actually a `SmartDbContext` instance is not directly resolved from DI container, but from the pooled `IDbContextFactory<SmartDbContext>` which is configured and created on app startup. The factory (registered as singleton) is responsible for creating and pooling `SmartDbContext` instances.
* An instance created by the factory is returned to the pool upon disposal, and not really disposed (but reset to initial state)
* Every time a `SmartDbContext` instance is requested, the pool will return an already existing unleased instance instead of creating a new one - or will create a new instance if pool is depleted.
* In high load scenarios pooling is very advantageous when it comes to performance because the factory prevents itself from instantiating too many objects.
* Therefore the `SmartDbContext` DI registration is actually a delegate that leases an instance from this pool, and not from the DI container.
* By default, the pool size is set to **1024** instances.
  * NOTE: The default pool size can be altered via `appsettings.json` --> `DbContextPoolSize` setting.
* In some situations it may be required to lease a context instance from the pool manually. For example, in singleton objects you cannot pass `SmartDbContext` as dependency because it is scoped (refer to [dependency-injection.md](dependency-injection.md "mention") to learn more about DI and scopes). Or you need granular control about your unit of work.
* In this case you could do:

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
