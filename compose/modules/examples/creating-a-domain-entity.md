# 🥚 Creating a Domain entity

Domain entities offer a way to add your own tables to Smartstore's database. In this tutorial you will write a simple notification system for your shop.

## Preparing the table

### Table schema

A simple notification will need the following properties:

* A unique identifier (`Id`, type: number)
* An author, represented by the customer id (`AuthorId`, type: number)
* A timestamp (`Published`, type: date-time)
* A message (`Message`, type: string)

Here is what that table could look like:

| Id | Author | Published                   | Messaage                           |
| -- | ------ | --------------------------- | ---------------------------------- |
| 0  | 543    | 2022-12-12 11:48:08.9937258 | Hello World!                       |
| 1  | 481    | 2022-12-13 19:02:55.7695421 | What a wonderful day it is :smile: |

### Create the Domain entity

The `Domain` is the model equivalent of your database table. You must specify the table name and the indexes you need as attributes.

```csharp
// Define your table name
[Table("TableNameInDatabase")]

// Define a property as an index
[Index(nameof(PropertyName), Name = "IX_ClassName_PropertyName")]
```

After that you simply add your columns as you would properties to a model.

```csharp
public int ColumnA { get; set; }

public bool ColumnB { get; set; } = true;
```

Add the file _HelloWorldNotification.cs_ to the new folder _Domain_.

{% code title="HelloWorldNotification.cs" %}
```csharp
[Table("HelloWorld_Notifications")]
[Index(nameof(AuthorId), Name = "IX_HelloWorldNotification_AuthorId")]
[Index(nameof(Published), Name = "IX_HelloWorldNotification_Published")]
public class HelloWorldNotification : BaseEntity
{
    public int AuthorId { get; set; }
    
    public DateTime Published { get; set; }
    
    public string Message { get; set; }
}
```
{% endcode %}

This creates the table `HelloWorld_Notifications` with the three columns: `AuthorId`, `Published` and `Message`. Because a lot of times you are going to be looking up notifications based on either `AuthorId` or `Published`, they are defined as indexes.

### Create the Migration

In order to add the `HelloWorld_Notifications` table to Smartstore's database, you need to create a migration. This will also be used when migrating from one version to the next. The methods needed for this are `Up()` and `Down()` for migrating to the next version or the previous version of your table. In this tutorial we are only going to use the `Up` method.

Create the folder _Migrations_ and add the migration class, which name includes the current date, _YYYYMMDDHHMMSS\_Initial.cs_. With the following attribute added to the class respectively:

```csharp
// Use the current date & time [YYYY-MM-DD HH:MM:SS]
[MigrationVersion("2022-12-14 10:34:22", "HelloWorld: Initial")]
```

The class needs to implement the `Migration` interface, to have access to SQL Database methods like `Create`, `Remove` or `Update`. With these you are now able to check if the table already exists, and if not, create it.

```csharp
if (!Schema.Table("HelloWorld_Notifications").Exists())
{
    Create.Table("HelloWorld_Notifications");
}
```

To add columns, specify indexes and primary keys, you can simply daisy chain the following `FluentMigrator` methods:

| Method            | Description                                         |
| ----------------- | --------------------------------------------------- |
| `InSchema`        | Defines the tables schema.                          |
| `WithColumn`      | Defines a new column.                               |
| `WithIdColumn`    | Defines an id column, that acts as the primary key. |
| `WithDescription` | Adds the table's description.                       |

Columns can furthermore be:

* given a type: Boolean, Integer (16, 32, 64), Binary, String, Date, Currency, ...
* specified as: (not) Nullable, Unique, a primary key, indexed, ...

With this information you can build the `HelloWorld_Notifications` table.

```csharp
Create.Table("HelloWorld_Notifications")
    .WithIdColumn()
    .WithColumn(nameof(HelloWorldNotification.AuthorId)).AsInt32().NotNullable()
        .Indexed("IX_HelloWorldNotification_AuthorId")
    .WithColumn(nameof(HelloWorldNotification.Published)).AsDateTime2().NotNullable()
        .Indexed("IX_HelloWorldNotification_Published")
    .WithColumn(nameof(HelloWorldNotification.Message)).AsString().NotNullable();
```

