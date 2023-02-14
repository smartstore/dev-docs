# 🥚 Creating a Domain entity

Domain entities offer a way to add your own tables to Smartstore's database. In this tutorial you will add a simple notification system to your [Hello World](../tutorials/building-a-simple-hello-world-module.md) module.

{% hint style="info" %}
This tutorial describes working in _modules_, but Domain entities can also be added to the _core_.
{% endhint %}

## Preparing the table

### Table schema

A simple notification could have the following properties:

* A unique identifier (`Id`, type: number)
* An author, represented by the customer id (`AuthorId`, type: number)
* A timestamp (`Published`, type: date-time)
* A message (`Message`, type: string)

Here is what that the table could look like:

| Id | Author | Published                   | Messaage                           |
| -- | ------ | --------------------------- | ---------------------------------- |
| 1  | 543    | 2022-12-12 11:48:08.9937258 | Hello World!                       |
| 2  | 481    | 2022-12-13 19:02:55.7695421 | What a wonderful day it is :smile: |

### Create the Domain entity

#### Overview

The `Domain` object is an abstract data structure that has all the properties of the entity it describes. The _Entity Framework_ (ORM Mapper) automates the projection between `Domain` objects and database tables.

Specify the table name and the indexes using `DataAnnotations` and add the properties that represent your database columns.

```csharp
// Outside the class.
// Specify your table name. By convention, the entity name is used.
[Table("TableNameInDatabase")]

// Declare a property to be used as an index.
[Index(nameof(PropertyName), Name = "IX_ClassName_PropertyName")]

// Inside the class.
// Define some columns.
public int ColumnA { get; set; }

public bool ColumnB { get; set; } = true;
```

#### Implementation

Add the file _Notification.cs_ to the new _Domain_ directory and do the following steps:

* Specify the table name `Notification`.
* Declare `AuthorId` and `Published` as indexes.
* Implement the abstract `BaseEntity` class.
* Add the properties for `AuthorId`, `Published` and `Message`.

{% hint style="info" %}
There is no need to declare an Identifier, because implementing the abstract `BaseEntity` class adds an `Id` entry automatically.
{% endhint %}

Your `Notification` class should look something like this:

{% code title="Notification.cs" %}
```csharp
[Table("Notification")]
[Index(nameof(AuthorId), Name = "IX_Notification_AuthorId")]
[Index(nameof(Published), Name = "IX_Notification_Published")]
public class Notification : BaseEntity
{
    public int AuthorId { get; set; }
    
    public DateTime Published { get; set; }
    
    public string Message { get; set; }
}
```
{% endcode %}

This represents the `Notification` table with the three columns: `AuthorId`, `Published` and `Message`. Because a lot of times you are going to be looking up notifications based on either `AuthorId` or `Published`, they are defined as indexes.

### Create the Migration

In order to add the `Notification` table to Smartstore's database, you need to create a migration. The migration framework creates the table on application start-up. In this tutorial you are only going to be overriding the `Up` method from the abstract `MigrationBase` class.

{% hint style="info" %}
To learn more, please refer to [Migrations](../../../framework/platform/database-migrations.md).
{% endhint %}

Create the folder _Migrations_ and add the migration class, which name includes the current date, _YYYYMMDDHHMMSS\_Initial.cs_. With the following attribute added to the class respectively:

```csharp
// Use the current date & time [YYYY-MM-DD HH:MM:SS]
[MigrationVersion("2022-12-14 10:34:22", "HelloWorld: Initial")]
```

The class needs to inherit from the `Migration` base class to have access to the SQL database methods like `Create`, `Remove` or `Update`. With these you are now able to check if the table already exists, and if not, create it.

```csharp
if (!Schema.Table("Notification").Exists())
{
    Create.Table("Notification");
}
```

To add columns, specify indexes and primary keys, you can simply daisy chain the following _FluentMigrator_ methods:

| Method         | Description                                         |
| -------------- | --------------------------------------------------- |
| `WithColumn`   | Defines a new column.                               |
| `WithIdColumn` | Defines an id column, that acts as the primary key. |

