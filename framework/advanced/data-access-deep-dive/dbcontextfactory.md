# 🐣 Pooled DbContext factory

In general, `DbContext` is a lightweight object: creating and disposing it doesn't involve a database operation, and most applications can do this without noticeable performance impact. However, each context instance sets up various internal services and objects necessary to perform its tasks, and the overhead of continuously doing so may be significant in high-performance scenarios. For these cases, EF Core can _pool_ your context instances:

* When you dispose your context, EF Core resets its state and stores it in an internal pool.
* The next time a new instance is requested, the pooled instance is returned instead of setting up a new one.

Context pooling allows you to pay context setup costs only once at program start-up, rather than continuously. Which is why Smartstore configures a pooled `DbContext` factory on start-up. When a context instance is requested from the factory it does one of the following:

* Look up a free / idle instance in pool and return it.
* Create an instance, return it and put back into the pool on return / dispose.

By default, the maximum number of instances retained by the pool (pool size) is set to **1024** and can be altered via `appsettings.json` using the `DbContextPoolSize` setting. Once the pool size is exceeded, new context instances aren’t cached and EF reverts to non-pooling behavior, creating instances as needed.

Smartstore registers a scoped service factory for the `SmartDbContext` service type, which internally resolves an instance of `SmartDbContext` from the pool. This instance is then returned to the pool when the request completes. So in general you don’t need to call the `CreateDbContext` method of the `IDbContextFactory<SmartDbContext>`.

However, there may be situations where working with `IDbContextFactory` is beneficial:

* If your code does not run within the scope of an HTTP request like the [SmartDbContextSink](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Logging/Serilog/SmartDbContextSink.cs) that resolves a `SmartDbContext` instance periodically, triggered by a timer.
  * EXAMPLE/CODE EXCERPT
* If you need to access `SmartDbContext` inside a singleton class like the [SettingFactory](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Configuration/Services/SettingFactory.cs).
  * EXAMPLE/CODE EXCERPT
* In very long-running processes that load or write a lot of entities. This can gradually decrease performance, because the change tracker tracks too many entities. In this case it may be beneficial to resolve a fresh instance from pool, instead of detaching all entities, after a batch completes.
  * EXAMPLE
