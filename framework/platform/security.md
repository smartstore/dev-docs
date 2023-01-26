# Security

## Overview

* Extensible permissions system where permssions are assigned to customer roles, which than are assigned to customers.
* Permissions are organised hierarchically. They can inherit from parent permissions. So a permission can have three statuses: _allowed_, _not allowed_ or _inherited_.
* Customer roles including assigned permissions can automatically assigned to customers by rule sets.

## Authorization

Use [IPermissionService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Security/Services/IPermissionService.cs) to check whether a given permission is granted to a customer. It expects strings as the name of the permission to be checked. Several constants are provided through `Permissions` class for this purpose. Example:

```csharp
await _permissionService.AuthorizeAsync(Permissions.System.ScheduleTask.Execute);
```

Action methods use the `PermissionAttribute` instead of [IPermissionService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Security/Services/IPermissionService.cs). Example:

```csharp
[Permission(Permissions.Catalog.Product.Read)]
public async Task<IActionResult> Edit(int id)
{
    //...
}
```

An `AccessDeniedException` is thrown when the permission is not granted. In case of an AJAX request, a message notification is shown and an alert is returned if HTML response is expected or a JSON object otherwise.

## Add custom permissions

The permission system can be extended to include custom permissions using [IPermissionProvider](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Security/Services/IPermissionProvider.cs). See the [DevToolsPermissionProvider](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Modules/Smartstore.DevTools/Permissions.cs) example. It is recommed to use singular for permission names and to define a root permission _Self_ that doesn't contain any dot by convention:

{% code title="MegaSearch root permission" %}
```csharp
public const string Self = "megasearch";
```
{% endcode %}

The localisation is done via string resources and also by a convention. The string resource key is `Permissions.DisplayName.<PermissionName>` for the core and `Modules.Permissions.DisplayName.<PermissionName>` for modules. In most cases only the root permission _Self_ is localised because other permissions are already localised by the core like _read_, _update_, _execute_ etc. See `PermissionService._displayNameResourceKeys` for a complete list.

HINT: module permissions are automatically added when the module is installed and removed when uninstalling the module. Module developers do not need to do anything else here.

TIP: for an [IMenuProvider](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Content/Menus/Services/MenuProviders/IMenuProvider.cs) such as the `AdminMenuProvider`, one or more permissions can be applied via `MenuItemBuilder.PermissionNames`. The related link in the admin menu is then only displayed if the admin has been granted the permission.

## Implement an external authentication
