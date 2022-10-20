---
description: >-
  TagHelpers are extensions of existing HTML tags. They can extend the
  functionality of a tag or even create completely new tags.
---

# 🥚 Tag Helpers

## How To Use TagHelpers

To use the Smartstore `TagHelpers` in your Views, you'll need to reference specific libraries.

You can do this either in your View file or by using `_ViewImports.cshtml`.

Add the following to lines and IntelliSense should recognise the TagHelpers.

```
@addTagHelper Smartstore.Web.TagHelpers.Shared.*, Smartstore.Web.Common
@addTagHelper Smartstore.Web.TagHelpers.Admin.*, Smartstore.Web.Common
```

{% hint style="info" %}
Try it out!

Type: <mark style="color:green;">`<span s`</mark> You should see a suggestion for `sm-if`, which is part of Smartstore's `IfTagHelper`.
{% endhint %}

A quick example:

```cshtml
<span sm-if="@Model.visible">Here I am!</span>
```

## Smartstore TagHelpers

You can find the code to Smartstore TagHelpers in `Smartstore.Web.Common` / `TagHelpers` /

### SettingEditorTagHelper

The `SettingEditorTagHelper` provides automatic HTML-Input type Mapping. It's tag is `setting-editor`.

```cshtml
<setting-editor asp-for="Name"></setting-editor>
```

It automatically checks the type of `Name` and looks for an appropriate HTML input.

### SmartLabelTagHelper

The `SmartLabelTagHelper` displays a label and an optional hint. It's tag is `smart-label`.

```cshtml
<smart-label asp-for="Name" />
```

### DataGrid

There are different type of DataGrid TagHelpers.

Most commonly used are:

```
Example 1

Example 2

Example 3
```

### CaptchaTagHelper

### HoneypotTagHelper

Honey is a cyber-security extension. It's tag name is `honeypot`.

```
Sample code
```

### Shared

* AttributesTagHelper
* BundleTagHelper
* CollapsedContentTagHelper
* ConfirmTagHelper
* FileIconTagHelper
* FileUploaderTagHelper
* IfTagHelper
* LanguageTagHelper
* LocalizationScriptTagHelper
* PageAssetTagHelper
* TagNameTagHelper
* WidgetTagHelper

### ZoneTagHelper

The ZoneTagHelper defines an area for widgets to inject content. It's tag is `zone`.

```
<zone name="a_widget_drop_zone_name">
```

It also supports these attributes:

* `model`
* `replace-content`
* `remove-if-empty`

### IfTagHelper

The `IfTagHelper` adds a conditional attribute to the Element. It's key is `sm-if`.

```cshtml
<span sm-if="@Model.visible">Here I am!</span>
```

#### Controls

* EntityPickerTagHelper
* MenuTagTagHelper
* ModalBodyTagHelper
* ModalFooterTagHelper
* ModalHeaderTagHelper
* ModalTagHelper
* PaginationTagHelper
* TabStripTagHelper
* TabTagHelper

#### Forms

* AjaxFormTagHelper
* CollectionItemTagHelper
* ColorBoxTagHelper
* EditorTagHelper
* FormControlTagHelper
* HintTagHelper
* NumberInputTagHelper
* TripleDatePickerTagHelper

#### Media

* AudioTagHelper
* BaseImageTagHelper
* BaseMediaTagHelper
* ImageTagHelper
* MediaTagHelper
* ThumbnailTagHelper
* VideoTagHelper

## Further Reading

If you are interested in writing your own `TagHelper`, use the following links.

* Microsoft Tutorial 1
* Other Tutorial 2
