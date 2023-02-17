# Theme inheritance

## Overview

Smartstore implements an inheritance system for themes. This makes it possible to inherit from other existing themes. All files and theme variables of the base theme are included and can be overwritten in the inheriting theme. This way, when you create a theme, you do not have to implement everything from scratch. Only the relevant parts are changed.

Thanks to multi-level inheritance, a theme can inherit from a theme that has already inherited from another theme. Smartstore's theme engine tries to find the actual physical file by determining all corresponding paths in the hierarchy chain of themes.

## Composite File System

Files can be overwritten at the theme level. It does not matter if the file to be overwritten exists in the base theme. It is only important that the file exists in the views directory or in the wwwroot directory of the Smartstore.Web project.

We have structured our CSS, Javascript, and Razor View files in a very granular way, so in most cases you will only need to overwrite a few files to change the design of a particular area. For example, if you want to completely redesign the footer and omit any existing CSS, create a `_footer.scss` in the _wwwroot_ directory of your theme and add your own CSS declarations. The file in the wwwroot directory of the base theme will be completely ignored from then on. Your theme file will be used instead.

If you want to change the HTML structure of the footer, first find the Razor view that is responsible for rendering in the Smartstore.Web project. In this case, it is _\Views\Shared\Components\Footer\Default.cshtml_.

Copy this file to the exact same path in your theme. Now you can isolate your changes in this file without encountering merge conflicts during upgrades. The file in the _Smartstore.Web_ project is now completely ignored. The theme file is used instead.

{% hint style="info" %}
If you work with an isolated theme and make graphical changes to the layout of your store, you have the advantage of being able to easily upgrade the base theme to a higher version when you upgrade. You will have the latest code from the higher version, and your changes will remain largely the same as before the update, since CSS selectors rarely change.

However, before overwriting a view, always ask yourself if you can't get the same result using CSS.
{% endhint %}

## Variable inheritance

Theme variables can be overridden in the derived theme by adding a Var node of the same name to the `theme.config` Vars node.

Example:

```xml
<Var name="border-radius" type="String">0.25rem</Var>
```

## View resolution

View resolution in ASP.NET Core MVC is the process of mapping a view name to a physical file on disk. It determines which view should be returned to the browser in response to a request.

We've improved the process of finding views to search in the _Views_ directory of the theme being used, as well as in the base themes. If no views are found in these theme directories, a fallback is made to the _Views_ directory of the Smartstore.Web project.

### View resolution pipeline

Below is the exact order in which a Razor view is searched in the file system.

```
General:
Own Theme > Views/{ControllerName}/{ActionName}.cshtml
Own Theme > Views/Shared/{ActionName}.cshtml
Base Themes > Views/{ControllerName}/{ActionName}.cshtml
Base Themes > Views/Shared/{ActionName}.cshtml
SmartStore.Web > Views/{ControllerName}/{ActionName}.cshtml
SmartStore.Web > Views/Shared/{ActionName}.cshtml

Modules:
Own Theme > Modules/MyModulePath/Views/{ControllerName}/{ActionName}.cshtml
Own Theme > Modules/MyModulePath/Views/Shared/{ActionName}.cshtml
	    Modules/MyModulePath/Views/{ControllerName}/{ActionName}.cshtml
	    Modules/MyModulePath/Views/Shared/{ActionName}.cshtml

Partials & Layouts:
Own Theme > Views/{ControllerName}/Partials/{ActionName}.cshtml
Own Theme > Views/Shared/Partials/{ActionName}.cshtml
Base Themes > Views/{ControllerName}/Partials/{ActionName}.cshtml
Base Themes > Views/Shared/Partials/{ActionName}.cshtml
SmartStore.Web > Views/{ControllerName}/Partials/{ActionName}.cshtml
SmartStore.Web > Views/Shared/Partials/{ActionName}.cshtml

Own Theme > Modules/MyModulePath/Views/{ControllerName}/Partials/{ActionName}.cshtml
Own Theme > Modules/MyModulePath/Views/Shared/Partials/{ActionName}.cshtml
	    Modules/MyModulePath/Views/{ControllerName}/Partials/{ActionName}.cshtml
	    Modules/MyModulePath/Views/Shared/Partials/{ActionName}.cshtml

Own Theme > Views/{ControllerName}/Layouts/{ActionName}.cshtml
Own Theme > Views/Shared/Layouts/{ActionName}.cshtml
Base Themes > Views/{ControllerName}/Layouts/{ActionName}.cshtml
Base Themes > Views/Shared/Layouts/{ActionName}.cshtml
SmartStore.Web > Views/{ControllerName}/Layouts/{ActionName}.cshtml
SmartStore.Web > Views/Shared/Layouts/{ActionName}.cshtml

Own Theme > Modules/MyModulePath/Views/{ControllerName}/Layouts/{ActionName}.cshtml
Own Theme > Modules/MyModulePath/Views/Shared/Layouts/{ActionName}.cshtml
	    Modules/MyModulePath/Views/{ControllerName}/Layouts/{ActionName}.cshtml
	    Modules/MyModulePath/Views/Shared/Layouts/{ActionName}.cshtml
```

### Localized Views

Localized views can also be stored in Smartstore. To do this, the `EnableLocalizedViews` setting in the `appsettings.json` file must be set to true. By default, this is set to false for performance reasons.

Smartstore's view engine first tries to find a localized view when a request is made. Localized views must include the appropriate SEO code for the language being used in the filename (e.g.: `MyView.de.cshtml` > `MyView.cshtml`).
