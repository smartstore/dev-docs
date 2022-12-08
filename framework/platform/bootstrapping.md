---
description: Start and initialize the application
---

# ✔ Bootstrapping

## Overview

Bootstrapping is the process of starting and initializing the application. This can take some time (something between 2 and 10 seconds), depending on the quality of the hardware and the number of modules loaded. When the application is shut down, the very first request usually triggers the application startup.

During the application startup, the following actions are taking place:

* All **core assemblies** are loaded into the application domain
* All **installed module assemblies** are discovered and loaded into the application domain
* All **services** are registered in the DI service container
* The HTTP request **pipeline** is configured
* Route **endpoints** are mapped

In a conventional ASP.NET Core application, all of these actions are performed in `Program.cs` (or `Startup.cs` in previous versions of ASP.NET), but this is not an option for Smartstore because external modules need to hook into the bootstrapping process. This is where modular **** _Starters_ come into play.

## Modular starters

The application core contains only a very slim bootstrapper (kind of a kernel). After all module assemblies are loaded into the application domain, the type scanner scans for concrete subclasses of the [IStarter](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Engine/Builders/IStarter.cs) interface in all assemblies. The starters are sorted and executed successively.

{% hint style="info" %}
Each project can have any number of starter classes or none. There are no restrictions at all.
{% endhint %}

### IStarter interface

Here is the definition of the `IStarter` interface:

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

In Smartstore, the [StarterBase](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Engine/Builders/StarterBase.cs) abstract class is used for more comfort. It implements the `IStarter` interface with virtual overridable methods, so your starter should be derived from this class and not from the `IStarter` interface.&#x20;

Besides the  `ConfigureServices` method, `StarterBase` class also provides the `ConfigureContainer` overridable method. The latter does the same as the former, but in the `Autofac` way (using `ContainerBuilder` instead of `IServiceCollection`). You can override both of them, one or none, doesn't matter at all.

{% hint style="info" %}
Following the convention, we place the `Startup.cs` file in the root of a module project, name the class `Startup`, derive it from `StarterBase` abstract class and make it _internal_.
{% endhint %}

### Conditional execution

For conditional execution of the starter, we override the `StarterBase.Matches()` method and return a value indicating whether the starter should be executed or be skipped. This can be done in our module where we explicitly allow or suppress starter execution based on the application installation state for the instance, as in the following example:

```csharp
internal class Startup : StarterBase
{
    // Should NOT run when app is not fully installed yet
    public override bool Matches(IApplicationContext appContext)
        => appContext.IsInstalled;
}
```

### Order of execution

By default, starters are executed in the order they were discovered by the type scanner (first core assemblies, then module assemblies, and so on). This is because, by default, in the `StarterBase` class the `IStarter.Order` property is assigned the `Default` value from the [`StarterOrdering`](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Engine/Builders/StarterOrdering.cs) static class. However, this value can be overridden in your starter implementation. If there are 2 starters with the same `Order` value and it is necessary to explicitly specify the order of execution, the `StarterBase.RunAfter()` method is used. Here is an example of such implementation:

```csharp
internal class Startup : StarterBase
{
    public override int Order => StarterOrdering.BeforeStaticFilesMiddleware;

    public Startup()
    {
        RunAfter<MyFirstModule>();
    }
} 
```

### Middleware and Endpoint ordering

Sometimes even precise starter ordering is not enough. Middleware and endpoints require a bit more control, for example, you need to be able to **precisely** define the order of a middleware component _within_ the request pipeline. Imagine you have developed two middleware components in a single module. One must come `BeforeStaticFilesMiddleware` and the other one `AfterRoutingMiddleware`. In Smartstore, you can accomplish this in the following way:

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

The static class [StarterOrdering](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Engine/Builders/StarterOrdering.cs) comes very handy here, because it defines numerous constants that represent the order of well-known middleware components (such as StaticFiles, Routing, Authentication, ExceptionHandler etc.). You just need to hook-in before or after something.

### Startup class full implementation example

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
        // decorators, adapters, metadata, Lazy<>, container, modules etc.
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

Implementations of [IApplicationInitializer](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Engine/Initialization/IApplicationInitializer.cs) are used to execute application initialization code during the very **first HTTP request** and **very early** in the request lifecycle. This distinguishes them from starters that are executed earlier (before `HttpContext` is initialized).

But some initialization logic simply needs a valid scope, like `HttpContext`, to resolve services from, since you just can't access scoped or transient dependencies in a starter (well, you could, but that's a very bad idea and pure evil 😄), unless you spawn a custom dependency scope. We won't cover that topic here, as there is plenty of material on the subject online.

By default, an initializer is executed only once unless you specify a higher value in the `MaxAttempts` property. However, this setting has no effect if `ThrowOnError` is set to _true._ The`ThrowOnError` property indicates whether to throw any error and stop execution of subsequent initializers. If the value is _false_, the initializer will be executed and `OnFailAsync` is invoked to give you the chance to do some logging or fix things.

{% hint style="info" %}
There is no need to register an initializer in the DI, as all types implementing `IApplicationInitializer` are auto-discovered and resolved during application initialization. Thus, initializers can take any dependency.
{% endhint %}

### Smartstore built-in initializers

* [ApplicationDatabasesInitializer](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Data/Bootstrapping/ApplicationDatabasesInitializer.cs) is the very first initializer to run and does exactly what its name says, it initializes the application database(s)&#x20;
* [TaskSchedulerInitializer](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Scheduling/Bootstrapping/TaskSchedulerInitializer.cs) activates the web scheduler after checking for valid hostnames, or returns a warning if no scheduler or store is registered
* [InstallPermissionsInitializer](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Security/Bootstrapping/InstallPermissionsInitializer.cs) checks for new permission records and seeds them
* [ModulesInitializer](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Modularity/ModulesInitializer.cs) discovers and refreshes changed module locale resources among other things
