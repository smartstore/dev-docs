---
description: Start and initialize application
---

# 👍 Bootstrapping

## Overview

* The process of starting and initializing the application
* May take some time, something between 2 and 10 sec., depending on hardware quality and amount of modules loaded
* Usually - if app is shut down - the very first request triggers the application start
* During application start
  * All core assemblies are loaded into the app domain
  * All installed module assemblies are discovered and loaded into the app domain
  * All services are registered in the DI service container
  * The HTTP request pipeline is configured
  * Route endpoints are mapped
* In a conventional ASP.NET Core application all this is performed in `Program.cs` (or `Startup.cs` in previous versions of ASP.NET)
* But this is not an option for Smartstore, because external modules need to hook into the bootstrapping process
* This is where modular _Starters_ enters the stage

## Modular starters

* The app core only contains a very slim bootstrapper, kind of a kernel
* After all module assemblies has been loaded into app domain, the type scanner scans for concrete subclasses of [IStarter](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Engine/Builders/IStarter.cs) in all assemblies.
* The starters are sorted and executed successively
* INFO: any project can have any number of starter classes, or none. No restrictions.

### IStarter interface

```csharp
public interface IStarter : ITopologicSortable<string>
{
    int Order { get; }

    // Allow or suppress starter execution based on some 
    // conditions like app installation state for instance
    bool Matches(IApplicationContext appContext);

    // Add services to the container
    void ConfigureServices(IServiceCollection services, IApplicationContext appContext);

    // Configure MVC services
    void ConfigureMvc(IMvcBuilder mvcBuilder, IServiceCollection services, IApplicationContext appContext);
    
    // Configure the application's request pipeline with precise middleware ordering.
    void BuildPipeline(RequestPipelineBuilder builder);
    
    // Register endpoint routes
    void MapRoutes(EndpointRoutingBuilder builder);
}
```

### StarterBase abstract class

* For more comfort
* Implements `IStarter` with virtual overridable methods
* Your starter should derive from this, not from `IStarter`
* Besides `ConfigureServices`, `StarterBase` also provides the overridable `ConfigureContainer` method. The latter does the same as the first, but in the `Autofac`-way (with `ContainerBuilder` passed instead of `IServiceCollection`).
* You can override both, one or none... doesn't matter.
* INFO: By convention, we place the file `Startup.cs` in the root of a module projects, name the class `Starter` and make it _internal_.

### Conditional execution

* Override the `StarterBase.Matches()` method and return a value indicating whether the starter should run or be skipped.
* _SAMPLE_

### Order of execution

* By default, starters are executed in the order they were discovered by the type scanner  (core assemblies first, then module assemblies etc.)
* But: `IStarter.Order`, ...
* And: `StarterBase.RunAfter()`
* ...give you control about the order your starter

### Middleware and Endpoint ordering

* But sometimes even precise starter ordering is not enough
* Middleware and endpoints require some more control
* For example, you must be able to **precisely** define the order of a middleware component _within_ the request pipeline
* Imagine you have developed two middleware components in a single module. One must come _before_ the `Static Files` middleware and the other one _after_ the `Routing` middleware.
* In Smartstore, you can accomplish this in the following way:

```csharp
internal class Startup : StarterBase
{
    public override void BuildPipeline(RequestPipelineBuilder builder)
    {
        builder.Configure(StarterOrdering.BeforeStaticFilesMiddleware, app =>
        {
            app.UseMiddleware<MyFirstMiddleware>();
        });
    
        builder.Configure(StarterOrdering.AfterRoutingMiddleware, app =>
        {
            app.UseMiddleware<MySecondMiddleware>();
        });
    }
}
```

* The [StarterOrdering](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Engine/Builders/StarterOrdering.cs) static class comes very handy here, because it defines numerous constants that represent the order of well-known middleware components (like StaticFiles, Routing, Authentication, ExceptionHandler etc.).&#x20;
* You just have to hook-in before or after something.

### Example

{% code title="Starter.cs" %}
```csharp
internal class Startup : StarterBase
{
    public Startup() 
    {
        RunAfter<MvcStarter>();
    }
    
    // Should NOT run when app is not fully installed yet
    public override bool Matches(IApplicationContext appContext)
        => appContext.IsInstalled;
    
    public override void ConfigureServices(
        IServiceCollection services, 
        IApplicationContext appContext)
    {
        // Override "ConfigureServices" for things that only
        // ASP.NET DI can do, like option configuration
        services.Configure<SomeOptions>(o => 
        {
            // ... configure options
        });
    }
    
    public override void ConfigureMvc(
        IMvcBuilder mvcBuilder, 
        IServiceCollection services, 
        IApplicationContext appContext)
    {
        // MVC configuration could be done in "ConfigureServices",
        // but isn't this well organized? :-)
        
        // ... configure some MVC stuff
    }
    
    public override void ConfigureContainer(
        ContainerBuilder builder, 
        IApplicationContext appContext)
    {
        // You can't do this in "ConfigureServices", because
        // ASP.NET DI does not support registration sources,
        // decorators, adapters, metadata, Lazy<>, container etc. modules etc.
        builder.RegisterSource(new MyAutofacRegistrationSource());
    }
    
    public override void BuildPipeline(RequestPipelineBuilder builder)
    {
        builder.Configure(StarterOrdering.BeforeStaticFilesMiddleware, app =>
        {
            app.UseMiddleware<MyFirstMiddleware>();
        });
    
        builder.Configure(StarterOrdering.AfterRoutingMiddleware, app =>
        {
            app.UseMiddleware<MySecondMiddleware>();
        });
    }
    
    public override void MapRoutes(EndpointRoutingBuilder builder)
    {
        if (builder.ApplicationContext.IsInstalled)
        {
            builder.MapRoutes(StarterOrdering.LateRoute, endpoints =>
            {
                endpoints.MapBlazorHub();
            });
        };
    }
}
```
{% endcode %}

## Initializers

* Implementations of [IApplicationInitializer](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Engine/Initialization/IApplicationInitializer.cs) are used to execute application initialization code during the very **first HTTP request** and **very early** in the request lifecycle
* This distinguishes them from starters that run earlier (before `HttpContext` is initialized)
* But some initialization logic simply needs a valid scope - like `HttpContext` - to resolve services from
  * You just can't access scoped or transient dependencies in a starter, unless you spawn a custom dependency scope...
  * Well, you could. But it is a VERY bad idea and pure evil :smile:
* What you can do with initializers:
  * [TaskSchedulerInitializer](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Scheduling/Bootstrapping/TaskSchedulerInitializer.cs) is a good example: it activates the web scheduler after testing for valid host names, or...
  * [InstallPermissionsInitializer](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Security/Bootstrapping/InstallPermissionsInitializer.cs): checks for new permission records and seeds them
  * [ModulesInitializer](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Modularity/ModulesInitializer.cs): among other things discovers and refreshes changed module locale resources
* By default, an initializer is executed only once, unless you specify a higher value in `MaxAttempts` property. But this setting has no effect if `ThrowOnError` is _true._
* `ThrowOnError`: whether to throw any error and stop executing subsequent initializers. If this is _false_, the initializer will be executed and `OnFailAsync` will be invoked to give you the chance to do some logging or fix things.
* No need to register in DI, all types implementing `IApplicationInitializer` are auto-discovered and resolved on app initialization. So: initializers can take any dependency.
