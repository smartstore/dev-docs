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

## Anatomy of a TagHelper

To get an idea of how a `TagHelper` is written, let's take a look at the `IfTagHelper`.

{% code title="IfTagHelper.cs" lineNumbers="true" %}
```csharp
using Microsoft.AspNetCore.Razor.TagHelpers;

namespace Smartstore.Web.TagHelpers.Shared
{
    [HtmlTargetElement("*", Attributes = IfAttributeName)]
    public class IfTagHelper : TagHelper
    {
        const string IfAttributeName = "sm-if";

        public override int Order => int.MinValue;

        /// <summary>
        /// A condition to check before outputting the tag.
        /// <c>false</c> will suppress the output completely.
        /// </summary>
        [HtmlAttributeName(IfAttributeName)]
        public bool Condition { get; set; } = true;

        public override Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
        {
            if (!Condition)
            {
                output.SuppressOutput();
            }

            return Task.CompletedTask;
        }
    }
}
```
{% endcode %}

`Line 5`: HtmlTargetElement: `*`, `IfAttributeName`

The first argument describes on what HTML element the TagHelper can be used on.

The second argument uses `IfAttributeName` (`Line 8`) to define the attribute key.

`Line 6`: extends the abstract TagHelper class

{% hint style="info" %}
The abstract `TagHelper` class provides:

* int Order
* void Init
* void Process
* Task ProcessAsync
{% endhint %}

`Line 8`: `IfAttributeName` - specifies the attribute name used in the HTML tag

`Line 16`: `[HtmlAttributeName(IfAttributeName)]` sets the attribute name

`Line 10`: `Order` - specifies the execution order in a set of `TagHelpers`

`Line 17`: `Condition` - property of the `TagHelper`

`Line 19`: `ProcessAsync` - is called when the tag is being processed

* `TagHelperContext` - contains the name and attribute list of the tag
* `TagHelperOutput` - object to interact with the DOM

`Line 21-26`: The program logic

* If `Condition` is `true`, the task is completed.
* If `Condition` is `false`, the ouput is suppressed.

## Generic TagHelpers and where to find them

You can find the code to Smartstore TagHelpers in `Smartstore.Web.Common` / `TagHelpers` /

### Basic TagHelpers

* BaseFormTagHelper
* SmartTagHelper

### Admin

* BackToTagHelper
* BootstrapIconTagHelper
*   SettingEditorTagHelper

    Provides automatic HTML-Input type Mapping

    usage of UIHint ->\[Smartstore.DevTools -> ConfigurationModel.cs -> UIHint(Smart-Editor-TagHelper-Type)]
* SmartLabelTagHelper

#### DataGrid

* GridColumnsTagHelper
* GridColumnTagHelper
* GridDataSourceTagHelper
* GridDetailViewTagHelper
* GridFilteringTagHelper
* GridPagingTagHelper
* GridRowCommandTagHelper
* GridSearchPanelTagHelper
* GridSortingTagHelper
* GridTagHelper
* GridToolbarTagHelper

### Public

* CaptchaTagHelper
*   HoneypotTagHelper

    Honey is a cyber-security extension.

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
* ZoneTagHelper

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

## How To Write Your Own TagHelper

Now that we know what TagHelpers are, how they are structured and what they are capable of doing, let's write one together.
