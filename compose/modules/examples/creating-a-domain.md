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

// Define a property as an index for quick lookups (IX means Index)
[Index(nameof(PropertyName), Name = "IX_ClassName_PropertyName")]
```

Add a file _HelloWorldNotification.cs_ to the new folder _Domain_.

{% code title="HelloWorldNotification.cs" %}
```csharp
[Table("HelloWorld_Notifications")]
[Index(nameof(AuthorId), Name = "IX_HelloWorldNotification_AuthorId")]
[Index(nameof(Published), Name = "IX_HelloWorldNotification_Published")]
public class HelloWorldNotification : BaseEntity
{
    public int AuthorId { get; set; }
    public DateTime Published { get; set; }
    public string Message { get; set; } = string.Empty
}
```
{% endcode %}

As you can see, `AuthorId` and `Published` are defined as indexes. This helps speed up database lookups.

### Create the Migration

The migration file name is the current date _YYYYMMDDHHMMSS\_Initial.cs_. With the following attribute added to the class:

```csharp
[MigrationVersion("YYYY-MM-DD HH:MM:SS", "HelloWorld: Initial")]
```

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
