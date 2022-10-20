---
description: >-
  There are many ways to display or inject Content in Smartstore. One of the
  methods is to use a Widget.
---

# 🎶 Creating a Widget provider

{% hint style="info" %}
For a more in-depth view on widgets, -zones and -invokers, please refer to [Widgets](../../../framework/content/widgets.md)
{% endhint %}

Following the [last tutorial](adding-tabs.md), you'll need to do these steps to implement a Widget:

* modify [**`Module.cs`**](creating-a-widget-provider.md#implementing-iwidget)**``**
* modify [**PublicInfoModel.cs**](creating-a-widget-provider.md#modify-publicinfomodel)****
* add **Components** \ [**ViewComponent.cs**](creating-a-widget-provider.md#adding-viewcomponent)****
* add **Views** \ **Shared** \ **Components** \ _**\<ModuleName>**_ \ [**Default.cshtml**](creating-a-widget-provider.md#undefined)****

## Implementing IWidget

Using the module from the [Adding Tabs tutorial](adding-tabs.md), we'll start with the Module.cs file

You'll need to add `IWidget` to the implementation

```csharp
public class Module : ModuleBase, IConfigurable, IWidget
```

This will force you to implement the following two Methods:

```csharp
public WidgetInvoker GetDisplayWidget(string widgetZone, object model, int storeId)

public string[] GetWidgetZones()
```

**`GetWidgetZones` ** is a string array containing every Widget-Zone we want to access.

```csharp
public string[] GetWidgetZones()
{
    return new string[] { "Target_Widget_Zone_Name" };
}
```

In this tutorial we'll be using the `productdetails_pictures_top` widget zone. It is placed above the product picture when using the Frontend.

{% hint style="info" %}
More examples of widget zone names can be found [here](../../../framework/content/widgets.md#list-of-all-core-widget-zone-names)
{% endhint %}

A simple implementation for 'GetDisplayWidget' would be

```csharp
public WidgetInvoker GetDisplayWidget(string widgetZone, object model, int storeId)
    => new ComponentWidgetInvoker(typeof(HelloWorldViewComponent), new {widgetZone, model, storeId});
```

which creates a new `ComponentWidgetInvoker` for every widgetZone we specified in `GetWidgetZones`

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

## Modify PublicInfoModel

For the widget to access `MyTabValue` from the [previous tutorial](adding-tabs.md), you'll need to add the following lines to **Models** \ **PublicInfoModel.cs**:

```csharp
[LocalizedDisplay("*MyTabValue")] //Optional reference for Localization
public string MyTabValue { get; set; }
```

## Adding ViewComponent

1. Right click on the project in the Solution Explorer.
2. Add a new folder. According to our guidelines we call it **Components**.
3. Place a new class called **ViewComponent.cs** in this folder.

This class implements `SmartViewComponent`

```csharp
public class HelloWorldViewComponent : SmartViewComponent
```

{% hint style="info" %}
By implementing `SmartViewComponent`we have access to a Logger, Localization and CommonServices
{% endhint %}

To use the product specific information we saved in `MyTabValue`, you need to add access to the database.

```csharp
private readonly SmartDbContext _db;
public HelloWorldViewComponent(SmartDbContext db)
{
    _db = db;
}
```

Next you'll add the `InvokeAsync` method, that gets called each time a widget zone is about to get rendered and holds a reference to the used model.

```csharp
public async Task<IViewComponentResult> InvokeAsync(string widgetZone, object model)
```

In case you're handling multiple widget zones and need to differentiate between them (or you don't want to live on the rebellious side of life), you might add an if- or a switch-block for `widgetZone`.

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

var productModel = (ProductDetailsModel) model;

var product = await _db.Products.FindByIdAsync(productModel.Id);

var value = product.GenericAttributes.Get<string>("HelloWorldMyTabValue");
```

And finally you create the PublicInfoModel and return the View.

```csharp
var publicInfoModel = new PublicInfoModel
{
	MyTabValue = value
};

return View(publicInfoModel);
```

The final code look like this:

{% code title="ViewComponents.cs" %}
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

		var productModel = (ProductDetailsModel) model;
		var product = await _db.Products.FindByIdAsync(productModel.Id);
		var value = product.GenericAttributes.Get<string>("HelloWorldMyTabValue");
		
		var publicInfoModel = new PublicInfoModel
		{
			MyTabValue = value
		};

		return View(publicInfoModel);
	}
}
```
{% endcode %}

## Adding a View

1. Right click on the **Views** folder in the Solution Explorer.
2. Add a new folder. According to our guidelines we call it **Shared**.
3. Add a new sub folder. According to our guidelines we call it **Components**.
4. Add a new sub folder. According to our guidelines we call it _**ModuleName**_.
5. Place a new _Razor View_ called **Default.cshtml** in this folder.

{% hint style="warning" %}
Don't forget to set the file properties to _**Content**_ and _**Copy if newer**_ as you did with [module.json](../tutorials/building-a-simple-hello-world-module.md#adding-module-metadata-module.json)

1. Right click on the Default.cshtml file in the Solution Explorer.
2. Select the **Properties** context item and change


{% endhint %}

Add the following lines for a simple Output:

```cshtml
@model PublicInfoModel

@{
    Layout = "";
}

<span>Widget content: @Model.MyTabValue</span>
```
