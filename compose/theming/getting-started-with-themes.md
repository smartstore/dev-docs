# Getting started with themes

## Overview

A Smartstore-Theme is a collection of sass files, razor views, images and scripts. In short, everything you need to create websites. Themes can be selected and customized by the store owners using the Theme Configurator (Admin > Configuration > Themes). Operators can modify them by configuring theme variables in the Theme Configurator. It is possible to set differing colors, font sizes, margins and much more.

We put a lot of effort into making the Theming Engine easy, flexible and convenient to use. By using techniques such as multi-level theme inheritance, an integrated Sass compiler (which automatically translates all changes made to Sass files into CSS at runtime in an intelligent and highly performant way), we have managed to make creating themes in Smartstore very easy.

Thanks to **multi-level theme inheritance**, it is possible to inherit from a base theme or from a theme that has inherited from another theme. This way, there is no need to start from scratch when developing a theme. Just use the existing components and change only what needs to be changed as you build the theme.

{% hint style="info" %}
New themes should always be derived from the Flex base theme or from a theme originally derived from Flex.
{% endhint %}

A theme provides **variables** that can be configured by the end user. These are automatically translated into Sass variables and can be used in custom Sass files.

Sass files are automatically compiled at runtime using the **built-in Sass parser**. Razor views are also **compiled at runtime**. A built-in file watcher keeps track of all changes made to Sass and Razor files. When a Razor file is changed, the Razor views are recompiled in the background. When a Sass file is changed, the CSS is regenerated. In both cases, the cache is cleared. Simply refresh the browser page while the application is running to see changes to Sass files and Razor views live.

To keep static files as small as possible, Smartstore minifies JavaScript, Sass, and CSS files. Files are packaged into **bundles**, which combine multiple physical files of a web project into one and are minified.

Vendor-specific prefixes are added to CSS declarations in Sass files by an **autoprefixer**.

Smartstore is built using the [MVC-Pattern](https://learn.microsoft.com/en-us/aspnet/core/mvc/overview?view=aspnetcore-7.0). This pattern specifies that the HTML output is provided by views. Views are Razor files located in the subdirectories of the web project's Views directory. They can be easily overwritten at the theme level without having to worry about preparing the model or deploying actions.

Even more developing convenience is provided by 3rd party components. Among others, Smartstore has integrated:

* **Bootstrap**: Very useful for creating HTML structures with its classes.
* **jQuery**: Simplifies DOM element selection and provides browser polyfills.
* **Modern icon libraries like Font Awesome**: Provide a sophisticated and consistent look.

## Anatomy of a theme

* Where are themes located?
* File organization table, similar to [this chapter](../modules/getting-started-with-modules.md#files-and-folders-best-practices).

## Runtime compilation

* Razor runtime compilation
* Sass runtime compilation
  * AutoPrefixer in production (not in Debug)
* Any change to an included file triggers recompilation
* Cache, DiskCache

## Libraries

* Bootstrap
* FontAwesome
* Fontastic
* jQuery
* etc.