Columns can also have a type (Boolean, Integer, String, Date, Currency, ...) and be specified as (not) nullable, unique, a primary key, indexed, etc. With this information you can build the `Notification` table.

```csharp
Create.Table("Notification")
    .WithIdColumn()
    .WithColumn(nameof(Notification.AuthorId))
        .AsInt32()
        .NotNullable()
        .Indexed("IX_Notification_AuthorId")
    .WithColumn(nameof(Notification.Published))
        .AsDateTime2()
        .NotNullable()
        .Indexed("IX_Notification_Published")
    .WithColumn(nameof(Notification.Message))
        .AsString()
        .NotNullable();
```

{% hint style="info" %}
It's best to use the `DateTime2` type instead of `DateTime`. It has an extended range and higher precision.
{% endhint %}

The class should look like this:

{% code title="20221214103422_Initial.cs" %}
```csharp
[MigrationVersion("2022-12-14 10:34:22", "HelloWorld: Initial")]
public class _20221214103422_Initial : Migration
{
    public override void Up()
    {
        // Tablename is taken from Domain->Attribute->Table
        var tableName = "Notification";
        
        if (!Schema.Table(tableName).Exists())
        {
            Create.Table(tableName)
                .WithIdColumn() // Adds the Id column, which defaults to primary key.
                .WithColumn(nameof(Notification.AuthorId))
                    .AsInt32()
                    .NotNullable()
                    .Indexed("IX_Notification_AuthorId")
                .WithColumn(nameof(Notification.Published))
                    .AsDateTime2()
                    .NotNullable()
                    .Indexed("IX_Notification_Published")
                .WithColumn(nameof(Notification.Message))
                    .AsString()
                    .NotNullable();
        }
    }

    public override void Down()
    {
        // Ignore this for now.
    }
}
```
{% endcode %}

## Providing table access in modules

Now that you have the table set up, you need to give your module access to it. To access the table from a `SmartDbContext` instance you need to add the following two files:

* _Startup.cs_ in the root of your directory
* _SmartDbContextExtensions.cs_ in the _Extensions_ directory (needs to be created)

Create the static `SmartDbContextExtension` class and add the following method:

```csharp
public static DbSet<Notification> Notifications(this SmartDbContext db)
    => db.Set<Notification>();
```

The Startup class inherits from the `StarterBase` class and includes the following lines:

{% code title="Startup.cs" %}
```csharp
public override void ConfigureServices(IServiceCollection services, IApplicationContext appContext)
{
    services.AddTransient<IDbContextConfigurationSource<SmartDbContext>, SmartDbContextConfigurer>();
}

private class SmartDbContextConfigurer : IDbContextConfigurationSource<SmartDbContext>
{
    public void Configure(IServiceProvider s, DbContextOptionsBuilder builder)
    {
        builder.UseDbFactory(b => 
        {
            b.AddModelAssembly(GetType().Assembly);
        });
    }
}
```
{% endcode %}

Now you can access the entities stored in your table using the `SmartDbContext`.

```csharp
var messages = await _db.Notifications()
    .Where(x => x.Message.Length > 0)
    .FirstOrDefaultAsync();
```

{% hint style="info" %}
Adding the `SmartDbContext` extension is not required for using the migration.
{% endhint %}

## Next steps

You will find the following steps included in the module code:

1. Add a [configuration view](../tutorials/building-a-simple-hello-world-module.md#adding-configuration). Configure the number of days a notification is shown.
2. Add a 'New notification' button. Let the current admin create a message.
3. Display the notification [using a widget](creating-a-widget-provider.md). This way you can position it freely around the shop.
4. Schedule a Task for cleaning up the table. This will increase database speed, by removing old unneccessary notifications from your table.

### Further ideas

There are many more things you could do with this module:

* Add a _Moderator_ `CustomerRole` and separate view so certain users can create notifications.
* Add categories to the notification. Show certain notifications on different pages.
* Make your entities accessible via the [Web API](../../../framework/web-api/web-api-in-detail.md#web-api-and-modules).

## Conclusion

In this tutorial you learned how to:

* Add a table to Smartstore's database
* Create a migration for it
* Extend SmartDbContext to include your tables

The code for this module can be downloaded here:

\<Insert Files>
