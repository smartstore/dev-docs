# 🥚 Pooled DbContext factory

A `DbContext` is generally a light object: creating and disposing one doesn't involve a database operation, and most applications can do so without any noticeable impact on performance. However, each context instance does set up various internal services and objects necessary for performing its duties, and the overhead of continuously doing so may be significant in high-performance scenarios. For these cases, EF Core can _pool_ your context instances: when you dispose your context, EF Core resets its state and stores it in an internal pool; when a new instance is next requested, that pooled instance is returned instead of setting up a new one. Context pooling allows you to pay context setup costs only once at program startup, rather than continuously.

* Smartstore configures a pooled DbContext factory at app start
* When a context instance is requested from the factory
  * Factory looks up a free/idle instance in pool and returns it, or...
  * Creates an instance, returns it and puts it to pool on return/dispose
  * The max number of instances retained by the pool (pool size) can be configured via _appsettings.json_: `Smartstore.DbContextPoolSize`
    * Which is 1024 by default
    * Once pool size is exceeded, new context instances are not cached and EF falls back to the non-pooling behavior of creating instances on demand.
* Smartstore registers a scoped service factory for the service type `SmartDbContext`.&#x20;
* This factory resolves an instance of `SmartDbContext` from the pool internally
  * This instance is then returned to the pool when request ends
* So, in general, there's no need for you to call the `CreateDbContext` method of `IDbContextFactory<SmartDbContext>`.
* However, there may be situations where working with `IDbContextFactory` offers benefits
  * If your code does not run within an HTTP request scope
    * e.g.: [SmartDbContextSink](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Logging/Serilog/SmartDbContextSink.cs) resolves a `SmartDbContext` instance periodically (triggered by a timer)
    * _EXAMPLE/CODE EXCERPT_
  * If you need access to `SmartDbContext` within a singleton class
    * e.g.: [SettingFactory](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Configuration/Services/SettingFactory.cs) is singleton and needs to access the database
    * _EXAMPLE/CODE EXCERPT_
  * In very long running processes where a lot of entities are loaded or written
    * Which can gradually decrease performance (because of the change tracker tracking too many entities)
    * In this case it may be beneficial to resolve a fresh instance from pool after a batch completes (instead of detaching all entities)
    * _EXAMPLE_
