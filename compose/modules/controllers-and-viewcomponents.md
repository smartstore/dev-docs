# 🥚 Controllers & ViewComponents

Smartstore modules are pure MVC projects. This means that normal controller classes can be implemented using the MVC pattern. Since varying fields of application often require repetitive tasks, we have created base controllers following the [DRY principle](https://en.wikipedia.org/wiki/Don't\_repeat\_yourself). These take care of many tasks and can of course be used to implement your own controllers.

The base controllers are located in the _Controllers_ directory of the Smartstore.Web.Common project. By including the most important attributes and implementing auxiliary methods, they keep controllers consistent and eliminate redundancy.

## SmartController

The `SmartController` is the base controller that all other controllers implement. The provided functionality and members are therefore present in all controllers. Since the `SmartController` is the lowest possible abstraction, it should not be used directly. It is better to use controllers that implement the `SmartController` as a base for your own controller, such as the `PublicController` or the `AdminController`.

### Members

<details>

<summary>Logger</summary>

Can be used to log errors or information.

```csharp
Logger.Error(ex);
```

For more information, see [Logging](../../framework/platform/logging.md).

</details>

<details>

<summary>T</summary>

Provides the `Localizer` using the `T` shortcut that can be used to access localized resources.

```csharp
T("Admin.Common.OK")
```

For more information, see [Localization](../../framework/content/localization.md).

</details>

<details>

<summary>Services</summary>

Provides the `ICommonServices` service collection using the `Services` shortcut.

```csharp
var customer = Services.WorkContext.CurrentCustomer;
```

For more information, see `ICommonServices`.

</details>

### Methods

<details>

<summary>Widget, View &#x26; Component rendering</summary>

The following methods can be used to invoke a view and return its HTML result content as a string. This can be useful when content needs to be returned by AJAX requests.

* `InvokeViewAsync`
* `InvokePartialViewAsync`
* `InvokeComponentAsync`

```csharp
var html = await InvokePartialViewAsync("MyPartialView", myPartialViewModel);
```

</details>

<details>

<summary>Notify</summary>

Send a message to the user using the `NotifyInfo`, `NotifyWarning`, `NotifySuccess`, `NotifyError` and `NotifyAccessDenied` shortcuts.

```csharp
NotifyError("Something went wrong during this operation.");
```

</details>

<details>

<summary>Redirection</summary>

The `RedirectToReferrer` method redirects users to the page they came from.

```csharp
public async Task<IActionResult> MyAction()
{
    // ...
    return RedirectToReferrer();
}
```

</details>

## ManageController

The `ManageController` provides auxiliary methods, that can be used to configure domain entities and settings. Since this is mostly used in the admin area, we recommend basing your controller on the [AdminController](controllers-and-viewcomponents.md#admincontroller) instead of the `ManageController` directly. It implements the `ManageController` and adds more auxiliary methods relevant to the admin area.

{% hint style="info" %}
All previous examples can be applied to controllers based on the `ManageController`, because it implements the `SmartController`.
{% endhint %}

### Methods

<details>

<summary>AddLocales / AddLocalesAsync</summary>

Please make sure you understand the [concept of localizable entities](../../framework/content/localization.md) before reading on.

To prepare models for localizable entities, a localized model must be added for each enabled language in the store. This model must contain all values of the localizable properties and be able to receive them for postback. Localized models must implement the `ILocalizedModel<MyLocalizedModel>` interface, where MyLocalizedModel contains all the properties of the localized values to be stored. This ensures that the `List<MyLocalizedModel> Locales` property is always available.

The following call initializes the model, as long as it doesn’t contain any entities. This is the case for `Create` methods.

```csharp
AddLocales(model.Locales);
```

If localized entities already exist, for example, when using `Edit` methods or methods that prepare the model for public rendering, the localized model can be initialized like this:

```csharp
var myEntity = await _db.MyEntities().FindByIdAsync(id, false);

AddLocales(model.Locales, (locale, languageId) =>
{
    locale.MyLocalizedProperty = myEntity.GetLocalized(x => x.MyLocalizedProperty, languageId, false, false);
});
```

</details>

<details>

<summary>SaveStoreMappingsAsync</summary>

Please make sure you understand the [concept of Multistores](../../framework/content/multistore.md) before reading on.

Entities can implement the `IStoreRestricted` interface, which allows the user to restrict displaying an entity to specific stores. `IStoreRestricted` requires the implementing entity to have the following property:

```csharp
public bool LimitedToStores { get; set; }
```

The model must provide the ability to hold the selected store IDs. The store restriction can be further configured by the store owner and is typically implemented in the model as follows:

```csharp
[UIHint("Stores")]
[AdditionalMetadata("multiple", true)]
[LocalizedDisplay("Admin.Common.Store.LimitedTo")]
public int[] SelectedStoreIds { get; set; }
```

To save the store mapping defined by the store owner you can then use the following call:

```csharp
var myEntity = await _db.MyEntities().FindByIdAsync(id, false);
await SaveStoreMappingsAsync(myEntity, model.SelectedStoreIds);
```

</details>

<details>

<summary>SaveAclMappingsAsync</summary>

Please make sure you understand the [concept of ACL](https://en.wikipedia.org/wiki/Access-control\_list) before reading on.

Entities can implement the `IAclRestricted` interface, which allows the user to restrict displaying an entity to specific `CustomerRoles`. `IAclRestricted` requires the implementing entity to have the following property:

```csharp
public bool SubjectToAcl { get; set; }
```

The model must provide the ability to hold the selected customer role IDs. ACL restriction can be further configured by the store owner and is typically implemented in the model as follows:

```csharp
[UIHint("CustomerRoles")]
[AdditionalMetadata("multiple", true)]
[LocalizedDisplay("Admin.Common.CustomerRole.LimitedTo")]
public int[] SelectedCustomerRoleIds { get; set; }
```

To save the ACL restrictions defined by the store owner you can then use the following call:

```csharp
var myEntity = await _db.MyEntities().FindByIdAsync(id, false);
await SaveAclMappingsAsync(myEntity, model.SelectedCustomerRoleIds);
```

</details>

<details>

<summary>GetActiveStoreScopeConfiguration</summary>

When configuring settings, the store owner needs to be able to define different settings for each store. This can be done by adding the `StoreScope ViewComponent` to the view. Then the `GetActiveStoreScopeConfiguration` method can be used in controller methods to access the store id set by the store owner in the StoreScope setting.

</details>

## AdminController

{% hint style="info" %}
All previous examples can be applied to controllers based on the `AdminController`, because it implements the `ManageController`.
{% endhint %}

The `AdminController` adds the following attributes:

| Attribute                    | Description                                                                                                                                                                              |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Area("Admin")                | Specifies `Admin` as area of the controller.                                                                                                                                             |
| AutoValidateAntiforgeryToken | An attribute that causes antiforgery token validation for all insecure HTTP methods. An antiforgery token is required for HTTP methods other than `GET`, `HEAD`, `OPTIONS`, and `TRACE`. |
| AuthorizeAdmin               | Checks whether the current user has permission to access the administration backend.                                                                                                     |
| ValidateAdminIpAddress       | Verifies that the IP address being used matches one of the addresses defined in `AdminAreaAllowedIpAddresses` from the `SecuritySettings`.                                               |

## PublicController

The `PublicController` implements the `SmartController` and adds the following attributes:

| Attribute                                                                             | Description                                                                                                   |
| ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| CookieConsent                                                                         | Checks if the visitor has already allowed the use of cookies and opens the `CookieManager` if not.            |
| PreviewMode                                                                           | Enables the use of preview mode in the public store as long as preview mode is enabled in the backend.        |
| CheckStoreClosed                                                                      | Checks if the store has been closed by the admin and returns a redirect result to the `StoreClosed` action.   |
| AuthorizeShopAccess                                                                   | Checks whether the current user has permission to access the shop.                                            |
| TrackActivity(Order = 100)                                                            | Saves current user activity information including the date, IP address and the visited page, in the database. |
| CheckAffiliate(Order = 100)                                                           | Checks if a visiting customer was referred to the shop by an affiliate by analyzing the request query.        |
| <p>SaveChanges(</p><p>    typeof(SmartDbContext),</p><p>    Order = int.MaxValue)</p> | Saves all pending changes in a `DbContext` instance to the database after an action method has been executed. |

## ViewComponents

`ViewComponents` is a feature that was added to ASP.NET Core. They provide developers with the means to create recurring components that are used in the application GUI. Similar to controllers, the `SmartViewComponent` base class adds ways to perform repetitive tasks and some auxiliary methods.

### Members

<details>

<summary>Logger</summary>

Can be used to log errors or information.

```csharp
Logger.Error(ex);
```

For more information, see [Logging](../../framework/platform/logging.md).

</details>

<details>

<summary>T</summary>

Provides the `Localizer` using the `T` shortcut that can be used to access localized resources.

```csharp
T("Admin.Common.OK")
```

For more information, see [Localization](../../framework/content/localization.md).

</details>

<details>

<summary>Services</summary>

Provides the `ICommonServices` service collection using the `Services` shortcut.

```csharp
var customer = Services.WorkContext.CurrentCustomer;
```

For more information, see `ICommonServices`.

</details>

### Methods

<details>

<summary>Results</summary>

There are three different ways to return rendered content using the `SmartViewComponent`:

* `Content`: Returns a result that renders HTML encoded text.
* `HtmlContent`: Returns a result that renders raw (unencoded) HTML content.
* `View`: Returns a result that renders the partial view with an explicit name, or `default`, if none is specified.

```csharp
public IActionResult MyMethod()
{
    // ...
    return View();
}
```

</details>

<details>

<summary>Notify</summary>

Send a message to the user using the `NotifyInfo`, `NotifyWarning`, `NotifySuccess`, `NotifyError` and `NotifyAccessDenied` shortcuts.

```csharp
NotifyError("Something went wrong during this operation.");
```

</details>

### Events

<details>

<summary>PublishResultExecutingEvent</summary>

In a `ViewComponent` that implements `SmartViewComponent`, the content is returned as a `IViewComponentResult` using one of the rendering methods (see Results). They call the internal method `PublishResultExecutingEvent`, which publishes a `ViewComponentResultExecutingEvent` that developers can subscribe to. This allows them to react to the rendering of the component. The `ResultExecutingEvent` is always published just before a view component is about to render the view. This event basically replaces the `OnActionExecuted` and `OnResultExecuting` child action filters of the classic MVC.

```csharp
public async Task HandleEventAsync(ViewComponentResultExecutingEvent message)
{
    var componentType = message.Descriptor.TypeInfo.AsType();

    // Only do stuff when InterestingViewComponent is rendered
    if (componentType == typeof(InterestingViewComponent))
    {
        // Do stuff
    }
}
```

</details>

<details>

<summary>ViewComponentInvokingEvent</summary>

The `ViewComponentInvokingEvent` is published just before a view component is about to be created or its model is about to be prepared. This event basically replaces the `OnActionExecuting` child action filter of classic MVC.

Unlike the `ViewComponentResultExecutingEvent`, which is **ALWAYS** published implicitly by the `SmartViewComponent`, this event must be published explicitly by the view component authors. This is best done just before model creation.

The component author should check if the model property has been assigned a non-null value by any event consumer. In this case, model creation should be skipped and an externally provided model should be used instead.

```csharp
var myComponentEvent = new ViewComponentInvokingEvent<MyModel>(ViewComponentContext);
await _eventPublisher.PublishAsync(myComponentEvent);
```

</details>