{% hint style="info" %}
It's best to use the `DateTime2` type instead of `DateTime`. It has an extended range and higher precision.
{% endhint %}

The class should look like the following:

{% code title="20221214103422_Initial.cs" %}
```csharp
[MigrationVersion("2022-12-14 10:34:22", "HelloWorld: Initial")]
public class _20221214103422_Initial : Migration
{
    public override void Up()
    {
        // Tablename is taken from Domain->Attribute->Table
        if (!Schema.Table("HelloWorld_Notifications").Exists())
        {
            Create.Table("HelloWorld_Notifications")
                .WithIdColumn() // Adds the Id column, which defaults to primary key.
                .WithColumn(nameof(HelloWorldNotification.AuthorId)).AsInt32().NotNullable()
                    .Indexed("IX_HelloWorldNotification_AuthorId")
                .WithColumn(nameof(HelloWorldNotification.Published)).AsDateTime2().NotNullable()
                    .Indexed("IX_HelloWorldNotification_Published")
                .WithColumn(nameof(HelloWorldNotification.Message)).AsString().NotNullable();
        }
    }

    public override void Down()
    {
        // Ignore this.
    }
}
```
{% endcode %}

## Providing table access

Now that you have the table set up, you need to give your module access to it. To access the table from the usual `_db` Context you need to add the following two files:

* _Startup.cs_ in the root of your folder.
* _SmartDbContextExtensions.cs_ in the _Extensions_ folder (needs to be created)

Create the `static` SmartDbContextExtension class and add the following method:

```csharp
public static DbSet<HelloWorldNotification> HelloWorldNotifications(this SmartDbContext db)
    => db.Set<HelloWorldNotification>();
```

The Startup class implements the `StarterBase` interface and includes the following lines:

{% code title="Startup.cs" %}
```csharp
public override void ConfigureServices(IServiceCollection services, IApplicationContext appContext){
    services.AddTransient<IDbContextConfigurationSource<SmartDbContext>, SmartDbContextConfigurer>();
}
private class SmartDbContextConfigurer : IDbContextConfigurationSource<SmartDbContext>{
    public void Configure(IServiceProvider services, DbContextOptionsBuilder builder){
        builder.UseDbFactory(b => {
            b.AddModelAssembly(GetType().Assembly);
        });
    }
}
```
{% endcode %}

Now you can access your table from the `SmartDbContext`.

```csharp
var messages = await _db.HelloWorldNotifications()
    .Where(x => x.Message.Length > 0)
    .FirstOrDefaultAsync();
```

## Adding a View

## Advanced topics

### Schedule a Task

Schedule a Task for cleaning up the table. Removing old unneccessary notifications from your table will increase database speed.

#### Add configuration

Create a setting `DaysToKeepNotification` in your configuration. This defines the number of days a notification is kept, before removing it.

```csharp
public int DaysToKeepNotification { get; set; } = 10;
```

#### Create a Task

Add a folder named _Tasks_ to your root and create the _DeleteOldNotificationsTask_ class, that implements the `ITask` interface.

```csharp
public async Task Run(TaskExecutionContext ctx, CancellationToken cancelToken = default){
    var date = DateTime.UtcNow.AddDays(_settings.DaysToKeepNotification);

    var notifications = await _db.HelloWorldNotifications()
        .Where(x => x.Published < date)
	.ToArrayAsync();

    _db.HelloWorldNotifications().RemoveRange(notifications);

    await _db.SaveChangesAsync();
}
```

#### Install / Uninstall a Task

Add to _Module.cs_:

```csharp
// In the Install method:
await _taskStore.GetOrAddTaskAsync<DeleteOldNotificationsTask>(x =>
{
	x.Name = T("Plugins.MyOrg.HelloWorld.DeleteOldNotificationsTask");
	x.CronExpression = "0 12 * * ?";
	x.Enabled = true;
});

// In the Uninstall method:
await _taskStore.TryDeleteTaskAsync<DeleteOldNotificationsTask>();
```

## Conclusion

In this tutorial you learned how to:

* Add a table to SmartStore's database
* Migrate it from previous versions
* Extend SmartDbContext to include your tables
* Add Cleanup Tasks

The code for this module can be downloaded here:

\<Insert Files>
