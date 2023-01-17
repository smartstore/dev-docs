---
description: Entities and O/R Mapping
---

# 🍪 Domain

## Overview

The domain tier contains all entity classes that are mapped to database tables. Some of the most used classes are:

* [Product](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Catalog/Products/Domain/Product.cs)
* [Category](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Catalog/Categories/Domain/Category.cs)
* [Manufacturer](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Catalog/Brands/Domain/Manufacturer.cs)
* [Order](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Checkout/Orders/Domain/Order.cs)
* [ShoppingCart](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Checkout/Cart/Domain/ShoppingCart.cs)
* [Customer](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Identity/Domain/Customer.cs)
* [MediaFile](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Content/Media/Domain/MediaFile.cs)

An entity class is a "Plain old CLR object" ([POCO](https://en.wikipedia.org/wiki/Plain\_old\_CLR\_object)). It usually represents one database table, with each property typically representing one column in the table.

{% hint style="info" %}
You can establish any relationship between entities (1:1, 1:n, n:m)
{% endhint %}

## BaseEntity

A concrete entity class **must** derive from the abstract class [BaseEntity](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Domain/BaseEntity.cs). By convention:

* `BaseEntity.Id` is used as the primary key.
* All public properties with a getter and a setter will be included in the database schema.

To customize the default mapping conventions:

* Use the _DataAnnotation_ attributes on classes and properties.
* Write _Fluent API_ mapping code.

{% hint style="info" %}
Refer to [Entity Framework Core: Creating and configuring a model](https://learn.microsoft.com/en-us/ef/core/modeling/) to learn more about data modelling in _Entity Framework_.
{% endhint %}

{% hint style="success" %}
It may seem a bit cluttered at first glance, but it is good practice to keep entity class and _Fluent API_ mapping in a single code file.
{% endhint %}

For performance reasons, Smartstore does not use proxy classes for lazy loading, but the `ILazyLoader` interface instead. Because of this an entity class requires two constructors as long as the entity contains at least one [navigation property](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/navigation-property).

Special consideration is required for navigation properties:

```csharp
public class MyEntity : BaseEntity 
{
    // Default constructor
    public MyEntity()
    {
    }
    
    // Constructor called by EF internally to enable 
    // lazy loading for navigation properties
    private MyEntity(ILazyLoader lazyLoader)
        : base(lazyLoader)
    {
    }
    
    // 1:1 navigation property
    private Product _product;
    public Product Product
    {
        // The "LazyLoader" property is declared in "BaseEntity" class
        get => _product ?? LazyLoader?.Load(this, ref _products);
        set => _product = value;
    }
    
    // 1:n navigation property
    private ICollection<Category> _categories;
    public ICollection<Category> Categories
    {
        // The "LazyLoader" property is declared in "BaseEntity" class
        get => LazyLoader?.Load(this, ref _categories) ?? (_categories ??= new HashSet<Category>());
        // "protected" --> prevent assignment
        protected set => _categories = value;
    }
}
```

## Domain assemblies

On app start-up Smartstore scans all application core assemblies for entity types. If you created a new entity class and want to make it accessible, there are different approaches depending on the project type.

In a core project simply extend the partial `SmartDbContext` class.

```csharp
public partial class SmartDbContext
{
    public DbSet<MyEntity> MyEntities { get; set; }
}
```

In a module project you need to inform Entity Framework that your module assembly contains entity types to pickup and register on start-up.

For this you need a starter class.

```csharp
internal class Startup : StarterBase
{
    public override void ConfigureServices(IServiceCollection services, IApplicationContext appContext)
    {
        services.AddTransient<IDbContextConfigurationSource<SmartDbContext>, SmartDbContextConfigurer>();
    }

    class SmartDbContextConfigurer : IDbContextConfigurationSource<SmartDbContext>
    {
        public void Configure(IServiceProvider services, DbContextOptionsBuilder builder)
        {
            builder.UseDbFactory(b =>
            {
                b.AddModelAssembly(this.GetType().Assembly);
            });
        }
    }
}
```

For more comfort while developing you can add this extension:

{% code title="Extensions/SmartDbContextExtensions.cs" %}
```csharp
public static class SmartDbContextExtensions
{
    public static DbSet<MyEntity> MyEntities(this SmartDbContext db)
        => db.Set<MyEntity>();
}
```
{% endcode %}

## Entity characteristics

Numerous interfaces can be implemented and some abstract base classes can be derived from to specify what an entity _has_/_is_, _supports_ or _represents_.

The following types declare some properties your entity must implement in order to follow the contract.

### Interfaces

| Name                 | Characteristic                           |
| -------------------- | ---------------------------------------- |
| **IAclRestricted**   | Has access restrictions                  |
| **IAttributeAware**  | Has some raw attributes                  |
| **IAuditable**       | Has auditing properties                  |
| **IDiscountable**    | Has applicable discounts                 |
| **IDisplayedEntity** | Is displayed somehow in UI               |
| **IDisplayOrder**    | Is orderable                             |
| **ILocalizedEntity** | Is localizable                           |
| **INamedEntity**     | Has a conceptual name                    |
| **IMergedData**      | Has mergeable data                       |
| **IPagingOptions**   | Has data paging options (page size etc.) |
| **IRulesContainer**  | Is a container for other rules           |
| **ISlugSupported**   | Supports SEO slugs                       |
| **ISoftDeletable**   | Is soft deletable                        |
| **IStoreRestricted** | Supports store mapping                   |
| **ITransient**       | Supports transiency                      |

### Abstract base classes

| Name                     | Characteristic                                                                                              |
| ------------------------ | ----------------------------------------------------------------------------------------------------------- |
| **EntityWithAttributes** | Has generic attributes (see [generic-attributes.md](../framework/advanced/generic-attributes.md "mention")) |
| **EntityWithDiscounts**  | Has applicable discounts                                                                                    |
| **CustomerContent**      | Represents content entered by customer                                                                      |
