---
description: Application configuration framework
---

# 👍 Configuration

## Overview

* Application configuration is usually performed by the user in the backend via **Configuration / Settings** UI
* Modules may provide their own settings and a form to edit them
* At the lowest tier, each individual setting is just a record in the database represented by the [Setting](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Configuration/Domain/Setting.cs) entity. A setting's value is saved in the `Value` field as plain text.
* But to make things easy to work with, settings are grouped and combined into POCO classes, e.g.:
  * [TaxSettings](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Checkout/Tax/Settings/TaxSettings.cs)
  * [ThemeSettings](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Theming/Settings/ThemeSettings.cs)
  * [MediaSettings](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Content/Media/Configuration/MediaSettings.cs)
  * [CatalogSettings](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Catalog/CatalogSettings.cs) (a really big one :smile:)
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

* [ISettingFactory](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Configuration/Services/ISettingFactory.cs) (which is also singleton) is responsible for activating and populating setting class instances that implement `ISettings`
* Loading settings
  * `LoadSettingsAsync<TSettings>()`: tries to load `TSettings` for a given store from cache or from database (if not cached yet)
* Saving settings
  * `SaveSettingsAsync<TSettings>()`: saves a settings instance for a given store in the database
* _SAMPLE_ (LoadSettingsAsync with storeId passed --> update settings --> SaveSettingsAsync with storeId passed)

### Accessing individual setting entries

* You can also access individual entries by using [ISettingService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Configuration/Services/ISettingService.cs)
* Updating individual entries automatically invalidates the class cache. E.g. updating or deleting the `ThemeSettings.DefaultTheme` entry removes all `ThemeSettings` instances from cache.
* INFO: you are not restricted to setting classes. Any setting entry can be created and accessed, without being part of a setting class.

## Tutorial

* How to provide custom settings with a full-blown multi-store enabled editor
* Some useful code for a pseudo _Blog_ module

### Create settings class

{% code title="Configuration/BlogSettings.cs" %}
```csharp
public class BlogSettings : ISettings
{
    /// <summary>
    /// Gets or sets a value indicating whether blog is enabled.
    /// </summary>
    public bool Enabled { get; set; } = true;

    /// <summary>
    /// Gets or sets the page size for posts.
    /// </summary>
    public int PostsPageSize { get; set; } = 10;

    /// <summary>
    /// Gets or sets a value indicating whether users can leave comments.
    /// </summary>
    public bool EnableComments { get; set; } = true;

    /// <summary>
    /// Gets or sets a value indicating whether to notify about new blog comments.
    /// </summary>
    public bool EnableNotifications { get; set; }
}
```
{% endcode %}

### Create settings model

* Although not mandatory, we gonna create a view model for `BlogSettings`...
* ...because we want to separate concerns: it is good practice to separate UI and service tier
* Refer to [data-modelling](data-modelling/ "mention") for more info about modelling in Smartstore
* Refer to [localization.md](../content/localization.md "mention") to learn more about the `LocalizedDisplay` attribute
* Refer to [validation.md](validation.md "mention") to learn how to validate your model on form post

{% code title="Models/ConfigurationModel.cs" %}
```csharp
[LocalizedDisplay("Plugins.My.Blog.")]
public class BlogSettingsModel : ModelBase
{
    [LocalizedDisplay("*Enabled")]
    public bool Enabled { get; set; }

    [LocalizedDisplay("*PostsPageSize")]
    public int PostsPageSize { get; set; }

    [LocalizedDisplay("*EnableComments")]
    public bool EnableComments { get; set; }

    [LocalizedDisplay("*EnableNotifications")]
    public bool EnableNotifications { get; set; }
}
```
{% endcode %}

### Create view

