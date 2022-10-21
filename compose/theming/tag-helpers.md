# 🥚 Tag Helpers

Tag Helpers are extensions of existing HTML tags. They can extend the functionality of a tag or even create completely new tags. Smartstore has created a lot of Tag Helpers to write simple and clear code and for better productivity.

## How To Use Tag Helpers

To use the Smartstore Tag Helpers in your Views, you'll need to reference specific libraries. You can do this either in your View file or by using `_ViewImports.cshtml`. Add the following lines and IntelliSense should recognise the Tag Helpers.

```
//for public Tag Helpers
@addTagHelper Smartstore.Web.TagHelpers.Public.*, Smartstore.Web.Common

//for shared Tag Helpers
@addTagHelper Smartstore.Web.TagHelpers.Shared.*, Smartstore.Web.Common

//for admin Tag Helpers
@addTagHelper Smartstore.Web.TagHelpers.Admin.*, Smartstore.Web.Common
```

{% hint style="info" %}
To learn more about \_ViewImports.cshtml, check out this [tutorial](https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro?view=aspnetcore-6.0#addtaghelper-makes-tag-helpers-available).
{% endhint %}

{% hint style="info" %}
Try it out!

Type: <mark style="color:green;">`<span s`</mark> You should see a suggestion for `sm-if`, which is part of Smartstore's `IfTagHelper`.
{% endhint %}

A quick example:

```cshtml
<span sm-if="@Model.visible">Here I am!</span>
```

## Smartstore Tag Helpers

Smartstore differentiates between three kinds of Tag Helpers.

* Public: Used in frontend
* Shared: Used in both front- and backend
* Admin: Used in backend

You can find the code to Smartstore Tag Helpers in _Smartstore.Web.Common/TagHelpers/_.

### Public

<details>

<summary>Captcha Tag Helper</summary>

The `CaptchaTagHelper` creates a [CAPTCHA](https://en.wikipedia.org/wiki/CAPTCHA).

```cshtml
<captcha sm-enabled="Model.DisplayCaptcha" />
```

It also supports the `sm-enabled` attribute.

</details>

<details>

<summary>Honeypot Tag Helper</summary>

The `HoneypotTagHelper` is Smartstore's cyber-security implementation of the [Honeypot](https://en.wikipedia.org/wiki/Honeypot\_\(computing\)) mechanism.

```cshtml
<honeypot />
```

</details>

### Shared

* ? AttributesTagHelper - allgemeine Attribute-Collection?
* BundleTagHelper - Bundles external scripts and styles
* CollapsedContentTagHelper
* ConfirmTagHelper
* FileIconTagHelper
* FileUploaderTagHelper
* LanguageTagHelper
* LocalizationScriptTagHelper
* PageAssetTagHelper
* TagNameTagHelper
* WidgetTagHelper

<details>

<summary>CollapsedContent Tag Helper</summary>

The CollapsedContentTagHelper hides text after a maximum height has been reached. It adds _Show more_ or _Show less_ to the Element.

```
<collapsed-content>
    Some text
</collapsed-content>
```

To specify the maximum number of pixel you want to show, add the `sm-max-height` attribute. Otherwise it uses the catalog's default setting.

</details>

<details>

<summary>If Tag Helper</summary>

The `IfTagHelper` adds a conditional attribute to the Element. The output is suppressed, if the condition evaluates to `false`.

```cshtml
<span sm-if="@Model.visible">Here I am!</span>
```

</details>

<details>

<summary>Zone Tag Helper</summary>

The `ZoneTagHelper` defines an area for widgets to inject content.

```cshtml
<zone name="a_widget_drop_zone_name">
```

It also supports these attributes:

* `model`
* `replace-content`
* `remove-if-empty`

</details>

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

### Admin

<details>

<summary>BackTo Tag Helper</summary>

The `BackToTagHelper` adds a left arrow icon to a link.

```cshtml
<a href="#" sm-backto>Go back</a>
```

</details>

<details>

<summary>DataGrid Tag Helper</summary>

There are different types of DataGrid Tag Helpers. They all work together to extend the functionality of the Grid Tag Helper.

Here is an excerpt from _Smartstore.Web/Areas/Admin/Views/ActivityLog\_Grid.ActivityLogs.cshtml_ to show this.

{% code title="_Grid.ActivityLogs.cshtml" %}
```cshtml
<datagrid allow-resize="true" allow-row-selection="true" allow-column-reordering="true">
    <datasource read="@Url.Action("ActivityLogList", "ActivityLog")"
                delete="@Url.Action("ActivityLogDelete", "ActivityLog")" />
    <sorting enabled="true">
        <sort by="CreatedOn" by-entity-member="CreatedOnUtc" descending="true" />
    </sorting>
    <paging position="Bottom" show-size-chooser="true" />
    <toolbar>
        <toolbar-group>
            <button datagrid-action="DataGridToolAction.ToggleSearchPanel" type="button" class="btn btn-light btn-icon">
                <i class="fa fa-fw fa-filter"></i>
            </button>
        </toolbar-group>
        <zone name="datagrid_toolbar_alpha"></zone>
        <toolbar-group class="omega"></toolbar-group>
        <zone name="datagrid_toolbar_omega"></zone>
        <toolbar-group>
            <button type="submit" name="delete-all" id="delete-all" value="clearall" class="btn btn-danger no-anims btn-flat">
                <i class="far fa-trash-alt"></i>
                <span>@T("Admin.Common.DeleteAll")</span>
            </button>
            <button datagrid-action="DataGridToolAction.DeleteSelectedRows" type="button" class="btn btn-danger no-anims btn-flat">
                <i class="far fa-trash-alt"></i>
                <span>@T("Admin.Common.Delete.Selected")</span>
            </button>
        </toolbar-group>
    </toolbar>
    <search-panel>
        <partial name="_Grid.ActivityLogs.Search" model="parentModel" />
    </search-panel>
    <columns>
        <column for="ActivityLogTypeName" entity-member="ActivityLogType.Name" hideable="false" />
        <column for="Comment" wrap="true" />
        <column for="CustomerEmail" entity-member="Customer.Email" type="string">
            <display-template>
                <a :href="item.row.CustomerEditUrl" class="text-truncate">
                    {{ item.value }}
                </a>
            </display-template>
        </column>
        <column for="IsSystemAccount" halign="center" sortable="false" />
        <column for="CreatedOn" entity-member="CreatedOnUtc" />
    </columns>
    <row-commands>
        <a datarow-action="DataRowAction.Delete">@T("Common.Delete")</a>
    </row-commands>
</datagrid>
```
{% endcode %}

</details>

<details>

<summary>SettingEditor Tag Helper</summary>

The `SettingEditorTagHelper` provides automatic HTML-Input type Mapping.

```cshtml
<setting-editor asp-for="Name"></setting-editor>
```

It automatically checks the type of `Name` and looks for an appropriate HTML input. Additionally it offers model binding and matching.

It also supports these attributes:

* `asp-template`
* `parent-selector`
* `sm-postfix`

</details>

<details>

<summary>SmartLabel Tag Helper</summary>

The `SmartLabelTagHelper` displays a label and an optional hint.

```cshtml
<smart-label asp-for="Name" />
```

It also supports these attributes:

* `sm-ignore-hint`
* `sm-text`
* `sm-hint`

</details>

## Further Reading

If you are interested in writing your own `TagHelper` or learning more about them, follow this [Microsoft Tag Helper tutorial](https://learn.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/intro?view=aspnetcore-6.0).
