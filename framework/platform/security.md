# 🥚 Security

## Overview

* Extensible, hierarchically organized permissions system where permissions are assigned to customer roles, which than are assigned to customers.
* Customer roles (including assigned permissions) can automatically be assigned to customers by rule sets.
* [IAclRestricted](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Security/Domain/IAclRestricted.cs) marks an entity with restricted access rights. Only customers assigned to specific customer groups can see or access it.
* Further security tools like captcha and honeypot.

## Permissions tree

Permissions are organized hierarchically in the form of a tree. A permission is granted when it has either granted itself or one of its parent permissions. The same applies to a permission that is explicitly not permitted. In this way, entire groups of permissions can be activated via a parent permission, where child permissions inherit from their parent. Therefore a permission can have three statuses: _allowed_, _not allowed_ or _inherited_.

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

Use the `NeverAuthorizeAttribute` when the access to the requested entpoint is always permitted. For instance the action method of the login page is decorated with this attribute.

An `AccessDeniedException` is thrown when the permission is not granted. In case of an AJAX request, a message notification is shown and an alert is returned if HTML response is expected or a JSON object otherwise.

## Access control list (ACL)

An entity supports restricted access rights when it implements [IAclRestricted](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Security/Domain/IAclRestricted.cs). It allows to limit an entity to certain customer roles. Only customers who are assigned to one of the associated customer roles can see the product, the category, the menu item or whatever.

Use [IAclService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Security/Services/IAclService.cs) to check whether the customer has the right to access an entity. If you want to only load those entities from the database to which the customer has access to, then use the `ApplyAclFilter` extension method.

Use `IAclService.ApplyAclMappingsAsync` when you want to modify the access rights of a customer to an entity. This is typically done when the entity is updated on its edit page.

## Captcha

Decorate an action method with the `ValidateCaptchaAttribute` to prevent malicious software from accessing your endpoint. The attribute uses the widely used Google reCAPTCHA for protection. CaptchaTagHelper renders the Google widget in your view. Example:

```cshtml
<captcha sm-enabled="Model.DisplayCaptcha" class="form-group" />
```

## Add custom permissions

The permission system can be extended to include custom permissions using [IPermissionProvider](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Security/Services/IPermissionProvider.cs). See the [DevToolsPermissionProvider](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Modules/Smartstore.DevTools/Permissions.cs) example. It is recommended to use singular for permission names and to define a root permission _Self_ that doesn't contain any dot by convention:

{% code title="MegaSearch root permission" %}
```csharp
public const string Self = "megasearch";
```
{% endcode %}

The localization is done via string resources and also by a convention. The string resource key is `Permissions.DisplayName.<PermissionName>` for the core and `Modules.Permissions.DisplayName.<PermissionName>` for modules. In most cases only the root permission _Self_ is localized because other permissions are already localised by the core like _read_, _update_, _execute_ etc. See `PermissionService._displayNameResourceKeys` for a complete list.

HINT: module permissions are automatically added when the module is installed and removed when uninstalling the module. Module developers do not need to do anything else here.

TIP: for an [IMenuProvider](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Content/Menus/Services/MenuProviders/IMenuProvider.cs) such as the `AdminMenuProvider`, one or more permissions can be applied via `MenuItemBuilder.PermissionNames`. The related link in the admin menu is then only displayed if the admin has been granted the permission.

## ~~Implement an external authentication~~

~~External authentication takes place when the user uses their Google, Facebook etc. account to log in so that they do not have to register in the store. Therefore an additional button is dislayed below the login form in frontend.~~

~~Use IExternalAuthenticationMethod to implement an external authentication method and provide a ComponentWidget that renders the button.~~ <mark style="color:red;">Misplaced, more</mark> <mark style="color:red;"></mark>_<mark style="color:red;">Identity</mark>_ <mark style="color:red;"></mark><mark style="color:red;">related...</mark>
