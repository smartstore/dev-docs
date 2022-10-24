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

* BundleTagHelper - Bundles external scripts and styles

<details>

<summary>CollapsedContent Tag Helper</summary>

The `CollapsedContentTagHelper` collapses the element to a maximum height. It also adds _Show more_ or _Show less_ to the Element.

```cshtml
<collapsed-content>
    Odit non aspernatur sunt ipsum dolorem nihil quibusdam earum.<br />
    Eius nulla magni cum cum delectus sit omnis. Quam aut itaque ut.<br />
    Adipisci nihil enim aut eos voluptas et. Iure ut maxime ut qui.<br />
    Impedit adipisci laborum quia pariatur. Laboriosam voluptatibus<br />
    atque qui minima et ut deleniti.<br />
    <br />
    Debitis beatae aut aut iusto non consequuntur. Et inventore placeat<br />
    alias ut consequatur corrupti. Ut qui laboriosam amet tempora velit<br />
    sed est. Dolorem doloremque reiciendis voluptatem quasi nemo<br />
    perferendis quo. Voluptas exercitationem consequatur dolorum omnis<br />
    porro necessitatibus dignissimos qui.<br />
    <br />
    Consectetur et corporis vel voluptas autem libero magnam. Mollitia<br />
    pariatur placeat ut. Dolores quidem molestiae dolore ut accusamus<br />
    quam dolorem iure. Nihil optio voluptatibus eum quis.<br />
    <br />
    Officia a accusantium nihil voluptas et. Error aut labore est qui<br />
    rem. Fugiat perspiciatis repellendus voluptatem aut qui dolorem.
</collapsed-content>
```

To specify the maximum number of pixel you want to show, add the `sm-max-height` attribute. Otherwise the catalog's default setting will be used.

</details>

<details>

<summary>Confirm Tag Helper</summary>

The `ConfirmTagHelper` adds a confirm button. There are many ways to customise it.

```cshtml
<confirm button-id="entry-delete" />

<confirm message="@T("Common.AreYouSure")" button-id="entry-delete" icon="fas fa-question-circle" action="EntryDelete" type="Action" />
```

</details>

<details>

<summary>FileIcon Tag Helper</summary>

The `FileIconTagHelper` display a file icon.

```cshtml
<file-icon file-extension="@Model.FileExtension" show-label="true" />
```

It also supports these attributes:

* `label`
* `show-label`
* `badge-class`

</details>

<details>

<summary>FileUploader Tag Helper</summary>

The `FileUploaderTagHelper` adds a highly customisable way to upload files.

Here is an excerpt from _Smartstore.Web/Views/Customer/Avatar.cshtml_ to show this.

{% code title="Avatar.cshtml" %}
```cshtml
<file-uploader 
    file-uploader-name="uploadedFile"
    upload-url='@Url.Action("UploadAvatar", "Customer")'
    type-filter="image"
    display-browse-media-button="false"
    display-remove-button="fileId != 0"
    display-remove-button-after-upload="true"
    upload-text='@T("Common.FileUploader.UploadAvatar")'
    onuploadcompleted="onAvatarUploaded"
    onfileremoved="onAvatarRemoved"
    multi-file="false"
    has-template-preview="true" />
```
{% endcode %}

</details>

<details>

<summary>If Tag Helper</summary>

The `IfTagHelper` adds a conditional attribute to the Element. The output is suppressed, if the condition evaluates to `false`.

```cshtml
<span sm-if="@Model.visible">Here I am!</span>
```

</details>

<details>

<summary>TagName Tag Helper</summary>

The `TagNameTagHelper` changes the tag at runtime.

```cshtml
<span sm-tagname="div">I always wanted to be a div...</span>
```

</details>

<details>

<summary>Widget Tag Helper</summary>

