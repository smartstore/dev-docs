# 🐣 Building a simple "Hello World" module

Before we get into the topic, please take a look at the introduction to creating modules [getting-started-with-modules.md](../getting-started-with-modules.md "mention"). The basic files required to create a module are already described here.

## Creating a project file

We start by creating a project file for our modules.

1. Open the Smartstore Solution _Smartstore.sln_
2. Right click on the _Modules_ Folder in the Solution Explorer
3. Add a **New Project** of type _Class Library_
4. Name it _MyOrg.HelloWorld_
5. Make sure the physical path of the project is _Smartstore/src/Smartstore.Modules_

Now we alter `MyOrg.HelloWorld.csproj` to the following:

```xml
<Project Sdk="Microsoft.NET.Sdk.Razor">
    <PropertyGroup>
        <Product>A Hello World module for Smartstore</Product>
        <OutputPath>..\..\Smartstore.Web\Modules\MyOrg.HelloWorld</OutputPath>
        <OutDir>$(OutputPath)</OutDir>
    </PropertyGroup>
</Project>
```

## Adding module metadata (module.json)

Let's add `module.json` now. For more information on this file refer to [getting-started-with-modules.md](../getting-started-with-modules.md "mention")

1. Right click on the project in the Solution Explorer.
2. **Add / New Item / Javascript JSON Configuration File**.
3. Name it `module.json`
4. Make another right click, select the **Properties** context item and change:

| Property                 | Value         |
| ------------------------ | ------------- |
| Build Action             | Content       |
| Copy to Output Directory | Copy if newer |

&#x20;  5\. Add the following content

{% code title="module.json" %}
```json
{
  "$schema": "../module.schema.json",
  "Author": "MyOrg",
  "Group": "Admin",
  "SystemName": "MyOrg.HelloWorld",
  "FriendlyName": "Hello World",
  "Description": "This module says Hello World",
  "Version": "5.0",
  "MinAppVersion": "5.0",
  "Order": 1,
  "ResourceRootKey": "Plugins.MyOrg.HelloWorld",
  "ProjectUrl": "https://myorg.com"
}
```
{% endcode %}

## Creating the module

Now we change the name of `Class1.cs` to `Module.cs` and add the following code:

{% code title="Module.cs" %}
```csharp
using System.Threading.Tasks;
using Smartstore.Engine.Modularity;
using Smartstore.Http;

internal class Module : ModuleBase, IConfigurable
{
    public RouteInfo GetConfigurationRoute()
        => new("Configure", "HelloWorldAdmin", new { area = "Admin" });

    public override async Task InstallAsync(ModuleInstallationContext context)
    {
        // Saves the default state of a settings class to the database 
        // without overwriting existing values.
        //await TrySaveSettingsAsync<HelloWorldSettings>();
        
        // Imports all language resources for the current module from 
        // xml files in "Localization" directory (if any found).
        await ImportLanguageResourcesAsync();
        
        // VERY IMPORTANT! Don't forget to call.
        await base.InstallAsync(context);
    }

    public override async Task UninstallAsync()
    {
        // Deletes all "HelloWorldSettings" properties settings from the database.
        //await DeleteSettingsAsync<HelloWorldSettings>();
        
        // Deletes all language resource for the current module 
        // if "ResourceRootKey" is module.json is not empty.
        await DeleteLanguageResourcesAsync();
        
        // VERY IMPORTANT! Don't forget to call.
        await base.UninstallAsync();
    }
}
```
{% endcode %}

If we now compile the project, we have a module that is recognized by Smartstore and can be installed via **Admin / Plugins / Manage Plugins / Hello World / Install**.

Note two things here:

1. If you click on **Configure** you will be led to a 404 page because we haven't added a controller and an action to handle the configuration route.
2. The method to add the default settings to the settings table in the database and the method to remove them are commented out because we haven't created a _Settings_ class yet.

## Adding a Settings class

For more detailed information on _Settings_ visit the section [configuration.md](../../../framework/platform/configuration.md "mention"). For this tutorial we add a simple _Setting_ class with just one string property.

1. Right click on the project in the Solution Explorer.
2. Add a new folder. According to our guidelines we call it _Configuration_.
3. Place a new class called `HelloWorldSettings.cs` in this folder.

{% code title="Module.cs" %}
```csharp
using Smartstore.Core.Configuration;

namespace MyOrg.HelloWorld.Settings
{
    public class HelloWorldSettings : ISettings
    {
        public string Name { get; set; } = "John Smith";
    }
}
```
{% endcode %}

Now we can uncomment the corresponding lines in our `Module.cs`, which saves the initial setting values when installing the module or removes them if the module becomes uninstalled. Now when the module is reinstalled, the `HelloWorldSettings.Name` setting is stored in the database with the default value of "John Smith".

## Adding configuration

Now that we have a setting for our module lets add the code to make this setting configurable. In our `Module.cs` we have implemented the interface `IConfigurable` which forces us to implement the method `GetConfigurationRoute` with the return type `RouteInfo`. This method will be called if the store owner clicks on the **Config** button next to the module in the **Plugin Management** section of the shop administration area.

