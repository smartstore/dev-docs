# 🥚 Database Migrations

## Overview

Migrations are a structured way to alter a database schema and required when changes need to be made to the Smartstore's database tables or new tables need to be added. Migrations can be in the Smartstore core or in a module, depending on who created the related entity.

Smartstore uses [Fluent Migrator](https://fluentmigrator.github.io/) as a framework for database migrations. Fluent Migrator supports numerous database systems such as MySQL or PostgreSQL where Migrations are described in C# classes that can be checked into the Fluent Migrator version control system.

Fluent Migrator can also write data into the database, but this is more comfortable with the help of [IDataSeeder](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Data/Migrations/IDataSeeder.cs) or its abstract implementation [DataSeeder](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Data/Migrations/DataSeeder%60T.cs). Data seeding is used, for example, when a newly created table is to be filled with sample data from the very beginning.

## Database Migrator

[DbMigrator](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Data/Migrations/DbMigrator%60T.cs) is responsible for performing migrations. `MigrateAsync` is used to migrate the database to a specified version or to the latest if no version is specified.

WARN: `MigrateAsync` has an assembly parameter to perform only those migrations of a specific module. In general, this should always be set. If it is `null`, the migrations of _all_ modules are executed!

Following `MigrateAsync` downgrades all migrations of a certain module. It allows to remove all changes made by the module to the database schema.

```csharp
internal class Module : ModuleBase
{
    private readonly DbMigrator<SmartDbContext> _dbMigrator;

    public Module(DbMigrator<SmartDbContext> dbMigrator)
    {
        _dbMigrator = dbMigrator;
    }

    public override async Task UninstallAsync()
    {
        await _dbMigrator.MigrateAsync(-1, GetType().Assembly);
        await base.UninstallAsync();
    }
}
```

## Migrations history

Fluent Migrator keeps a version history of successfully executed migrations in the `__MigrationVersionInfo` database table. The content of this table could look like this

<figure><img src="../../.gitbook/assets/migration-history.png" alt=""><figcaption><p>Content of __MigrationVersionInfo table.</p></figcaption></figure>

By deleting an entry in this history, it is possible to run the related migration again at the next application startup. For example, if I want to run the inital migration of the Google Merchant Center module again, then delete the row for the related migration (see highlighted row in the image above). I also delete the `GoogleProduct` table because it is created by this migration. Without deleting of the table, the migration would be executed, but it would do nothing because `GoogleProduct` already exists. Or it would generate an error if it didn't first check if the table already exists.

WARN: Manual modifications to the database schema should only be carried out with great caution and sufficient knowledge of Smartstore. They may cause the software to stop working correctly and may require a database restorage from a backup.

## What does a migration look like?

A migration is an internal class inherited from Fluent Migrator's abstract `Migration` class. It must be decorated with the [MigrationVersionAttribute](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Data/Migrations/MigrationVersionAttribute.cs). It specifies the version and the short description. The version is determined from a timestamp, which usually always represents the date when the migration was created. The timestamp can have one of the following formats:

* yyyy-MM-dd HH:mm:ss
* yyyy/MM/dd HH:mm:ss
* yyyy.MM.dd HH:mm:ss

To make the short description more uniform, we recommend the format `<modul name>:<short description of migration>`, e.g. _Core: InternationalDiallingCodes_. The very first migration is usually named _Initial_, e.g. _MegaSearch: Initial_. A good example of a typical migration is [ProductComparePriceLabel](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Migrations/20221103091500\_ProductComparePriceLabel.cs).

```csharp
[MigrationVersion("2022-11-03 09:15:00", "Core: ProductComparePriceLabel")]
internal class ProductComparePriceLabel : Migration, ILocaleResourcesProvider, IDataSeeder<SmartDbContext>
{
    public bool RollbackOnFailure => false;
    
    public override void Up()
    {
        // Code omitted for clarity.
    }
    
    public override void Down()
    {
        // Code omitted for clarity.
    }
    
    public async Task SeedAsync(SmartDbContext context, CancellationToken cancelToken = default)
    {
        await context.MigrateLocaleResourcesAsync(MigrateLocaleResources);
    }

    public void MigrateLocaleResources(LocaleResourcesBuilder builder)
    {
        // Code omitted for clarity.
    }
}
```

| Method or property                              | Description                                                                                                               |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| IMigration:Up                                   | Collects the _up_ migration expressions.                                                                                  |
| IMigration:Down                                 | Collects the _down_ migration expressions.                                                                                |
| IDataSeeder:RollbackOnFailure                   | Gets a value indicating whether migration should be completely rolled back when an error occurs during migration seeding. |
| IDataSeeder:SeedAsync                           | Seeds any data into the database.                                                                                         |
| ILocaleResourcesProvider:MigrateLocaleResources | Seeds new or updated locale resources after a migration has run.                                                          |

More examples can be found in the [DevTools](https://github.com/smartstore/Smartstore/tree/main/src/Smartstore.Modules/Smartstore.DevTools/Migrations) module. They are for illustration purposes and are therefore commented out so that they are not executed.

TIP: For `ProductComparePriceLabel`, each statement is preceded by an existence check of the related resource (for a table, column, foreign key, field index, etc.). This is not mandatory but it is recommended. It makes your migration less vulnerable and ensures that in case of unpredictable events the migration will still execute correctly, regardless of how many times it is executed. If your migration was only partially executed due to an error, it will not be able to run successfully again without these existence checks and manual modification of the database schema would be required.

## Requirements for modules

## Data seeding

SeedingDbMigrationEvent, SeededDbMigrationEvent

### Seeding tools

LocaleResourcesBuilder, SettingsBuilder --> MigrationBuilderDbContextExtensions