The `WidgetTagHelper` adds HTML content and injects it into a [zone](../../framework/content/widgets.md#zones). More information can be found in [Widgets](../../framework/content/widgets.md#widget-tag-helper).

```cshtml
<widget target-zone="my_widget_zone">
    <span>Widget content</span>
</widget>
```

</details>

<details>

<summary>Zone Tag Helper</summary>

The `ZoneTagHelper` defines an zone for widgets to inject content. More information can be found in [Widgets](../../framework/content/widgets.md#zones).

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

<details>

<summary>AjaxForm Tag Helper</summary>

The `AjaxFormTagHelper` adds unobtrusive AJAX to a form.

```cshtml
<form sm-ajax method="post" asp-area="" asp-action="Do" sm-onsuccess="OnDo@(Model.Id)" sm-loading-element-id="#do-prog-@(Model.Id)">
```

It also supports these attributes:

* `sm-ajax`
* `sm-confirm`
* `sm-onbegin`
* `sm-oncomplete`
* `sm-onfailure`
* `sm-onsuccess`
* `sm-allow-cache`
* `sm-loading-element-id`
* `sm-loading-element-duration`
* `sm-update-target-id`
* `sm-insertion-mode`

Further information can be found in this [explanation](https://www.learnrazorpages.com/razor-pages/ajax/unobtrusive-ajax).

</details>

<details>

<summary>CollectionItem Tag Helper</summary>

The `CollectionItemTagHelper` prepends a hidden prefix element to all it's current children.

Unsure??

```cshtml
<collection-item name="@collectionName">
    //A collection of nodes
</collection-item>
```

</details>

<details>

<summary>ColorBox Tag Helper</summary>

The `ColorBoxTagHelper` adds a color-picker. It is used under the hood of the `SettingEditorTagHelper` to display colors.

```cshtml
<colorbox asp-for="MyColour" sm-default-color="#ff2030")" />
```

</details>

<details>

<summary>Editor Tag Helper</summary>

The `EditorTagHelper` adds a customisable input field. It works similiar to the `SettingEditorTagHelper`.

```cshtml
<editor asp-for="PriceInclTax" sm-postfix="@primaryStoreCurrencyCode" />
```

</details>

<details>

<summary>FormControl Tag Helper</summary>

The FormControlTagHelper adds labels and CSS classes to form elements.

```cshtml
<input type="text" asp-for="Name" />
<input type="checkbox" asp-for="MyBool" sm-switch />
```

It also supports these attributes:

* `asp-items`
* `sm-append-hint`
* `sm-ignore-label`
* `sm-control-size`
* `sm-plaintext`
* `sm-required`

</details>

<details>

<summary>Hint Tag Helper</summary>

The `HintTagHelper` converts the element into a hint.

```cshtml
<div><span>John Smith</span><hint asp-for="Name" /></div>
```

</details>

<details>

<summary>NumberInput Tag Helper</summary>

The `NumberInputTagHelper` extends the number-input's customisability and styles the element.

```cshtml
<input type="number" sm-decimals="2" sm-numberinput-style="centered" asp-for="price"/>
```

</details>

<details>

<summary>TripleDatePicker Tag Helper</summary>

The TripleDatePickerTagHelper adds a customisable date-picker displaying the day, month and year.

{% code overflow="wrap" %}
```cshtml
<triple-date-picker day-name="@(controlId + "-day")" month-name="@(controlId + "-month")" year-name="@(controlId + "-year")" day="Model.SelectedDay" month="Model.SelectedMonth" year="Model.SelectedYear" begin-year="Model.BeginYear" end-year="Model.EndYear" disabled="Model.IsDisabled" />
```
{% endcode %}

</details>

#### Media

<details>

<summary>Audio Tag Helper</summary>

The `AudioTagHelper` adds an audio element.

```cshtml
<audio sm-file="AudioFile" />
```

</details>

<details>

<summary>Image Tag Helper</summary>

The `ImageTagHelper` adds an image with attributes used in `Model` or the File.

```cshtml
<img sm-file="JPGFile"/>
```

</details>

<details>

<summary>Media Tag Helper</summary>

The `MediaTagHelper` adds a suitable tag for a given media type.

```cshtml
<media sm-file="Model.CurrentFile" sm-size="Model.ThumbSize" alt="@picAlt" title="@picTitle" />
```

</details>

<details>

<summary>Thumbnail Tag Helper</summary>

The `ThumbnailTagHelper` adds thumbnail of a media file.

```cshtml
<media-thumbnail sm-file="MediaFile" sm-size="ThumbSize" />
```

</details>

<details>

<summary>Video Tag Helper</summary>

The `VideoTagHelper` adds a video element.

```cshtml
<video sm-file="VideoFile" controls preload="metadata" />
```

</details>

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
