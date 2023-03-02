# 🐣 Getting started with themes

## Overview

A Smartstore theme is a collection of Sass files, Razor views, images and scripts. In short, everything you need to create websites. Themes can be selected and customized by the store owner using the Theme Configurator (**Admin / Configuration / Themes**). Owners can modify them by configuring theme variables in the Theme Configurator. It is possible to set differing colors, font sizes, margins and much more.

We have put a lot of effort into developing the Theming Engine so that creating themes is easy, flexible and convenient to use. In addition, we have managed to make creating themes in Smartstore very easy by using techniques such as:

* Multi-level theme inheritance
* An integrated Sass compiler, that automatically translates all changes made to Sass files into CSS at runtime in an intelligent and highly performant way.
* CSS _Autoprefixer_
* Modern CSS and icon libraries&#x20;
* And many more

Thanks to **multi-level theme inheritance**, it is possible to inherit from a base theme or from a theme that has inherited from another theme. This way, there is no need to start from scratch when developing a theme. Just use the existing components and change only what needs to be changed as you build the theme.

{% hint style="info" %}
New themes should always be derived from the _Flex_ base theme or from a theme originally derived from _Flex_.
{% endhint %}

A theme provides **variables** that can be configured by the end user. These are automatically translated into Sass variables and can be used in custom Sass files.

Sass files are automatically compiled at runtime using the **built-in Sass parser**. Razor views are also **compiled at runtime**. A built-in file watcher keeps track of all changes made to Sass and Razor files. When a Razor file is changed, the Razor views are recompiled in the background. When a Sass file is changed, the CSS is regenerated and the cache is cleared. Simply refresh the browser page while the application is running to see changes to Sass files and Razor views live.

To keep static files as small as possible, Smartstore minifies JavaScript, Sass, and CSS files. Files are packaged into **bundles**, which combine multiple physical files of a web project into one and are minified.

The _Autoprefixer_ adds vendor-specific prefixes to CSS declarations coming from the Sass parser.

