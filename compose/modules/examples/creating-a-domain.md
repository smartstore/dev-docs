# 🥚 Creating a Domain

Domains offer a way to add your own tables to SmartStore's database. In this tutorial you will write a simple notification system for your shop.

## Preparing the table

### Table layout

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

### Create the Domain

The `Domain` is the model equivalent of your database table. You must specify the table name and the indexes you need as attributes.

```csharp
// Define your table name
[Table("TableNameInDatabase")]

// Define a property as an index (IX means Index)
[Index(nameof(PropertyName), Name = "IX_ClassName_PropertyName")]
```

{% hint style="info" %}
Defining a column as an index optimises and therefore speeds up database lookups.
{% endhint %}

After that you simply add your columns as you would properties to a model.

```csharp
public int columnA { get; set; }

public bool columnB { get; set; } = true;
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
    
    public string Message { get; set; } = string.Empty;
}
```
{% endcode %}

This creates the table `HelloWorld_Notifications` with the three columns: `AuthorId`, `Published` and `Message`. Because a lot of times you are going to be looking up notifications based on either `AuthorId` or `Published`, they are defined as indexes.

### Create the Migration

In order to add the `HelloWorld_Notifications` table to SmartStore's database, you need to create a migration. This will also be used when migrating from one Version to the next. The methods needed for this are `Up()` and `Down()` for migrating to the next version or the previous version of your table. In this tutorial we are only going to use the `Up` method.

Create the folder _Migrations_ and add the migration class, which name includes the current date, _YYYYMMDDHHMMSS\_Initial.cs_. With the following attribute added to the class respectively:

```csharp
// Use the current date & time [YYYY-MM-DD HH:MM:SS]
[MigrationVersion("2022-12-14 10:34:22", "HelloWorld: Initial")]
```

{% code title="20221214103422_Initial.cs" %}
```csharp
// Some code
```
{% endcode %}

## Providing table access

## Adding a View

## Advanced topics

### Schedule a Task

Schedule a Task for cleaning up the table. Removing old unneccessary notifications from your table will increase database speed.

#### Add configuration

#### Create a Task

#### Install / Uninstall a Task

## Conclusion

In this tutorial you learned how to:

* Add a table to SmartStore's database
* Migrate it from previous versions
* Extend SmartDbContext to include your tables
* Add Cleanup Tasks

The code for this module can be downloaded here:

\<Insert Files>
