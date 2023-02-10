# 🥚 Database Migrations

## Overview

Migrations are a structured way to alter a database schema and required when changes need to be made to the Smartstore's database tables or new tables need to be added. Migrations can be in the Smartstore core or in a module, depending on who created the related entity.

Smartstore uses [FluentMigrator](https://fluentmigrator.github.io/) as a framework for database migrations. FluentMigrator supports numerous database systems such as MySQL or PostgreSQL where Migrations are described in C# classes that can be checked into the FluentMigrator version control system.

FluentMigrator can also write data into the database, but this is more comfortable with the help of [IDataSeeder](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Data/Migrations/IDataSeeder.cs) or its abstract implementation [DataSeeder](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Data/Migrations/DataSeeder%60T.cs). Data seeding is used, for example, when a newly created table is to be filled with sample data from the very beginning.

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

## What does a migration look like?

## Requirements for modules

## Data seeding

SeedingDbMigrationEvent, SeededDbMigrationEvent

### Seeding tools

LocaleResourcesBuilder, SettingsBuilder --> MigrationBuilderDbContextExtensions