{% code title="Views/BlogAdmin/Configure.cshtml" %}
```cshtml
@model BlogSettingsModel

@{
    // Specialized layout for setting editors
    Layout = "_SettingLayout";
}

<form asp-action="Configure">
    @* Page header with title and save button  *@
    <div class="section-header">
        <div class="title">
            @T("Plugins.My.Blog.Title")
        </div>
        <div class="options">
            <button type="submit" name="save" value="save" class="btn btn-warning">
                <i class="fa fa-check"></i>
                <span>@T("Admin.Common.Save")</span>
            </button>
        </div>
    </div>
    
    @* 
        Render "StoreScope" component if your setting class has 
        one or more multi-store enabled properties.
        It renders a store chooser that sets the current store scope.
        This way individual settings can be overridden on store level.
    *@
    @await Component.InvokeAsync("StoreScope")

    <div asp-validation-summary="All"></div>

    <div class="adminContent">
        <div class="adminRow">
            <div class="adminTitle">
                <smart-label asp-for="Enabled" />
            </div>
            <div class="adminData">
                @*
                    The "setting-editor" TagHelper is an extended
                    variant of the "setting" TagHelper: it additionally
                    renders an "override" checkbox next to the actual
                    editor control.
                *@
                <setting-editor asp-for="Enabled"></setting-editor>
            </div>
        </div>
        <div class="adminRow">
            <div class="adminTitle">
                <smart-label asp-for="PostsPageSize" />
            </div>
            <div class="adminData">
                <setting-editor asp-for="PostsPageSize"></setting-editor>
                <span asp-validation-for="PostsPageSize"></span>
            </div>
        </div>
        <div class="adminRow">
            <div class="adminTitle">
                <smart-label asp-for="EnableComments" />
            </div>
            <div class="adminData">
                @*
                    If you don't want to enable multi-store overrides
                    on property level, just use "editor" instead
                    of "setting-editor"
                *@
                <editor asp-for="EnableComments"></editor>
            </div>
        </div>
        <div class="adminRow">
            <div class="adminTitle">
                <smart-label asp-for="EnableNotifications" />
            </div>
            <div class="adminData">
                <setting-editor asp-for="EnableNotifications"></setting-editor>
            </div>
        </div>
    </div>
</form>
```
{% endcode %}

### Create controller actions

* We gonna decorate the GET action with `LoadSettingAttribute`, and the POST action with  `SaveSettingAttribute`. They are not required, but save us from writing tedious, repetitive code.
  * `LoadSettingAttribute` resolves all setting class action parameters automatically from DI - in our case `BlogSettings` - and passes them to the method. It is also capable of querying the current store scope.
  * `SaveSettingAttribute` roughly does the same + patching the model parameter according to current store scope (leaving out non-overwritten properties) + saving the setting instance to database.
* Refer to [security.md](security.md "mention") to learn more about `PermissionAttribute` and how to secure your actions.
* Refer to [model-mapping.md](data-modelling/model-mapping.md "mention") to learn more about the tiny and cute [MiniMapper](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/ComponentModel/MiniMapper.cs).

{% code title="Controllers/BlogAdminController.cs" %}
```csharp
[Route("[area]/blog/{action=index}/{id?}")]
public class BlogAdminController : AdminController
{
    [Permission(BlogPermissions.Read)]
    [LoadSetting]
    public IActionResult Configure(BlogSettings settings)
    {
        // Map BlogSettings --> BlogSettingsModel
        var model = MiniMapper.Map<BlogSettings, BlogSettingsModel>(settings);

        // Pass mapped model to view
        return View(model);
    }

    [Permission(BlogPermissions.Update)]
    [HttpPost, SaveSetting]
    public IActionResult Configure(BlogSettingsModel model, BlogSettings settings)
    {
        if (!ModelState.IsValid)
        {
            // Re-render editor if model is invalid
            return Configure(settings);
        }

        ModelState.Clear();

        // Map BlogSettingsModel --> BlogSettings.
        // "SaveSettingAttribute" filter handles saving for us later.
        MiniMapper.Map(model, settings);

        return RedirectToAction(nameof(Configure));
    }
}
```
{% endcode %}

### Create menu item

* There are many ways to _hook_ your settings page into the backend
* Please refer to [menus.md](../content/menus.md "mention") to learn about the menu system and how to hook in.
