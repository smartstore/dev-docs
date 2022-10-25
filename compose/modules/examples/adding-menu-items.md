# 🥚 Adding menu items

{% hint style="info" %}
For a more in-depth view on widgets, zones and invokers, please refer to [Menus](../../../framework/content/menus.md).
{% endhint %}

Menus are a very important tool to structurise Smartstore's interface. Building on the previous tutorial, you will add a menu entry to the admin menu.

## Adding the menu class

First you add the class `AdminMenu.cs` to the root folder of your module.

```csharp
namespace MyOrg.HelloWorld
{
    public class AdminMenu
    {
    }
}
```

### Implement AdminMenuProvider

You need to add the interface `AdminMenuProvider` and implement the method `BuildMenuCore`.

```csharp
public class AdminMenu : AdminMenuProvider
{
    protected override void BuildMenuCore(TreeNode<MenuItem> modulesNode)
    {
    }
}
```

### Create a MenuItem

Next you create a `MenuItem`.

```csharp
var myMenuItem = new MenuItem().ToBuilder()
    .ResKey("Plugins.MyOrg.HelloWorld.MyMenuItem")
    .Icon("gear", "bi")
    .Action("Configure", "HelloWorldAdmin", new { area = "Admin" })
    .AsItem();
```

* `ResKey` is a reference to your localization files.
* `Icon` adds a menu icon.
*   `Action` creates a route to your action, defined in the specified controller.

    This menu will simply point to your `Configure` method from the `HelloWorldAdminController`.

### Add a localization string

Add a new item to your localization, so that the menu text will show.

```xml
<LocaleResource Name="MyMenuItem">
    <Value>Configure Module</Value>
</LocaleResource>
```

### Create TreeNodes

Create a `TreeNode` _menuNode_ from `myMenuItem`.

```csharp
var menuNode = new TreeNode<MenuItem>(myMenuItem);
```

Fetch a reference node from `modulesNode`. In this example the menu id `settings` is used.

```csharp
var refNode = modulesNode.Root.SelectNodeById("settings");
```

{% hint style="info" %}
For more menu ids:

* Open the Smartstore admin page
* Right-Click the desired menu
* Choose the _inspector_
* Look for **`data-id`**
{% endhint %}

Insert `menuNode` after `refNode`.

```csharp
menuNode.InsertAfter(refNode);
```

Now you should see a menu item in the admin configuration menu.

Your code should look something like this:

{% code title="AdminMenu.cs" %}
```csharp
using Smartstore.Collections;
using Smartstore.Core.Content.Menus;
using Smartstore.Web.Rendering.Builders;

namespace MyOrg.HelloWorld
{
    public class AdminMenu : AdminMenuProvider
    {
        protected override void BuildMenuCore(TreeNode<MenuItem> modulesNode)
        {
            var myMenuItem = new MenuItem().ToBuilder()
                .ResKey("Plugins.MyOrg.HelloWorld.MyMenuItem")
                .Icon("gear", "bi")
                .Action("Configure", "HelloWorldAdmin", new { area = "Admin" })
                .AsItem();

            var menuNode = new TreeNode<MenuItem>(myMenuItem);
            var refNode = modulesNode.Root.SelectNodeById("settings");
            menuNode.InsertAfter(refNode);
        }
    }
}
```
{% endcode %}

## Conclusion

In this tutorial you created a menu item and added it to the admin menu.

The code for this module can be downloaded here:

\--Insert File--
