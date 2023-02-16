# 🥚 Getting started with themes

## Overview

A Smartstore theme is a collection of Sass files, Razor views, images and scripts. In short, everything you need to create websites. Themes can be selected and customized by the store owner using the Theme Configurator (**Admin / Configuration / Themes**). Operators can modify them by configuring theme variables in the Theme Configurator. It is possible to set differing colors, font sizes, margins and much more.

We put a lot of effort into making the theming engine easy, flexible and convenient to use. By using techniques such as multi-level theme inheritance, an integrated Sass compiler (which automatically translates all changes made to Sass files into CSS at runtime in an intelligent and highly performant way), we have managed to make creating themes in Smartstore very easy.

Thanks to **multi-level theme inheritance**, it is possible to inherit from a base theme or from a theme that has inherited from another theme. This way, there is no need to start from scratch when developing a theme. Just use the existing components and change only what needs to be changed as you build the theme.

{% hint style="info" %}
New themes should always be derived from the Flex base theme or from a theme originally derived from Flex.
{% endhint %}

A theme provides **variables** that can be configured by the end user. These are automatically translated into Sass variables and can be used in custom Sass files.

Sass files are automatically compiled at runtime using the **built-in Sass parser**. Razor views are also **compiled at runtime**. A built-in `FileWatcher` keeps track of all changes made to Sass and Razor files. When a Razor file is changed, the Razor views are recompiled in the background. When a Sass file is changed, the CSS is regenerated and the cache is cleared. Simply refresh the browser page while the application is running to see changes to Sass files and Razor views live.

To keep static files as small as possible, Smartstore minifies JavaScript, Sass, and CSS files. Files are packaged into **bundles**, which combine multiple physical files of a web project into one and are minified.

Vendor-specific prefixes are added to CSS declarations in Sass files by an **autoprefixer**.

Smartstore is built using the [MVC-Pattern](https://learn.microsoft.com/en-us/aspnet/core/mvc/overview?view=aspnetcore-7.0). This pattern specifies that the HTML output is provided by views. Views are Razor files located in the subdirectories of the web project's _Views_ directory. They can be easily overwritten at the theme level without having to worry about preparing the model or deploying actions.

Even more developing convenience is provided by 3rd party components. Among others, Smartstore has integrated:

* [Bootstrap](getting-started-with-themes.md#bootstrap): Very useful for creating HTML structures with its classes.
* [jQuery](getting-started-with-themes.md#jquery): Simplifies DOM element selection and provides browser polyfills.
* [Modern icon libraries](getting-started-with-themes.md#icon-libraries) like [Font Awesome](getting-started-with-themes.md#font-awesome): Provide a sophisticated and consistent look.

## Anatomy of a theme

Themes are located in the Smartstore.Web project in the _Themes_ directory. Any folder in here that contains a `theme.config` file is treated as a theme.

### Files & Folders Best Practices

There are some conventions for organizing files and directories within a theme. While there is no requirement to follow them, it makes things predictable and easier to maintain.

The following is an exhaustive list of files and directories.

| Entry                              | Description                                         |
| ---------------------------------- | --------------------------------------------------- |
| **wwwroot**                        | Static files (including Sass files)                 |
| **wwwroot/images**                 | Images                                              |
| **wwwroot/css**                    | CSS files                                           |
| **wwwroot/js**                     | Javascript files                                    |
| **Views**                          | Razor view / template files                         |
| theme.config                       | Required. Theme metadata manifest.                  |
| Views/Shared/ConfigureTheme.cshtml | Configuration view for configuring theme variables. |

{% hint style="info" %}
For more information about file organization, see [Files & Folders: Best Practices](../modules/getting-started-with-modules.md#files-and-folders-best-practices).
{% endhint %}

## Runtime compilation

### Razor runtime compilation

Razor runtime compilation is a feature in ASP.NET Core that allows Razor pages to be dynamically compiled, rather than being compiled when the application is compiled. This allows changes to Razor pages to be applied in real time without having to restart the application.

Smartstore has a setting for this feature that is enabled by default. If you want to disable it, open the `appsettings.json` file in the root of the Smartstore.Web project and change the value of the `EnableRazorRuntimeCompilation` property.

### Sass runtime compilation

[Sass](https://sass-lang.com/) is a CSS preprocessor, which means it extends the CSS language by adding features like variables, mixins, functions, and many other techniques. These allow you to create CSS that is more maintainable, themable, and extensible.

In Smartstore, CSS declarations are declared in in `.scss` files. This way Sass variables and functions can be used. Sass is automatically translated into CSS by the built-in Sass parser, so any browser can render it, unlike Sass.

Smartstore's built-in `FileWatcher` keeps track of all changes made to the included Sass files while the application is running. When changes are made, the cache is automatically cleared and the Sass files are retranslated into CSS. This provides a convenient, time-saving way to check for CSS changes on page refresh without having to restart the application.

### AutoPrefixer

[Vendor prefixes](https://developer.mozilla.org/en-US/docs/Glossary/Vendor\_Prefix) are a part of CSS that can be added to certain properties and values. They enable experimental, non-standard features in different browsers. For example, the `-webkit-` prefix is used for properties and values supported by WebKit browsers (Google, Safari, ...), and Mozilla Firefox uses the `-moz-` prefix.

Without using a tool like CSS Autoprefixer, you would have to take care of adding the correct prefixes yourself. This can be tedious and error-prone.

To ensure compatibility with different browsers, Smartstore has a built-in CSS autoprefixer. It is enabled in production mode, but not in debugging mode. This allows you to write CSS code without having to add vendor prefixes yourself, as the tool will add them automatically. It uses the latest available [Can I Use](https://caniuse.com/) data to add the prefix to each corresponding CSS property and value.

By using the CSS Autoprefixer, developers can rest assured that all CSS styles will display correctly in all major browsers. They can focus on designing the site without worrying about compatibility.

### Cache, DiskCache

Created assets are cached in RAM and on disk. This keeps the whole process highly performant and delays page rendering by only a few milliseconds when regenerating CSS files. The cache is automatically invalidated when an included file changes.

## Libraries

### jQuery

### Bootstrap

## Icon Libraries

### Font Awesome

#### Font Awesome Pro

### Bootstrap Icons

### Fontastic
