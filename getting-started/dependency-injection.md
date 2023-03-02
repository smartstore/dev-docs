# Dependency injection

## Overview

Smartstore uses Autofac and the inversion of control (IoC) concept to resolve dependencies of components. Autofac is an IoC container that manages the dependencies between classes, so that Smartstore stay easy to change as it grow in size and complexity.

> "The idea behind inversion of control is that, rather than tie the classes in your application together and let classes "new up" their dependencies, you switch it around so dependencies are instead passed in during class construction."\
> — _Autofac_

A simple output writer example in the [Autofac](https://autofac.readthedocs.io/en/latest/getting-started/index.html#structuring-the-application) documentation illustrates the basic idea behind IoC and de-coupled architecture very well.

## Registering services

In order for the dependencies to be resolved, the related service must be registered.

## Resolving services

### Scope and Lifetime

After a service has been registered it can be resolved from the IoC container or from child lifetime scopes.

> "While it is possible to resolve components right from the root container, doing this through your application in some cases may result in a memory leak. It is recommended you always resolve components from a lifetime scope where possible to make sure service instances are properly disposed and garbage collected."\
> — _Autofac_

ILifetimeScope, IServiceProvider...

### Constructor injection

Constructor injection is the preferred way to resolve dependencies. Your component must be registered to use it.

### Property injection

While constructor injection is the preferred method of passing dependencies to a component being constructed, you can also use the `PropertiesAutowired` method to let Autofac auto inject properties.

HINT: Smartstore recommends to avoid property injection if possible and to use it only for special cases (like abstract classes) or very simple services (like `Logger` or `Localizer`). Auto injected properties must be public, although in most cases a component dependency should not be.

### "Work of" dependency

Sometimes a dependency needs to be resolved late, when it is used for the first time, rather than when the component's constructor is called (e.g. when it is called at a very early stage when the IoC container cannot yet resolve services). A solution for this is the [Work](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Engine/Work.cs) class. The dependency is resolved from `ILifetimeScope` when its `Value` property is accessed for the first time.

```csharp
public class MyComponent
{
    private readonly Work<ILanguageService> _languageService;

    public MyComponent(Work<ILanguageService> languageService)
    {
        // ILanguageService not resolved yet.
        _languageService = Guard.NotNull(languageService);
    }
    
    public void Process()
    {
        // ILanguageService now resolved via "Value" property.
        var languages = _languageService.Value.GetAllLanguages();
    }
}
```
