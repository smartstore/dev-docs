---
description: Entities and O/R Mapping
---

# 👍 Domain

## Overview

* Domain tier contains all entity classes that are mapped to database tables
* Among the most frequently used classes are:
  * [Product](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Catalog/Products/Domain/Product.cs)
  * [Category](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Catalog/Categories/Domain/Category.cs)
  * [Manufacturer](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Catalog/Brands/Domain/Manufacturer.cs)
  * [Order](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Checkout/Orders/Domain/Order.cs)
  * [ShoppingCart](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Checkout/Cart/Domain/ShoppingCart.cs)
  * [Customer](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Identity/Domain/Customer.cs)
  * [MediaFile](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Content/Media/Domain/MediaFile.cs)
  * and many many more...
* Entity classes are "Plain old CLR objects" (POCO)
* Each entity class usually represents one database table
* Each entity property usually represents one column in this table
* You can establish any relationship between entities (1:1, 1:n, n:m)

## BaseEntity

* A concrete entity class **must** derive from the abstract [BaseEntity](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Domain/BaseEntity.cs) class
* `BaseEntity.Id` is PK by convention
* By convention, all public properties with a getter and a setter will be included in the database schema. To customize the default mapping conventions:
  * Use _DataAnnotation_ attributes on class and property level...
  * Write _Fluent API_ mapping code...
  * ...or mix both.
  * Refer to [Entity Framework Core: Creating and configuring a model](https://learn.microsoft.com/en-us/ef/core/modeling/) to learn more about data modelling in EF.
  * TIPP: It may seem a bit cluttered at first glance, but it is good practice to keep entity class and Fluent API mapping in a single code file
* For performance reasons, Smartstore does not use proxy classes for lazy loading, but the `ILazyLoader` instead.
* Because of this an entity class requires two constructors (but only if the entity contains at least one navigation property)...
* ...and special consideration is required for navigation properties:

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

* By default, Smartstore scans all application core assemblies for entity types on app startup.
* So, if you created a new entity class in a core project, all you need to do is to extend the partial `SmartDbContext` class to make it accessible:

```csharp
public partial class SmartDbContext
{
    public DbSet<MyEntity> MyEntities { get; set; }
}
```

* If you however created a new entity class in a module project, there are more steps involved.
* First you need to inform EF that your module assembly contains entity types that it should pickup and register on startup. This is done in a starter class:

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

* The next step is optional but it gives you more comfort while developing:

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

* There are numerous interfaces you can implement and some abstract base classes you can derive from to specify what an entity _has/is, supports_ or _represents_.
* These types declare some properties that your entity must implement in order to follow the contract

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
