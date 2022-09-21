---
description: Application configuration framework
---

# Configuration

* Application configuration is usually performed by the user in the backend via **Configuration / Settings** UI
* Modules may provide their own settings and a form to edit them
* At the lowest tier, each individual setting is just a record in the database represented by the [`Setting`](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Configuration/Domain/Setting.cs) entity. A setting's value is saved in the `Value` field as plain text.
* But to make things easy to work with, settings are grouped and combined into POCO classes, e.g.:
  * ``[`TaxSettings`](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Checkout/Tax/Settings/TaxSettings.cs)``
  * ``[`ThemeSettings`](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Theming/Settings/ThemeSettings.cs)``
  * ``[`MediaSettings`](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Content/Media/Configuration/MediaSettings.cs)``
  * ``[`CatalogSettings`](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Catalog/CatalogSettings.cs) (a really big one :smile:)
  * and many more

## Technical concept

* It is the `ISettings` marker interface that makes a class a settings class
* The class **must have** public parameterless constructor
* Each property in the class represents an individual setting entry
* The setting name in the database is a combination of type and property name. E.g. the `DefaultTheme` property in the `ThemeSettings` class results in: _ThemeSettings.DefaultTheme_.
* The setting value in the database is the string representation of the property value.
* Therefore: the property type must be convertible **to** and **from** string.
* Only public properties with both a getter and a setter are eligible as persistable settings. All other properties (or members in general) are ignored, but they do no harm.
* The application's inbuilt [type conversion system](../advanced/type-conversion.md) is used to convert between types.
* Setting entries are multi-store enabled
  * An entry's value can optionally be overwritten on store level.
* WARN: don't use complex types for setting properties. But if you must, [create and register a type converter](../advanced/type-conversion.md) for your type.

## Accessing settings

### By DI

* The easiest and most widely used setting access pattern is to pass them around as dependencies...
* ...because a special component registration source registers all classes implementing `ISettings` dynamically as **singleton** dependencies.
* _SAMPLE_ (fake service class with one or two settings in ctor)
* WARN: don't update setting properties programmatically. Because setting classes are singletons, your changes will live as long as the app runs or the cache is cleared. But if you must, you have to save your changes (read further below).

### By ISettingFactory

* ``[`ISettingFactory`](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Configuration/Services/ISettingFactory.cs) (which is also singleton) is responsible for activating and populating setting class instances that implement `ISettings`
* Loading settings
  * `LoadSettingsAsync<TSettings>()`: tries to load `TSettings` for a given store from cache or from database (if not cached yet)
* Saving settings
  * `SaveSettingsAsync<TSettings>()`: saves a settings instance for a given store in the database
* _SAMPLE_ (LoadSettingsAsync with storeId passed --> update settings --> SaveSettingsAsync with storeId passed)

### Accessing individual setting entries

* You can also access individual entries by using [`ISettingService`](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Configuration/Services/ISettingService.cs)``
* Updating individual entries automatically invalidates the class cache. E.g. updating or deleting the `ThemeSettings.DefaultTheme` entry removes all `ThemeSettings` instances from cache.
* INFO: you are not restricted to setting classes. Any setting entry can be created and accessed, without being part of a setting class.

## Tutorial

* How to provide custom settings with a full-blown multi-store enabled editor

### Create settings class

Lorem ipsum

### Create settings model

Lorem ipsum

### Create controller actions

Lorem ipsum

### Create view

Lorem ipsum

### Create menu item

Lorem ipsum
