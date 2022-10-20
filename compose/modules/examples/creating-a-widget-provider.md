---
description: >-
  There are many ways to display or inject Content in Smartstore. One of the
  methods is to use a Widget.
---

# 🥚 Creating a Widget provider

{% hint style="info" %}
For a more in-depth view on widgets, -zones and -invokers, please refer to [Widgets](../../../framework/content/widgets.md).
{% endhint %}

Following the [last tutorial](adding-tabs.md), you'll need to do these steps to implement a Widget:

* modify [**`Module.cs`**](creating-a-widget-provider.md#implementing-iwidget)
* create [**`ViewComponentModel.cs`**](creating-a-widget-provider.md#modify-publicinfomodel)
* modify [**`MyOrg.HelloWorld.csproj`**](creating-a-widget-provider.md#adding-database-access)****
* add **Components /** [**`ViewComponent.cs`**](creating-a-widget-provider.md#adding-viewcomponent)
* add [**`CacheableRoutes.cs`**](creating-a-widget-provider.md#adding-the-cacheableroutes)
* add **Views /** **Shared /** **Components /** _**\<ComponentName> /**_ [**`Default.cshtml`**](creating-a-widget-provider.md#adding-a-view)

## Implementing the IWidget

Using the module from the [Adding Tabs tutorial](adding-tabs.md), we'll start with the `Module.cs` file.

You'll need to add the interface `IWidget` to the implementation.

```csharp
public class Module : ModuleBase, IConfigurable, IWidget
```

This will force you to implement the following two Methods:

```csharp
public WidgetInvoker GetDisplayWidget(string widgetZone, object model, int storeId)

public string[] GetWidgetZones()
```

`GetWidgetZones` is a string array containing every Widget-Zone we want to access.

```csharp
public string[] GetWidgetZones()
{
    return new string[] { "target_widget_zone_name" };
}
```

In this tutorial we'll be using the `productdetails_pictures_top` widget zone. It is placed above the product picture when using the Frontend.

{% hint style="info" %}
More examples of widget zone names can be found [here](../../../framework/content/widgets.md#list-of-all-core-widget-zone-names).
{% endhint %}

{% hint style="info" %}
If you want to see the widget zones in your store, you can use DevTools.

1. Install _Smartstore Developer Tools_.
2. Click on _Configure_.
3. Activate the option to _Display Widget Zones_.
{% endhint %}

A simple implementation for `GetDisplayWidget` would be

```csharp
public WidgetInvoker GetDisplayWidget(string widgetZone, object model, int storeId)
    => new ComponentWidgetInvoker(typeof(HelloWorldViewComponent), new {widgetZone, model, storeId});
```

which creates a `ComponentWidgetInvoker` for all `widgetZone` we specified in `GetWidgetZones`.

Your code should look something like this:

{% code title="Module.cs" %}
```csharp
public class Module : ModuleBase, IConfigurable, IWidget
{
    ...

    public WidgetInvoker GetDisplayWidget(string widgetZone, object model, int storeId)
        => new ComponentWidgetInvoker(typeof(HelloWorldViewComponent), new {widgetZone, model, storeId});

    public string[] GetWidgetZones()
    {
        return new string[] { "productdetails_pictures_top" };
    }

    ...
}
```
{% endcode %}

## Create the ViewComponentModel

For the widget to access `MyTabValue` from the [previous tutorial](adding-tabs.md), you'll need to create a new model.

1. Right click on the _Models_ folder in the Solution Explorer.
2. Place a new class called `ViewComponentModel.cs` in this folder.

Then add the following lines:

<pre class="language-csharp" data-title="ViewComponentModel.cs"><code class="lang-csharp"><strong>using Smartstore.Web.Modelling;
</strong>
namespace MyOrg.HelloWorld.Models
{
    public class ViewComponentModel : ModelBase
    {
        public string MyTabValue { get; set; }
    }
}
</code></pre>

## Adding database access

For us to display product specific information we need database access.

To access the database you need to add these lines to your project file:

```xml
<ItemGroup>
    <ProjectReference Include="..\..\Smartstore.Web\Smartstore.Web.csproj">
        <Private>False</Private>
        <CopyLocal>False</CopyLocal>
        <CopyLocalSatelliteAssemblies>False</CopyLocalSatelliteAssemblies>
    </ProjectReference>
</ItemGroup>
```

{% hint style="info" %}
To get to the project file, simply click on your project in the Solution Explorer.
{% endhint %}



## Adding the ViewComponent

1. Right click on the project in the Solution Explorer.
2. Add a new folder. According to our guidelines we call it _Components_.
3. Place a new class called _HelloWorldViewComponent.cs_ in this folder.

This class implements `SmartViewComponent`.

```csharp
public class HelloWorldViewComponent : SmartViewComponent
```

{% hint style="info" %}
By implementing `SmartViewComponent`we have access to a Logger, Localization and CommonServices
{% endhint %}

To use the product specific information we saved in `MyTabValue`, you need to add access to the database using dependency injection in the constructor.

```csharp
private readonly SmartDbContext _db;
public HelloWorldViewComponent(SmartDbContext db)
{
    _db = db;
}
```

Next you'll add the `InvokeAsync` method, that gets called each time a widget zone is about to get rendered and holds the model passed from `GetDisplayWidget`.

```csharp
public async Task<IViewComponentResult> InvokeAsync(string widgetZone, object model)
```

In case you're handling multiple widget zones and need to differentiate between them, you might add an if- or a switch-block for `widgetZone`.

If you just want to make sure that your widget isn't displaying anything, when it is not supposed to, add the following lines:

```csharp
if (widgetZone != "productdetails_pictures_top")
{
    return Empty();
}
```

After checking whether the correct model is being used, you fetch the product from the database and get the specified `MyTabValue`.

```csharp
if (model.GetType() != typeof(ProductDetailsModel))
{
    return Empty();
}

var productModel = (ProductDetailsModel)model;

var product = await _db.Products.FindByIdAsync(productModel.Id);

var attrValue = product.GenericAttributes.Get<string>("HelloWorldMyTabValue");
```

And finally you create the `ViewComponentModel` and return the View.

```csharp
var viewComponentModel = new ViewComponentModel
{
    MyTabValue = attrValue
};

return View(viewComponentModel);
```

The final code looks like this:

{% code title="HellowWorldViewComponents.cs" %}
```csharp
public class HelloWorldViewComponent : SmartViewComponent
{
    private readonly SmartDbContext _db;

    public HelloWorldViewComponent(SmartDbContext db)
    {
        _db = db;
    }

    public async Task<IViewComponentResult> InvokeAsync(string widgetZone, object model)
    {
        if (widgetZone != "productdetails_pictures_top")
        {
            return Empty();
        }

        if (model.GetType() != typeof(ProductDetailsModel))
        {
            return Empty();
        }

        var productModel = (ProductDetailsModel)model;
        var product = await _db.Products.FindByIdAsync(productModel.Id);
        var attrValue = product.GenericAttributes.Get<string>("HelloWorldMyTabValue");

        var viewComponentModel = new ViewComponentModel
        {
            MyTabValue = attrValue
        };

        return View(viewComponentModel);
    }
}
```
{% endcode %}

## Adding the CacheableRoutes

Old cache can cause trouble whilst developing modules. To avoid our model being `null`, you need to specify, that this `ViewComponent` gets to use the same cache until it's properly invalidated.

You can do this by adding the `CacheableRoutes` class.

1. Right click on the project in the Solution Explorer.
2. Place a new class called _CacheableRoutes.cs_ in this folder.

Then add these lines:

{% code title="CacheableRoutes.cs" %}
```csharp
using Smartstore.Core.OutputCache;

namespace MyOrg.HelloWorld
{
    internal sealed class CacheableRoutes : ICacheableRouteProvider
    {
        public int Order => 0;

        public IEnumerable<string> GetCacheableRoutes()
        {
            return new string[]
            {
                "vc:MyOrg.HelloWorld/HelloWorldViewComponent"
            };
        }
    }
}
```
{% endcode %}

This way we always get the most recent version in our View.

## Adding the View

1. Right click on the _Views_ **** folder in the Solution Explorer.
2. Add a new folder. According to our guidelines we call it _Shared_.
3. Add a new sub folder. According to our guidelines we call it _Components_.
4. Add a new sub folder. According to our guidelines we call it _\<ComponentName>_.
5. Place a new _Razor View_ called _Default.cshtml_ in this folder.

{% hint style="warning" %}
Don't forget to set the file properties to _Content_ and _Copy if newer_ as you did with [`module.json`](../tutorials/building-a-simple-hello-world-module.md#adding-module-metadata-module.json)

1. Right click on the _Default.cshtml_ file in the Solution Explorer.
2. Select the _Properties_ context item and change.


{% endhint %}

Add the following lines for a simple Output:

{% code title="Default.cshtml" %}
```cshtml
@model ViewComponentModel

@{
    Layout = "";
}

<span>Widget content: @Model.MyTabValue</span>
```
{% endcode %}

## Conclusion

Now you should be able to define a property in your product catalog and see it displayed above the product picture.

{% hint style="warning" %}
Don't forget to activate your widget!

1. Go to the Smartstore admin settings
2. Navigate to CMS -> Widgets
3. You should see your widget listed. Press _activate_.
{% endhint %}

In this tutorial you built a widget using the IWidget interface and specified your widget zones. You created a ViewComponent and bound it to different widget zones. And finally you can avoided an invalid model through proper cache use.

Hopefully this will get you started with widgets and enable you to build more complex modules.

The code for this module can be downloaded here:

{% file src="../../../.gitbook/assets/MyOrg.HelloWorldWidget.zip" %}