Smartstore is built using the [MVC-Pattern](https://learn.microsoft.com/en-us/aspnet/core/mvc/overview?view=aspnetcore-7.0). This pattern specifies that the HTML output is provided by views. Views are Razor files located in the subdirectories of the web project's _Views_ directory. They can be easily overwritten at the theme level without having to worry about preparing the model or implementing actions.

Widely used third-party components make development even easier. Among others, Smartstore has integrated:

* [Bootstrap](getting-started-with-themes.md#bootstrap): Very useful for creating HTML structures with its classes.
* [jQuery](getting-started-with-themes.md#jquery): Simplifies DOM element selection and provides browser polyfills.
* [Modern icon libraries](getting-started-with-themes.md#icon-libraries) like [Font Awesome](getting-started-with-themes.md#font-awesome): Provide a sophisticated and consistent look.

## Anatomy of a theme

Themes are located in the `Smartstore.Web` project in the _Themes_ directory. Any directory in here that contains a `theme.config` file is treated as a theme.

### Files & Folders: Best Practices

There are some conventions for organizing files and directories within a theme. While there is no requirement to follow them, it makes things predictable and easier to maintain.

The following is an exhaustive list of files and directories.

| Entry                                              | Description                                         |
| -------------------------------------------------- | --------------------------------------------------- |
| :file\_folder: **wwwroot**                         | Static files (including Sass files)                 |
| :file\_folder: **wwwroot/images**                  | Images                                              |
| :file\_folder: **wwwroot/css**                     | Static CSS files                                    |
| :file\_folder: **wwwroot/js**                      | JavaScript files                                    |
| :file\_folder: **Views**                           | Razor view / template files                         |
| :file\_cabinet: theme.config                       | Required. Theme metadata manifest.                  |
| :file\_cabinet: Views/Shared/ConfigureTheme.cshtml | Configuration view for configuring theme variables. |

## Runtime compilation

### Razor runtime compilation

Razor runtime compilation is a feature in ASP.NET Core that allows Razor pages to be dynamically compiled, rather than being compiled when the application is compiled. This allows changes to Razor pages to be applied in real time without having to restart the application.

Smartstore has a setting for this feature that is enabled by default. If you want to disable it, open the `appsettings.json` file in the root of the `Smartstore.Web` project and change the value of the `EnableRazorRuntimeCompilation` property.

<mark style="color:blue;">Erläutern: um ein Kompilat zu erzeugen, dass keine präkompilierten Views enthält, muss man in VS die Build-Konfiguration</mark> <mark style="color:blue;"></mark><mark style="color:blue;">**DebugNoRazorCompile**</mark> <mark style="color:blue;"></mark><mark style="color:blue;">wählen. Damit lässt sich die Solution auch sehr viel schneller kompilieren. Aber mit dem Nachteil, dass die Seiten-Ausführungsgeschwindigkeit für den ersten Hit etwas abnimmt (weil im Hintergrund erstmal jene Views kompiliert werden müssen, die gerade benötigt werden). Für</mark> <mark style="color:blue;"></mark><mark style="color:blue;">**Hot Reload**</mark> <mark style="color:blue;"></mark><mark style="color:blue;">während Debugging empfehlen wir definitiv die Build-Konfiguration</mark> <mark style="color:blue;"></mark><mark style="color:blue;">**DebugNoRazorCompile**</mark> <mark style="color:blue;"></mark><mark style="color:blue;">zu wählen, weil</mark> <mark style="color:blue;"></mark><mark style="color:blue;">**Debug**</mark> <mark style="color:blue;"></mark><mark style="color:blue;">sehr sehr langsam ist beim Ermitteln und Anwenden von Code-Änderungen.</mark>

### Sass runtime compilation

[Sass](https://sass-lang.com/) is a CSS preprocessor, which means it extends the CSS language by adding features like variables, mixins, functions, and many other techniques. These allow you to create CSS that is more maintainable, themable and extensible.

Smartstore uses `.scss` files for CSS declarations. They provide a way to use Sass variables and functions. At runtime, Sass is automatically translated into CSS by the built-in Sass parser, which, unlike Sass, can be read by any browser.

Smartstore's built-in file watcher keeps track of all changes made to the included Sass files while the application is running. When a change is detected, the cache is automatically cleared and the Sass files are retranslated into CSS. This provides you with a convenient, time-saving way to check for CSS changes on page refresh without having to restart the application.

### Autoprefixer

[Vendor prefixes](https://developer.mozilla.org/en-US/docs/Glossary/Vendor\_Prefix) are a part of CSS that is added to certain properties and values. They enable experimental, non-standard features in different browsers. For example, the `-webkit-` prefix is used for properties and values supported by WebKit browsers (Google, Safari, etc.), and Mozilla Firefox uses the `-moz-` prefix.

Without using a tool like CSS Autoprefixer, you would have to take care of adding the correct prefixes yourself. This can be tedious and error-prone.

To ensure compatibility with different browsers, Smartstore has a built-in CSS Autoprefixer. It is enabled in production mode, but not in debug mode. This allows you to write CSS code without having to add vendor prefixes yourself, as the tool will add them automatically. It uses the latest available [Can I Use](https://caniuse.com/) data to add the prefix to each corresponding CSS property and value.

By using CSS Autoprefixer, developers can rest assured that all CSS styles will display correctly in all major browsers. They can focus on designing the site without worrying about compatibility.

### Cache & DiskCache

Generated assets are cached in RAM and on disk. This keeps the whole process highly performant and delays page rendering by only a few milliseconds when regenerating CSS files. The cache is automatically invalidated when an included file changes.

<mark style="color:blue;">Erläutern: DiskCache hat den Vorteil, dass generierte Assets einen App-Neustart überdauern. Ohne DiskCache (was man abschalten kann, bitte erläutern) muss bspw. bei jedem App-Neustart der Sass-Parser angeschmissen werden, was wiederum den Start etwas verzögert. Der DiskCache legt Dateien im Ordner</mark> <mark style="color:blue;"></mark>_<mark style="color:blue;">App\_Data/Tenants/Default/BundleCache</mark>_ <mark style="color:blue;"></mark><mark style="color:blue;">ab.</mark>

## Libraries

### jQuery

jQuery is an open-source JavaScript library that makes it easy for developers to work with the Document Object Model (DOM) and interact with HTML pages. It provides a set of methods for selecting, manipulating, and animating DOM elements. jQuery uses a short and concise syntax to simplify the use of JavaScript, and supports a variety of modern web browsers.

{% hint style="info" %}
For more information, see [jQuery](https://jquery.com/).
{% endhint %}

### Bootstrap

Bootstrap is a front-end framework based on HTML, CSS, and JavaScript. It is designed to provide developers with a quick and easy way to create responsive websites and web applications. It provides a set of CSS classes and JavaScript plugins that are preconfigured to ensure a consistent look and feel.

One of Bootstrap's most important features is its grid system. It's a flexible grid system that allows developers to control how content is displayed on different screen sizes. Bootstrap's grid system can be used to render a website so that it looks good on desktops, tablets, and mobile devices.

Bootstrap's grid system is based on 12 columns controlled by CSS classes. Each column can be scaled to a specific width to best display content on different devices. Bootstrap also provides different classes for displaying content on different device sizes, giving developers a flexible and customizable solution for displaying content on different devices.

In Smartstore, the mobile-first approach is fully implemented with Bootstrap's CSS classes. All CSS classes provided by Bootstrap can be used in Smartstore to create HTML structures.

{% hint style="info" %}
For more information, see [Bootstrap](https://getbootstrap.com/docs/4.6/layout/overview/).
{% endhint %}

## Icon Libraries

### Font Awesome

Font Awesome is an icon library that provides high quality, scalable font icons for use in websites and applications. The icons can be easily embedded using CSS classes and are available in different styles and sizes.

One of the advantages of Font Awesome is that the icons are scalable, so they look good and fit well on different screen resolutions. In addition, the icons can be customized with CSS to match the design of the website. Font Awesome offers a wide range of icons for different areas such as social media, user interfaces, signs and much more.

Font Awesome is fully integrated into Smartstore and can be used in all Razor views and CSS declarations.

{% hint style="info" %}
For more information, see [Font Awesome](https://fontawesome.com/icons).
{% endhint %}

#### Default font-weight

All icons come in one of four `font-weight` classes. Their default values are set by `$icon-font-weight-default` and `$icon-font-variants`. Changing these values has no effect in the free version.

| Class   | Prefix  | Sass variable                   | Default font-weight |
| ------- | ------- | ------------------------------- | ------------------- |
| Neutral | fa      | $icon-font-weight-default       | 900                 |
| Solid   | fa**s** | $icon-font-variants\["solid"]   | 900                 |
| Regular | fa**r** | $icon-font-variants\["regular"] | 400                 |
| Light   | fa**l** | $icon-font-variants\["light"]   | 300                 |

#### Font Awesome Free

The free version of Font Awesome contains a subset of icons. All solid and few regular icons are included, but none of the light icons.

To keep the store's look consistent, the `font-weight` of all icons is set to `900`, then all supported regular icons and their light counterparts are set to `400`. This will ensure that all supported icons are displayed, although the `font-weight` will change if light icons are used.

{% hint style="info" %}
The `font-weight` value for neutral icons is not affected.
{% endhint %}

#### Font Awesome Pro

The professional version of Font Awesome removes the icon limitations and includes all solid, regular and light icons.

Solid, regular and light icons all follow their font-weight values from `$icon-font-variants`. The `font-weight` of all neutral icons is set to `$icon-font-weight-default` unless it is not set to `900`.

For licensing reasons, we cannot ship Font Awesome Pro directly. In order to use Font Awesome Pro after you bought a license, you must complete the following steps.

* The `fa-use-pro` theme variable must be set to `true` in the `theme.config` file.
* The Font Awesome Pro includes must be added to an appropriate location, such as `_Layout.cshtml`:

```cshtml
<link sm-target-zone="head_links" rel="preconnect" href="https://pro.fontawesome.com" />
<link sm-target-zone="stylesheets" rel="stylesheet" href="https://pro.fontawesome.com/releases/v5.10.1/css/all.css" integrity="your key" crossorigin="anonymous" />
```

### Bootstrap Icons

Bootstrap Icons is a library of graphics and icons designed specifically for the Bootstrap framework. They can be used in HTML and CSS code to add graphical elements to websites and mobile applications.

Bootstrap icons are only available in Smartstore's backend, as the CSS for the frontend should remain as lightweight as possible.

{% hint style="info" %}
For more information, see [Bootstrap Icons](https://icons.getbootstrap.com/).
{% endhint %}

### Fontastic

Fontastic is another icon library that has been integrated into Smartstore. Since we also have to be careful not to overload the CSS in the frontend, we have made a special selection that is relevant for e-commerce.

{% hint style="info" %}
A full list of all available icons can be found in [fontastic.css](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Web/wwwroot/lib/fontastic/fontastic.css).
{% endhint %}