{% code title="Module.cs" %}
```csharp
public RouteInfo GetConfigurationRoute()
    => new("Configure", "HelloWorldAdmin", new { area = "Admin" });
```
{% endcode %}

With the `RouteInfo` we are returning here, we tell Smartstore __ to look for an action called `Configure` in a Controller named `HelloWorldAdminController` in the area `Admin`.

## MVC parts

### Controller

So lets add the controller:

1. Right click on the project in the Solution Explorer.
2. Add a new folder. According to our guidelines we call it _Controllers_.
3. Place a new class called `HelloWorldAdminController.cs` in this folder.

{% code title="HelloWorldAdminController.cs" %}
```csharp
using Microsoft.AspNetCore.Mvc;
using Smartstore.ComponentModel;
using Smartstore.Core.Security;
using MyOrg.HelloWorld.Models;
using MyOrg.HelloWorld.Settings;
using Smartstore.Web.Controllers;
using Smartstore.Web.Modelling.Settings;

namespace MyOrg.HelloWorld.Controllers
{
    public class HelloWorldAdminController : AdminController
    {
        [LoadSetting, AuthorizeAdmin]
        public IActionResult Configure(HelloWorldSettings settings)
        {
            var model = MiniMapper.Map<HelloWorldSettings, ConfigurationModel>(settings);
            return View(model);
        }

        [HttpPost, SaveSetting, AuthorizeAdmin]
        public IActionResult Configure(ConfigurationModel model, HelloWorldSettings settings)
        {
            if (!ModelState.IsValid)
            {
                return Configure(settings);
            }

            ModelState.Clear();
            MiniMapper.Map(model, settings);

            return RedirectToAction(nameof(Configure));
        }
    }
}
```
{% endcode %}

We haven't added a configuration model yet so there will be 3 errors right now. This will be the next step.

Notice the area attribute the controller is decorated with. This means all actions of this controller will be reachable within this area only. If you want to add actions to the module within another area don't forget to decorate these actions with the desired area or add another controller.

According to the MVC pattern, we have two actions in this controller to handle the configure view we are about to add. The first action is for the `GET` request and the second will handle `POST` requests.

The `AuthorizeAdmin` attribute makes sure the current user has the right to access this view.

The `LoadSetting` attribute loads the setting values of the the settings class passed as the action parameter automatically from the database.

The `SaveSetting` attribute saves the setting values of the the settings class passed as the action parameter automatically to the database after the action was executed. So we have the opportunity to store our model values into the settings object. We'll do this here with the `MiniMapper` which can map simple properties with the same name to each other. Calling `MiniMapper.Map(model, settings)` will map the _Name_ property of the setting class to the `Name` property of the model class.

If the `ModelState` is not valid we must do a postback by returning `Configure(settings)` in order to display model validation errors. Else we redirect to the `GET` action in order to prevent unnecessary form posts.

### Model

As already stated the model will be a simple equivalent to the settings class. Lets add it:

1. Right click on the project in the Solution Explorer.
2. Add a new folder. According to our guidelines we call it _Models_.
3. Place a new class called `ConfigurationModel.cs` in this folder.

{% code title="ConfigurationModel.cs" %}
```csharp
using Smartstore.Web.Modelling;

namespace MyOrg.HelloWorld.Models
{
    [LocalizedDisplay("Plugins.MyOrg.HelloWorld.")]
    public class ConfigurationModel : ModelBase
    {
        [LocalizedDisplay("*Name")]
        public string Name { get; set; }
    }
}
```
{% endcode %}

### View

Lets add the view which is rendered by the `GET` action of the controller.

1. Right click on the project in the Solution Explorer.
2. Add a new folder and call it _Views/HelloWorldAdmin_.
3. Place a new **Empty Razor View** called `Configure.cshtml` in this folder.

{% code title="Configure.cshtml" %}
```cshtml
@model ConfigurationModel

@{
    Layout = "_ConfigureModule";
}

@* 
    Render "StoreScope" component if your setting class has 
    one or more multi-store enabled properties.
    It renders a store chooser that sets the current store scope.
    This way individual settings can be overridden on store level.
*@

@await Component.InvokeAsync("StoreScope")

@* Render the save button in admin toolbar *@
<widget target-zone="admin_button_toolbar_before">
    <button id="SaveConfigButton" type="submit" name="save" class="btn btn-warning" value="save">
        <i class="fa fa-check"></i>
        <span>@T("Admin.Common.Save")</span>
    </button>
</widget>

<form asp-action="Configure">
    <div asp-validation-summary="All"></div>
    <div class="adminContent">
        <div class="adminRow">
            <div class="adminTitle">
                <smart-label asp-for="Name" />
            </div>
            <div class="adminData">
                <setting-editor asp-for="Name"></setting-editor>
                <span asp-validation-for="Name"></span>
            </div>
        </div>
    </div>
</form>
```
{% endcode %}

To spare some using directives in the view it is recommended to add a `_ViewImports.cshtml` file directly in the _Views_ directory. It'll add the most important namespaces and the model namespace because every view has to deal with a model somehow. Also included in this sample are the Microsoft built in Tag Helpers as well as the Tag Helpers that are included in Smartstore.

