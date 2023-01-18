---
description: Application configuration framework
---

# 🐣 Configuration

## Overview

Configuring an application is usually done by the user in the backend via the **Configuration / Settings** UI. Modules can provide their own settings and a form to edit them.

At the lowest tier, each individual setting is just a record in the database represented by the [Setting](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Configuration/Domain/Setting.cs) entity. A setting's value is saved in the `Value` field as plain text.

To make things easy to work with, settings are grouped and combined into POCO classes. Here are some examples:

* [TaxSettings](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Checkout/Tax/Settings/TaxSettings.cs)
* [ThemeSettings](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Theming/Settings/ThemeSettings.cs)
* [MediaSettings](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Content/Media/Configuration/MediaSettings.cs)
* [CatalogSettings](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Catalog/CatalogSettings.cs)

## Technical concept

Every settings class needs to implement the `ISettings` interface and **must have** a public constructor without parameters.

Each property in the class represents an individual setting which name in the database is a combination of the class and property name.

{% hint style="info" %}
The `DefaultTheme` property in the `ThemeSettings` class results in: _ThemeSettings.DefaultTheme_.
{% endhint %}

The settings database value is the string representation of the property value, which means that the property type must be convertible **to** and **from** a string.

Only public properties with a getter and a setter are eligible as persistent settings. All other properties (or members in general) are ignored.

{% hint style="info" %}
The application's own [type conversion system](../advanced/type-conversion.md) is used to convert between types.
{% endhint %}

Setting entries are multi-store enabled and an entry's value can optionally be overwritten on store level.

{% hint style="warning" %}
Don't use complex types for setting properties. But if you must, [create and register a type converter](../advanced/type-conversion.md) for your type.
{% endhint %}

## Accessing settings

### By `Dependency Injection`

The easiest and most widely used setting access pattern is to pass them around as dependencies.

A special component registration source registers all classes implementing `ISettings` dynamically as **singleton** dependencies.

```csharp
private readonly MySettings1 _mySettings1;
private readonly MySettings2 _mySettings2;
 
public MyFakeService(MySettings1 mySettings1, MySettings2 mySettings2)
{
    _mySettings1 = mySettings1;
    _mySettings2 = mySettings2;
}
```

{% hint style="warning" %}
Don't update setting properties programmatically. Because setting classes are singletons, your changes will live as long as the app runs or the cache is cleared.

But if you must, you need to save your changes (read further below).
{% endhint %}

### By `ISettingFactory`

The singleton [ISettingFactory](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Configuration/Services/ISettingFactory.cs) is responsible for activating and populating setting class instances that implement `ISettings.`

The method for loading settings is `LoadSettingsAsync<TSettings>()`. It tries to load `TSettings` for a given store from cache or from database, if it’s not yet cached.

Similarly the method for saving settings is `SaveSettingsAsync<TSettings>()`. It saves a settings instance for a given store in the database.

```csharp
public void UpdateSettings(string name, int storeId)
{
    var mySettings = await Services.SettingFactory.LoadSettingsAsync<MySettings>(storeId);

    mySettings.Name = name;

    await Services.SettingFactory.SaveSettingsAsync(mySettings, storeId);
}
```

### Accessing individual setting entries

You can also access individual entries by using the [ISettingService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Configuration/Services/ISettingService.cs).

Updating individual entries automatically invalidates the classes cache. An example of which would be that updating or deleting the `ThemeSettings.DefaultTheme` entry removes all `ThemeSettings` instances from cache.

{% hint style="info" %}
You are not restricted to setting classes. Any setting entry can be created and accessed, without being part of a setting class.
{% endhint %}

## Tutorial

The following code shows how to provide custom settings with a full-blown multi-store enabled editor. It also provides some useful code for a pseudo-Blog module.

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

Although it’s not mandatory, it’s good practice to separate the UI from the service tier. To do that, create a view model for `BlogSettings`.

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

{% hint style="info" %}
Refer to:

* [data-modelling](data-modelling/ "mention") for more info about modelling in Smartstore.
* [localization.md](../content/localization.md "mention") to learn more about the `LocalizedDisplay` attribute.
* [validation.md](validation.md "mention") to learn how to validate your model on form post.
{% endhint %}

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

Decorate the GET action with the `LoadSetting` Attribute, and the POST action with the `SaveSetting` Attribute. They are not required, but save you from writing tedious, repetitive code.

The `LoadSetting` Attribute resolves all setting class action parameters automatically from Dependency Injection (in this case `BlogSettings`) and passes them to the method. It’s also fills the parameter `int storeScope` with the current store id, if it is stated.

The `SaveSetting` Attribute does roughly the same including:

* Patching the model parameter according to current store scope and leaving out non-overwritten properties.
* Saving the setting instance to database.

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

{% hint style="info" %}
Refer to:

* [security.md](security.md "mention") to learn more about `PermissionAttribute` and how to secure your actions.
* [model-mapping.md](data-modelling/model-mapping.md "mention") to learn more about the tiny and cute [MiniMapper](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/ComponentModel/MiniMapper.cs).
{% endhint %}

### Create menu item

There are many ways to hook your settings page into the backend.

{% hint style="info" %}
Please refer to [menus.md](../content/menus.md "mention") to learn about the menu system and how to hook in.
{% endhint %}
