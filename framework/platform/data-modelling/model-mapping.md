---
description: Object mapping made easy
---

# 🥚 Model mapping

## Overview

* Smartstore comes with ultra-lightweight and fast object mapping utilities
* Automatic object mapping comes handy whenever you need to convert from type A to B, e.g. an entity type (like `Product`) to a view model type (like `ProductModel`) or vice-versa.
* Instead of manually assigning all members one by one, an object mapper does this in a very generic way using reflection behind the scenes, thus reducing the amount of code massively: it is just a one-liner.
* [MiniMapper](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/ComponentModel/MiniMapper.cs): for very simple object mapping
* [MapperFactory](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/ComponentModel/MapperFactory.cs): for more advanced and flexible mapping

## MiniMapper

* A lightweight and simple object mapper utility which tries to map properties **of the same name** between two objects using reflection.
* It uses [FastProperty](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/ComponentModel/FastProperty.cs) under the hood to query and access properties, so it's really fast!
* If matched properties have different types, the mapper tries to convert them using the [type conversion system](../../advanced/type-conversion.md).
* If conversion fails, the property is skipped (no exception is thrown)
* Use `MiniMapper` when source and destination type look roughly the same and you don't need full control over mapping.
* WARN: `MiniMapper` cannot deep-copy collection types.
* WARM: `MiniMapper` does **not** clone reference types.

### Usage

```csharp
BlogSettings settings = /* get instance somehow */;

// Maps BlogSettings --> BlogSettingsModel (must have a parameterless constructor)
// and returns the destination instance
var model = MiniMapper.Map<BlogSettings, BlogSettingsModel>(settings);

// Maps BlogSettings --> BlogSettingsModel by populating the given target instance.
var model = new BlogSettingsModel();
MiniMapper.Map(settings, model);
```

## MapperFactory

* A static factory that can create custom type mapper instances ([IMapper\<TFrom, TTo>](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/ComponentModel/IMapper.cs))
* Use for more control over mapping
* To resolve a mapper instance from the factory, call `MapperFactory.GetMapper<TFrom, To>()`.
* To map object instances, call one of the `Map*()` methods (the corresponding mapper is resolved internally in this case). Yo can also pass dynamic parameters to the map method (kind of mapping options).
* `MapperFactory` automatically scans for all concrete `IMapper<TFrom, TTo>` classes in all loaded assemblies upon initialization
* A mapper is DI-enabled and therefore can depend on any registered service.
* If no mapper is found for a specific mapping operation, then a generic mapper is used which internally delegates object mapping to `MiniMapper` (see above)

### Implementing a mapper

* Create a class that implements `IMapper<TFrom, TTo>`
* There is nothing wrong with implementing multiple interfaces, e.g. `IMapper<News, NewsModel>` **and** `IMapper<NewsModel, News>` in a single class
* No need to register in DI
* TIPP: it is good practice to keep model and mapper classes in the same file.

<pre class="language-csharp"><code class="lang-csharp">public class NewsMapper :
    IMapper&#x3C;News, NewsModel>,
    IMapper&#x3C;NewsModel, News>
{
    public NewsItemMapper()
    {
         // Pass dependencies if required
    }

    public Task MapAsync(News from, NewsModel to, dynamic parameters = null)
    {
        // ... map News --> NewsModel ...

<strong>        return Task.CompletedTask;
</strong>    }

    public Task MapAsync(NewsModel from, News to, dynamic parameters = null)
    {
        // ... map NewsModel --> News ...

        return Task.CompletedTask;
    }
}
</code></pre>

### Resolve & map

```csharp
News entity = /* get entity instance somehow */;
NewsModel model = new NewsModel();

// Get the mapper = an instance of "NewsMapper" above,
// because "NewsMapper" implements "IMapper<News, NewsModel>"
var mapper = MapperFactory.GetMapper<News, NewsModel>();

// Map News --> NewsModel
await mapper.MapAsync(entity, model);
// - OR, short way without resolving the mapper first -
await MapperFactory.MapAsync(entity, model);

// Map News --> NewsModel with parameters
await mapper.MapAsync(entity, model, new { Option1 = true, Option2 = "stuff" });
```
