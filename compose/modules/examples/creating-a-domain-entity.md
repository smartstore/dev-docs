# 🥚 Creating a Domain entity

Domain entities offer a way to add your own tables to Smartstore's database. In this tutorial you will add a simple notification system to your [Hello World](../tutorials/building-a-simple-hello-world-module.md) module.

{% hint style="info" %}
Here we are working in modules, but Domain entities can also be added to the core.
{% endhint %}

## Preparing the table

### Table schema

A simple notification will need the following properties:

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

The `Domain` is the model equivalent of your database table. You can specify the table name and the indexes you need as attributes.

```csharp
// Define your table name. By convention, the entity name is used.
[Table("TableNameInDatabase")]

// Define a property as an index
[Index(nameof(PropertyName), Name = "IX_ClassName_PropertyName")]
```

After that you simply add your columns as you would properties to a model.

```csharp
public int ColumnA { get; set; }

public bool ColumnB { get; set; } = true;
```

Add the file _Notification.cs_ to the new folder _Domain_.

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

In order to add the `Notification` table to Smartstore's database, you need to create a migration. In this tutorial you are only going to be overriding the `Up` method from the abstract `MigrationBase` class.

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

## Providing table access

Now that you have the table set up, you need to give your module access to it. To access the table from a `SmartDbContext` instance you need to add the following two files:

* _Startup.cs_ in the root of your folder.
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

Now you can access your table from the `SmartDbContext`.

```csharp
var messages = await _db.Notifications()
    .Where(x => x.Message.Length > 0)
    .FirstOrDefaultAsync();
```

{% hint style="info" %}
Adding the `SmartDbContext` extension is not required for using the migration.
{% endhint %}

## Adding a View

Add configuration view

* show for number of days after publishing - DaysToShowNotification
* Add 'new notification' button
  * AuthorID: CurrentCustomer.Id
  * Message: user input
  * Published: now()

Add widget to display notifications.

## Further ideas

You will find the following ideas included in the module code:

Schedule a Task for cleaning up the table. This will increase database speed, by removing old unneccessary notifications from your table.

## Conclusion

In this tutorial you learned how to:

* Add a table to SmartStore's database
* Create a migration for it
* Extend SmartDbContext to include your tables

The code for this module can be downloaded here:

\<Insert Files>
