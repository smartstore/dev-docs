---
description: Inject content into zones
---

# 👍 Widgets

## Overview

* Widgets are pieces/snippets of HTML content that can be injected into particular zones
* In modular applications like Smartstore, the ability to inject external content into existing pages is indispensable....
* Common scenarios for this would be to:...&#x20;
  * include some extra JavaScript code in the HEAD section
  * add extra menu items
  * implement a custom sidebar
  * add buttons to data grids, or pages in general
  * add more content to product listings and details
  * add more content to the shopping cart
  * and many many more
* In ASP.NET Core, _view components_ or _partial views_ are kind of widgets.
* In Smartstore, we have an abstraction for widgets: [WidgetInvoker](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Widgets/WidgetInvoker.cs): because view components and partials are - technically speaking - two different things. But we need to unify them to be able to feed widgets from different content sources.
  * `HtmlWidgetInvoker`: renders any `IHTMLContent` instance
  * `ComponentWidgetInvoker`: invokes and renders a view component
  * `PartialViewWidgetInvoker`: invokes and renders a partial view
  * INFO: You can implement a custom widget invoker by deriving from `WidgetInvoker` and overriding the `InvokeAsync` method.
* Furthermore, the `widget` Tag Helper allows you to compose HTML content in any view template and to inject it into any zone (much like the _section_ directive in ASP.NET)

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

```cshtml
@if (Display.ZoneHasContent("wishlist_items_top")) 
{
    <div class="some-wrapper">
        <zone name="wishlist_items_top" />
    </div>
}
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

## Widget invoker

* Unifies view component, partial view and `IHtmlContent`
* Before injecting or rendering a widget we have to create an instance:

```csharp
// From view component: by type.
// -----------------------------
var widget = new ComponentWidgetInvoker<WeatherViewComponent>() 
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
var widget = new ComponentWidgetInvoker("Weather", "My.Module", new 
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
var widget = new PartialViewWidgetInvoker("Weather", "My.Module");

// From any HTML content.
// ----------------------
var tag = new TagBuilder("div");
tag.InnerHtml.SetContent("Lorem ipsum");
var widget = new HtmlWidgetInvoker(tag);
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
    
    public void OnResultExecuting(ResultExecutingContextfilterContext)
    {
        _widgetProvider.RegisterWidget(
            "end", 
            new ComponentWidgetInvoker("CookieManager", null));
    }
    
    public void OnResultExecuted(ResultExecutedContext filterContext)
    {
        // Too late for widgets here: page is rendered already.
    }
}
```

* The `RegisterWidget` method has also some overloads that let you pass an array of zone names or even a regular expression.
* `HasContent` method lets you check whether a zone contains at least one injected widget
* `GetWidgets` method lets you enumerate all injected widgets in a given zone

## Static widgets (aka widget providers)

* The legacy way of handling widgets
* [IWidget](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Widgets/IWidget.cs) interface makes an application feature provider a widget (see [modularity-and-providers.md](../platform/modularity-and-providers.md "mention") for more info about providers)
* The provider class defines _what_ to render (`GetDisplayWidget` method), and _where_ to render it (`GetWidgetZones` method).
* Example: [Google Analytics Module](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Modules/Smartstore.Google.Analytics/Module.cs) injects script content into the _head_ zone.
* INFO: Static widgets require explicit activation by the user in the backend (**CMS / Widgets**), otherwise they are not rendered. But by decorating a non-widget provider with the [DependentWidgetsAttribute](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Widgets/DependentWidgetsAttribute.cs), you can specify widget providers, which should automatically get (de)activated when the provider gets (de)activated. This is useful in scenarios where separate widgets are responsible for the displaying of provider data.

## List of all core widget zone names

A list of common zone names are recorded in the file _App\_Data/widgetzones.json_.&#x20;
