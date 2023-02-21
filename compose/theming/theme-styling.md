# 🐣 Theme styling

## Overview

To create a completely different look & feel of the store frontend, it is sufficient to create a manifest file (`theme.config`) in a separate directory in the theme directory and define the variables. For deeper changes, add your own CSS to modify existing declarations, overwrite entire sections with your own CSS, or change the given HTML structure by overwriting it with your own Razor views.

## theme.scss

After `theme.config`, the most important file in a theme is `theme.scss`. It is the root Sass file that includes all other Sass files. Our Sass files are organized in a very granular way, representing each section of the store separately. For example, there is one file that contains all the CSS for the checkout process, one for the footer, one for the product detail page, and so on. A complete list of all included Sass files can be found below.

Bootstrap's Sass files are included very early in `theme.scss`, so you get access to Bootstrap's variables and mixins in subsequent includes.

For example, it's possible to use the mixins provided by Bootstrap for responsive rendering everywhere:

```scss
@include media-breakpoint-up(lg) {
    --content-padding-x: #{$content-padding-x};
} 
```

## AutoPrefixer

Smartstore has an integrated CSS autoprefixer to ensure compatibility with different browsers. It is enabled in production mode, but not in debug mode. This allows writing CSS code without vendor prefixes, because the Autoprefixer adds them automatically. It uses the latest available [Can I Use](https://caniuse.com/) data to add the prefix to each corresponding CSS property and value.

## All `.scss` files in Flex

| File                    | Description                                                                                                             |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| /.app/themevars.scss    | Virtual file that fetches all theme variables from the database. We use these variables to override Bootstrap defaults. |
| \_variables-custom.scss | Empty file to override defaults without modifying source files.                                                         |
| \_variables.scss        | Includes theme-specific variable defaults and Bootstrap variable overrides.                                             |
| \_fonts.scss            | Empty file with predefined CSS declarations that can be overridden for language-specific font families.                 |

### Bootstrap 4 + variables reset

| File                              | Description                                                                                                       |
| --------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| /lib/bs4/scss/bootstrap-head.scss | Custom Smartstore file that contains only the import of Bootstrap functions and variables.                        |
| \_variables-reset.scss            | Used to omit, reset, or redefine imported bootstrap variables with custom values before importing Bootstrap main. |
| /lib/bs4/scss/bootstrap-main.scss | Main file to include Bootstrap Sass files.                                                                        |

### Vendor components (neutral)

| File                           | Description                                                                                                                                                           |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| lib/rfs/\_rfs.scss             | Used to set automated, responsive values for font sizes, padding, margins, and more.                                                                                  |
| lib/photoswipe/photoswipe.scss | Styles for basic PhotoSwipe functionality (sliding area, open / close transitions)                                                                                    |
| lib/slick/slick.scss           | Styles for Slick Slider                                                                                                                                               |
| lib/select2/scss/core.scss     | Styles for Select2, which gives you a customizable select box with support for searching, tagging, remote records, infinite scrolling, and many more popular options. |
| lib/aos/scss/aos.scss          | Styles for AOS (Animate On Scroll) library.                                                                                                                           |

### Button tweaks

| File           | Description                                         |
| -------------- | --------------------------------------------------- |
| \_buttons.scss | Used to override some Bootstrap values for buttons. |

### Global / Shared

| File                           | Description                                                                                                          |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------- |
| shared/\_colors.scss           | The Material Design color palette (commented out for performance reasons).                                           |
| shared/\_variables-shared.scss | Used to define & override variables previously included by third-party libraries.                                    |
| shared/\_mixins.scss           | Contains our own mixins.                                                                                             |
| shared/\_spacing.scss          | Mixins for responsive spacing (replaces the default Bootstrap spacing include).                                      |
| shared/\_typo.scss             | Defines typography styles.                                                                                           |
| shared/\_fa.scss               | Contains custom styles for Font Awesome.                                                                             |
| shared/\_alert.scss            | Contains enhancements for Bootstrap's alert classes.                                                                 |
| shared/\_buttons.scss          | Contains enhancements for Bootstrap's button classes & introduces own button classes.                                |
| shared/\_dropdown.scss         | Contains enhancements for Bootstrap's dropdown classes.                                                              |
| shared/\_card.scss             | Contains enhancements for Bootstrap's card classes.                                                                  |
| shared/\_forms.scss            | Contains enhancements for Bootstrap's form classes.                                                                  |
| shared/\_numberinput.scss      | Styles for our own number input control.                                                                             |
| shared/\_breadcrumb.scss       | Contains enhancements for Bootstrap's breadcrumb classes.                                                            |
| shared/\_pagination.scss       | Contains enhancements for Bootstrap's pagination classes.                                                            |
| shared/\_nav.scss              | Contains enhancements for Bootstrap's Navbar classes & introduces own components based on Bootstraps Navbar classes. |
| shared/\_nav-collapsible.scss  | Contains styles for a collapsible navbar.                                                                            |
| shared/\_modal.scss            | Contains enhancements for Bootstrap's modal classes.                                                                 |
| shared/\_throbber.scss         | Styles used by our throbber plugin.                                                                                  |
| shared/\_spinner.scss          | Styles for a spinner that uses the entire available space of the browser window.                                     |
| shared/\_star-rating.scss      | Styles to display beautiful stars for ratings.                                                                       |
| shared/\_sortable-grip.scss    | Styles to display a gripper.                                                                                         |
| shared/\_choice.scss           | Styles to display our choice templates (used by Variants etc.)                                                       |
| shared/\_offcanvas.scss        | Contains styles for our offcanvas components (e.g. Offcanvas-Cart)                                                   |
| shared/\_sections.scss         | Contains styles to define theme colored sections.                                                                    |
| shared/\_bg.scss               | Contains declarations to be used as background classes.                                                              |
| shared/\_custom-scrollbar.scss | Defines a slimmer custom scrollbar.                                                                                  |
| shared/\_box.scss              | Contains classes & effects used to display content in boxes (e.g. box image, box scale, box rise).                   |
| shared/\_utils.scss            | Contains utility classes.                                                                                            |
| shared/\_switch.scss           | Styles for our own (boolean) switch control.                                                                         |
| shared/\_media.scss            | Styles for media file display (in the context of Media Manager)                                                      |
| shared/\_text-expander.scss    | Styles for our own text expander control.                                                                            |
| shared/\_entity-picker.scss    | Styles for our own entity picker control.                                                                            |

### Vendor component (skinning)

| File                       | Description                |
| -------------------------- | -------------------------- |
| skinning/\_select2.scss    | Skins Select2 component    |
| skinning/\_pnotify.scss    | Skins PNotify component    |
| skinning/\_photoswipe.scss | Skins PhotoSwipe component |
| skinning/\_slick.scss      | Skins Slick component      |
| skinning/\_drift.scss      | Skins Drift component      |
| skinning/\_fileupload.scss | Skins Fileupload component |
| skinning/\_summernote.scss | Skins Summernote component |

### Main

| File                  | Description                                                                                           |
| --------------------- | ----------------------------------------------------------------------------------------------------- |
| \_layout.scss         | Contains general layout styles.                                                                       |
| \_block.scss          | Contains styles to be used when a topic is rendered as a widget and should be wrapped by a container. |
| \_shopbar.scss        | Contains styles used by the shopbar.                                                                  |
| \_footer.scss         | Contains styles used in the footer.                                                                   |
| \_menu.scss           | Contains styles used by menus.                                                                        |
| \_megamenu.scss       | Contains styles used by the mega menu.                                                                |
| \_search.scss         | Contains styles used in instant search and search result page.                                        |
| \_login.scss          | Contains styles used on the login page.                                                               |
| \_rating.scss         | Contains styles used for reviews and ratings.                                                         |
| \_artlist.scss        | Contains styles used to display article lists.                                                        |
| \_product.scss        | Contains styles used on product pages.                                                                |
| \_gallery.scss        | Contains styles used by the Media Gallery on product pages.                                           |
| \_cart.scss           | Contains styles used on the shopping cart page.                                                       |
| \_checkout.scss       | Contains styles used in the checkout process.                                                         |
| \_accordion.scss      | Contains styles used to display an accordion.                                                         |
| \_cookie-manager.scss | Contains styles used by the Cookie Manager window.                                                    |
| \_misc.scss           | Contains various styles that do not fit anywhere else.                                                |
| \_print.scss          | Contains styles used when printing documents.                                                         |

### Custom imports from modules

| File                     | Description                                                                                                                                                                                                                 |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| /.app/moduleimports.scss | Virtual file that fetches all styles from modules defined by public.scss in the wwwroot directories in Modules.                                                                                                             |
| \_custom.scss            | The \_custom.scss file can be used, for example, to override CSS declarations already made in the base theme. This file is the last to be included in our main theme (Flex), so its declarations have the highest priority. |
| \_user.scss              | The \_user.scss file is for the end user, and should be present in every theme, but empty, so that the end user can place their own CSS here, which will remain untouched by updates to the actual theme.                   |
