---
description: Inject content into zones
---

# 🥚 Widgets

## Overview

Widgets are pieces/snippets of HTML content that can be injected into the [widget zones](widgets.md#zones) of a page. The ability to inject external content into existing pages is essential for modular applications like Smartstore. Common scenarios include:

* Include additional JavaScript code into the HEAD section of your page.
* Add more menu items to the navigation bar.
* Implement a custom sidebar.
* Add more content to:
  * data grids or pages
  * product listings and details
  * the shopping cart

Smartstore has a widget abstraction called [Widget](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Widgets/Widget.cs). This is because ASP.NET Core _view components_ and _partial views_ are technically two different things, though they behave very similarly to widgets in Smartstore. In order to use them in Smartstore, they need to be unified so that widgets can be fed from different content sources.

* `HtmlWidget`: Renders any `IHTMLContent` instance
* `ComponentWidget`: Invokes and renders an ASP.NET Core view component
* `PartialViewWidget`: Invokes and renders an ASP.NET Core partial view

{% hint style="info" %}
By deriving from `Widget` and overriding the `InvokeAsync` method, you can implement a custom widget class. You can just return your content directly if the output is simple enough. Otherwise, implement the rendering portion as an [`IWidgetInvoker<TWidget>`](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Widgets/Services/IWidgetInvoker.cs). Then the `Widget.InvokeAsync` method should resolve and call the invoker.
{% endhint %}

In addition, the [widget](widgets.md#widget-tag-helper) Tag Helper allows you to compose HTML content in any view template and inject it into any zone, much like the _section_ directive in ASP.NET.

## Zones

* Zones allow you to define spots in any view file where widgets may inject custom markup
* They are similar to ASP.NET _Sections_, but much more powerful
* The Smartstore view templates contain hundreds of zones. See below for a complete list.
* You can define zones in any Razor view with the `zone` Tag Helper.

```cshtml
<zone name="wishlist_items_top" />
```

* Zones can also contain default content

```cshtml
@* 
    The "replace-content" attribute specifies whether
    the default content should be removed if at least one
    widget is rendered in the zone:
        true: yes, remove default content.
        false: no, keep content and place widget before or after 
            content, according to "WidgetInvoker.Prepend" option.
*@
<zone name="wishlist_items_top" replace-content="true">
    <div>Lorem</div>
    <div>Ipsum</div>
</zone>
```

* Some HTML tags can also act like a zone: `div`, `span`, `p`, `section`, `aside`, `header`, `footer`

```cshtml
@* 
    The "remove-if-empty" attribute specifies whether
    to remove the tag when it has no content. Default: false.
*@
<div name="wishlist_items_top" remove-if-empty="true"></div>
```

* Sometimes you may need to check whether a zone has content _before_ declaring the `zone` tag. This is the case if the `zone` tag is wrapped, and the wrapper HTML output should be suppressed if no content exists:

```aspnet
@if (await Display.ZoneHasContentAsync("wishlist_items_top")) 
{
    <div class="some-wrapper">
        <zone name="wishlist_items_top" />
    </div>
}
```

Another way to suppress surrounding content: `sm-suppress-if-empty-zone` Tag Helper. This sort of pre-renders a given zone, and, if the zone content is empty or whitespace, suppresses the output of the parent tag:

```aspnet
<div sm-suppress-if-empty-zone="wishlist_items_top" class="some-wrapper">
    <div class="inner-wrapper m-4">
        <zone name="wishlist_items_top" />
    </div>
</div>
```

## Widget Tag Helper

* The `widget` Tag Helper allows you to compose HTML content in any view template and to inject it into any zone (much like the _section_ directive in ASP.NET)

```cshtml
@*
    target-zone: Required,
    order: Sort order within target zone. Optional, 
    prepend: Whether to insert BEFORE existing zone content (instead AFTER). Optional,
    key: When set, ensures uniqueness within a particular zone. Optional.
*@
<widget target-zone="wishlist_items_top" order="10" prepend="false" key="MyWidgetInstanceKey">
    @await Component.InvokeAsync("MyComponent1")
    <div>Lorem ipsum</div>
    @await Component.InvokeAsync("MyComponent2")
</widget>
```

* Some HTML tags can also act like widgets: `div`, `span`, `section`, `form`, `script`, `style`, `link`, `meta`, `ul`, `ol`, `svg`, `img`, `a` &#x20;
* The attribute names must be prefixed with **sm-** in this case
* Unlike `widget`,  a _widgetized_ HTML tag is moved completely to its designated zone, whereas `widget` moves the child content only (the root tag `widget` is removed from output).

```cshtml
@*
    Specifying "data-origin" attribute is good practice, because
    it gives you an idea about where a piece of code came from
    while inspecting the document markup in a browser. 
*@
<script sm-target-zone="scripts" data-origin="blog-list">
    $(function () {
        $(".blogposts").masonryGrid(".bloglist-item");
    });
</script>

<style type="text/css" sm-target-zone="stylesheets" >
    .some-selector { }
</style>

<script src="~/bundle/js/fileuploader.js" 
    sm-target-zone="scripts" 
    sm-key="fileuploader"></script>
```

## Widget class

* Unifies view component, partial view and `IHtmlContent`
* Before injecting or rendering a widget we have to create an instance:

```csharp
// From view component: by type.
// -----------------------------
var widget = new ComponentWidget<WeatherViewComponent>() 
{ 
    // sort order within target zone. Optional.
    Order = 10,
    // Whether to insert BEFORE existing zone content (instead AFTER). Optional.  
    Prepend = false,
    // When set, ensures uniqueness within a particular zone. Optional. 
    Key = "MyWidgetInstanceKey" 
};

// From view component: by name + arguments passed.
// ------------------------------------------------
// The second parameter "My.Module" is the system name of the module
// where the view component is located. This must be specified, 
// otherwise component resolution by name will fail.
var widget = new ComponentWidget("Weather", "My.Module", new 
{
    // Pass arguments to the view component's "Invoke" method
    arg1 = "Hello",
    arg2 = "World"
});

// From partial view by name.
// --------------------------
// The second parameter "My.Module" is the system name of the module
// where the partial view is located. This must be specified, 
// otherwise view resolution will fail.
var widget = new PartialViewWidget("Weather", "My.Module");

// From any HTML content.
// ----------------------
var tag = new TagBuilder("div");
tag.InnerHtml.SetContent("Lorem ipsum");
var widget = new HtmlWidget(tag);
```

* The most common way to inject a widget into a zone is by using [IWidgetProvider](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Widgets/Services/IWidgetProvider.cs). It is a request scoped registrar for widget instances.

```csharp
internal class CookieConsentFilter : IResultFilter
{
    private readonly IWidgetProvider _widgetProvider;

    public CookieConsentFilter(IWidgetProvider widgetProvider)
    {
        _widgetProvider = widgetProvider;
    }
    
    public void OnResultExecuting(ResultExecutingContextFilterContext)
    {
        _widgetProvider.RegisterWidget(
            "end", // The zone name to render widget into
            new ComponentWidget("CookieManager", null));
    }
    
    public void OnResultExecuted(ResultExecutedContext filterContext)
    {
        // Too late for widgets here: page is rendered already.
    }
}
```

* The `RegisterWidget` method has also some overloads that let you pass an array of zone names or even a regular expression.
* `HasWidgets` method lets you check whether a zone contains at least one injected widget
* `GetWidgets` method lets you enumerate all injected widgets in a given zone

## Static widgets (aka widget providers)

* The legacy way of handling widgets
* [IActivatableWidget](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Widgets/IActivatableWidget.cs) interface makes an application feature provider a widget (see [modularity-and-providers.md](../platform/modularity-and-providers.md "mention") for more info about providers)
* The provider class defines _what_ to render (`GetDisplayWidget` method), and _where_ to render it (`GetWidgetZones` method).
* Example: [Google Analytics Module](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Modules/Smartstore.Google.Analytics/Module.cs) injects script content into the _head_ zone.
* INFO: Static widgets require explicit activation by the user in the backend (**CMS / Widgets**), otherwise they are not rendered. But by decorating a non-widget provider with the [DependentWidgetsAttribute](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Widgets/DependentWidgetsAttribute.cs), you can specify widget providers, which should automatically get (de)activated when the provider gets (de)activated. This is useful in scenarios where separate widgets are responsible for the displaying of provider data.

## List of all core widget zone names

A list of common zone names are recorded in the file _App\_Data/widgetzones.json_.&#x20;
