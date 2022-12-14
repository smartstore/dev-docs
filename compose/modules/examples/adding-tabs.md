# 🥚 Adding tabs

In _Smartstore_, we use tabs in different places in the backend and frontend. For this purpose we use the `TabTagHelper`.

In a _Razor_ view, the following markup is used for this purpose.

```html
<tabstrip id="my-tab-config" sm-nav-style="Material" sm-nav-position="Top">
    <tab sm-title="Tab title 1" sm-selected="true">
        Tab content 1
    </tab>
    <tab sm-title="Tab title 2">
        Tab content 2
    </tab>
</tabstrip>
```

You can use the _TagHelper_ in any view. To do this, you must make the tag helper known in the view. To do this, either add the following line to the view or use the `ViewImports.cshtml` file in the root of the views folder.

```html
@addTagHelper Smartstore.Web.TagHelpers.Shared.*, Smartstore.Web.Common
```

### Add individual tabs to existing tabstrips

If you as a developer are faced with the task to extend an existing entity e.g.: _Products_, _Categories_ or _Manufacturers_, you should not do this in the core code itself but attach to the tab from a module. To demonstrate this, in the course of this tutorial we will extend the _HelloWorld_ module we created in the last tutorial.

To do this, we'll add a new `Events.cs` class in the root of the module. When a tabstrip is created, the TagHelper fires the event `TabStripCreated`. The event message contains everything to add a custom tab to the tab strip. The code to add a custom tab in the product detail configuration in the admin area looks like this:

```csharp
using System.Threading.Tasks;
using Smartstore.Events;
using Smartstore.Web.Modelling;
using Smartstore.Web.Rendering.Events;

namespace MyOrg.HelloWorld
{
    public class Events : IConsumer
    {
        public async Task HandleEventAsync(TabStripCreated eventMessage)
        {
            var tabStripName = eventMessage.TabStripName;

            if (tabStripName == "product-edit")
            {
                var entityId = ((TabbableModel)eventMessage.Model).Id;
                // Add in a custom tab
                await eventMessage.TabFactory.AppendAsync(builder => builder
                    .Text("My Tab")
                    .Name("tab-MyTab")
                    .Icon("star", "bi")
                    .LinkHtmlAttributes(new { data_tab_name = "MyTab" })
                    .Action("AdminEditTab", "HelloWorldAdmin", new { entityId })
                    .Ajax());
            }
        }
    }
}
```

First, let's see which tab strip it is. The tab we are interested in has the id `product-edit`. This is available in the event message as `TabStripName`. In the event message the model of the containing view is also supplied. Thus we have e.g. access to the id of the entity for which the detail view was requested.

Using the `TabFactory` of the event message we can inject a new tab.

| Method                 | Description                                                                        |
| ---------------------- | ---------------------------------------------------------------------------------- |
| **Text**               | The text of the tab item that appears in the tab strip                             |
| **Name**               | Unique name/id of tab item                                                         |
| **Icon**               | The icon of the tab                                                                |
| **LinkHtmlAttributes** | HTML attributes to be added to the tab's link                                      |
| **Action**             | The MVC `action` that should be invoked to display the tab                         |
| **Ajax**               | Specifies whether the tab should be reloaded via Ajax when the tab link is clicked |

Now that the tab has been added, we need to add the action that will control the tab. To do this, we open the admin controller and add the following action:

```csharp
public async Task<IActionResult> AdminEditTab(int entityId)
{
    var product = await _db.Products.FindByIdAsync(entityId, false);

    var model = new AdminEditTabModel
    {
        EntityId = entityId,
        MyTabValue = product.GenericAttributes.Get<string>("HelloWorldMyTabValue")
    };
    
    ViewData.TemplateInfo.HtmlFieldPrefix = "CustomProperties[MyTab]";
    return View(model);
}
```

The value with which the model is filled is taken from the `GenericAttributes` property of the product, since this has not yet been saved, it is empty at the present time. More about this later.

{% hint style="info" %}
To learn more about generic attributes, please refer to [generic-attributes.md](../../../framework/advanced/generic-attributes.md "mention")
{% endhint %}

Here we use the `SmartDbContext` to get the instance of the product we just edited. Therefore we need to make the `SmartDbContext` known to the controller via dependency injection. So we add the following code at the very top:

```csharp
private readonly SmartDbContext _db;

public HelloWorldAdminController(SmartDbContext db)
{
    _db = db;
}
```

Since we use a model with two simple properties in the action we just add and return a view we have to create this next.

The `AdminEditTabModel.cs` class belongs in the _Models_ folder, of course.

```csharp
using Smartstore.Web.Modelling;

namespace MyOrg.HelloWorld.Models
{
    [CustomModelPart]
    public class AdminEditTabModel : ModelBase
    {
        public int EntityId { get; set; }

        [LocalizedDisplay("Plugins.MyOrg.HelloWorld.MyTabValue")]
        public string MyTabValue { get; set; }
    }
}

```

Please also place the localized values for this model in the corresponding XML files. We have already explained how to do this in the [building-a-simple-hello-world-module.md](../tutorials/building-a-simple-hello-world-module.md "mention") tutorial.

The view for the model of course belongs in the view folder _HelloWorldAdmin_ and contains the following code:

```html
@model AdminEditTabModel

@{
    Layout = "";
}

<input type="hidden" name="CustomProperties[MyTab].__Type__" value="@Model.GetType().AssemblyQualifiedName" />
<input type="hidden" asp-for="EntityId" />

<div class="adminContent">
    <div class="adminRow">
        <div class="adminTitle">
            <smart-label asp-for="MyTabValue" />
        </div>
        <div class="adminData">
            <input asp-for="MyTabValue" />
            <span asp-validation-for="MyTabValue"></span>
        </div>
    </div>
</div>
```

When the project is now compiled and the product configuration is opened in the admin area, the tab that was added by the module appears here.

Next, we make sure that the value that is entered is also saved when the product is saved. For this we listen to the `ModelBoundEvent`, which is published whenever a form was posted and the MVC model binder bound the model. To find all the places where the event is pubslihed, search throughout the solution for the following code:

```csharp
EventPublisher.PublishAsync(new ModelBoundEvent
```

The code to save the value of our tab belongs to the class `Events.cs` looks like this:

```csharp
public async Task HandleEventAsync(ModelBoundEvent message, SmartDbContext db)
{
    if (!message.BoundModel.CustomProperties.ContainsKey("MyTab"))
        return;

    if (message.BoundModel.CustomProperties["MyTab"] is not AdminEditTabModel model)
        return;

    var product = await db.Products.FindByIdAsync(model.EntityId);
    product.GenericAttributes.Set("HelloWorldMyTabValue", model.MyTabValue);

    await db.SaveChangesAsync();
}
```

The value is stored here in the `GenericAttributes` of the product. A `GenericAttribute` is a separate entity with the help of which any simple values can be stored for each entity. For more complex data structures you should provide your own domain object in your plugin.

{% hint style="info" %}
To learn more about events, please refer to [events.md](../../../framework/platform/events.md "mention")
{% endhint %}

### Download

{% file src="../../../.gitbook/assets/MyOrg.HelloWorldTabs.rar" %}