{% code title="_ViewImports.cshtml" %}
```cshtml
@inherits Smartstore.Web.Razor.SmartRazorPage<TModel>

@using System
@using System.Globalization
@using Smartstore.Web.TagHelpers.Admin
@using Smartstore.Web.Rendering
@using MyOrg.HelloWorld.Models

@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper Smartstore.Web.TagHelpers.Shared.*, Smartstore.Web.Common
@addTagHelper Smartstore.Web.TagHelpers.Admin.*, Smartstore.Web.Common
```
{% endcode %}

After the the module has been built, you can click on the **Configure** button and will be able to store a value for the setting `HelloWorldSettings.Name` __ to the database by just entering it in the provided input field of the configuration view.

## Adding localization

If you look at the `ConfigurationModel` you'll see that the properties of the model are decorated with the `LocalizedDisplay` attribute. By doing so you can add localized display values that describe the property. The attribute on property level can either contain the full resource key `[LocalizedDisplay("Plugins.MyOrg.HelloWorld.Name")]` or inherit a part from the declaring class that is also decorated with this attribute like it is done in our example.

The resource values itself must be added by XML files. Let's do this:

1. Right click on the project in the Solution Explorer.
2. Add a new folder. The folder must be called _Localization_.
3. Place a new XML file called `resources.en-us.xml` in this folder.
4. Make another right click, select the **Properties** context item and change&#x20;

| Property                 | Value         |
| ------------------------ | ------------- |
| Build Action             | Content       |
| Copy to Output Directory | Copy if newer |

```xml
<Language Name="English" IsDefault="false" IsRightToLeft="false">
    <LocaleResource Name="Plugins.FriendlyName.MyOrg.HelloWorld" AppendRootKey="false">
        <Value>Hello World</Value>
    </LocaleResource>
    <LocaleResource Name="Plugins.Description.MyOrg.HelloWorld" AppendRootKey="false">
        <Value>This plugin says Hello World.</Value>
    </LocaleResource>

    <LocaleResource Name="Plugins.MyOrg.HelloWorld" AppendRootKey="false">
        <Children>
            <LocaleResource Name="Name">
                <Value>Name to greet</Value>
            </LocaleResource>
            <LocaleResource Name="Name.Hint">
                <Value>Enter the name of the person to be greeted.</Value>
            </LocaleResource>
        </Children>
    </LocaleResource>
</Language>
```

After building the module you can press the button **Update resources** to update the newly added localized resources from your XML file.

## Say Hello

Now that we can configure the name of the person that should be greeted by the module let's do some public rendering.&#x20;

We add another controller, a model and a view for the public action.&#x20;

1. Right click on the _Controllers_ directory in the Solution Explorer.
2. Add a new class called `HelloWorldController.cs`&#x20;

{% code title="Controllers/HelloWorldController.cs" %}
```csharp
using Microsoft.AspNetCore.Mvc;
using MyOrg.HelloWorld.Models;
using MyOrg.HelloWorld.Settings;
using Smartstore.ComponentModel;
using Smartstore.Web.Controllers;
using Smartstore.Web.Modelling.Settings;

namespace MyOrg.HelloWorld.Controllers
{
    public class HelloWorldController : PublicController
    {
        [LoadSetting]
        public IActionResult PublicInfo(HelloWorldSettings settings)
        {
            var model = MiniMapper.Map<HelloWorldSettings, PublicInfoModel>(settings);
            return View(model);
        }
    }
}
```
{% endcode %}

1. Right click on the _Models_ directory in the Solution Explorer.
2. Add a new class called `PublicInfoModel.cs`&#x20;

{% code title="Models/PublicInfoModel.cs" %}
```csharp
using Smartstore.Web.Modelling;

namespace MyOrg.HelloWorld.Models
{
    public class PublicInfoModel : ModelBase
    {
        public string Name { get; set; }
    }
}
```
{% endcode %}

1. Right click on the _Views_ directory in the Solution Explorer.
2. Add a new folder named _HelloWorld_
3. Add a new Razor View called `PublicInfo.cshtml`&#x20;

{% code title="Views\HelloWorld\PublicInfo.cshtml" %}
```html
@model PublicInfoModel

@{
    Layout = "_Layout";
}

<div>
    Hello @Model.Name
</div>
```
{% endcode %}

The public view will be displayed when opening the URL: [http://localhost:59318/helloworld/publicInfo](http://localhost:59318/helloworld/publicInfo)&#x20;

## Finally

Open the project file and remove all `ItemGroup` properties because they are not required for the _Smartstore_ build process.

Now we have built a simple module that can store a setting and renders its value in the frontend when accessing the route _helloworld/publicinfo_. Of course this is only a starting point on the way to build more complex modules by using _Action Filters_, own _DataContext_ and last but not least _View Components_ which will be rendered into _Widget Zones_.

The code for this tutorial can be downloaded here:

{% file src="../../../.gitbook/assets/MyOrg.HelloWorld.zip" %}
