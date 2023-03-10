# 🐥 Filters

Smartstore modules are pure MVC projects. This means that normal action filters, provided by the ASP.NET Core framework, can be implemented. Implementing filters in modules is the best way to extend, intercept and modify existing functionality in Smartstore.

{% hint style="info" %}
For more information, see the [different filters that can be implemented with ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/mvc/controllers/filters?view=aspnetcore-7.0).
{% endhint %}

## Basic example

Let’s say a module needs to render a link in the header navigation of the store. This link leads to a page provided by the module. To do this, create a filter class that renders the link in the `header_menu_special` widget zone.

{% hint style="info" %}
For more information on widgets and widget-zones, see [Widgets](../../framework/content/widgets.md).
{% endhint %}

The filter might look like this:

```csharp
public class MyFilter : IResultFilter
{
    private readonly IWidgetProvider _widgetProvider;   
    private readonly IUrlHelper _urlHelper;

    public MyFilter(IWidgetProvider widgetProvider, IUrlHelper urlHelper)
    {
        _widgetProvider = widgetProvider;
        _urlHelper = urlHelper;
    }

    public void OnResultExecuting(ResultExecutingContext filterContext)
    {
        // Should only run on a full view rendering result or HTML ContentResult.
	if (!filterContext.Result.IsHtmlViewResult())
	{
	    return;
	}

	// Menu item in global header
	var html = $"<a class='menubar-link' href='{_urlHelper.RouteUrl("MyRoute")}'>My Link</a>";
	_widgetProvider.RegisterHtml("header_menu_special", new HtmlString(html), 100);
    }

    public void OnResultExecuted(ResultExecutedContext filterContext)
    {
    }
}
```

### Register a filter in Startup

The \`StartUp' class of the module is used to register a filter.

```csharp
internal class Startup : StarterBase
{
    public override void ConfigureServices(IServiceCollection services, IApplicationContext appContext)
    {
        // ...
        services.Configure<MvcOptions>(o =>
        {
	    o.Filters.Add<MyFilter>();
        });
    }
}
```

Now the filter will be applied to every `Action` in the entire project.

### Conditions

The rendered link in this example should only be displayed in the frontend. Instead of using the filter to control the output by querying different values from the current context, you can register the filter only for relevant controllers and actions using the `AddConditional` method.

Since the link should be rendered in a global location such as the header, you should register the filter for each frontend action. All frontend controllers implement the base controller `PublicController`. This way, the condition for the filter execution can be to run on every `PublicController` action.

A controller that doesn't implement the `PublicController` will never cause the filter to run in this example:

```csharp
o.Filters.AddConditional<MyFilter>(
    context => context.ControllerIs<PublicController>());
```

Filters for specific controllers, like the `ProductController` in the frontend, can be registered in the same way. Since there is one `ProductController` for the frontend and one for the backend, an additional condition must be added to ensure that the correct controller is used.

```csharp
o.Filters.AddConditional<MyFilter>(
    context => context.ControllerIs<ProductController>()
    && context.ControllerIs<PublicController>());
```

If only a specific action is needed to be filtered, the action name can be passed to the `AddConditional` method, using a lambda expression.

```csharp
o.Filters.AddConditional<WidgetZoneFilter>(
    context => context.ControllerIs<ProductController>(x => x.AskQuestion(1)));
```
