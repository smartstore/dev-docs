# Licensable modules

## Overview

A module can be marked as licensable piece of code by decorating the module class (inherited from `ModuleBase`) with the `LicensableModuleAttribute`.

HINT: There must be a dependency on the _Smartstore.Web.Common_ project for the module to use the licensing component.

To be able to fully use the module, the shop administrator must enter a license key in the administration backend, which he has previously purchased on the [Smartstore Marketplace](http://community.smartstore.com/index.php?/files/). The key is activated after it is entered.

HINT: A license key is only valid for the IP address that activated the key.

`LicensableModuleAttribute` has an option `HasSingleLicenseForAllStores` related to multi-stores. If set to _True_, then the license key has no domain binding and is therefore valid for all stores. Recommended is the default value _False_, i.e. a new license key is required for each store.

The module should perform a license check at the beginning of important functions and disable or restrict them if there is no valid license.

## License check

The module should check the license when executing important functions. It is recommended to do this not permanently but throttled via `Throttle.Check`.

WARN: Always use the system name of the module to check a license, not those of a provider!

```csharp
[LicensableModule]
internal class Module : ModuleBase, IConfigurable
{
    public static string SystemName => "MyCompany.MyModule";

    public async static Task<bool> CheckLicense(bool async)
    {
        // TODO: set key. Must be globally unique!
        var key = "any random characters";
        // Check once per hour.
        var interval = TimeSpan.FromHours(1);

        if (async)
        {
            return await Throttle.CheckAsync(key, interval, true,
                async () => await LicenseChecker.CheckStateAsync(SystemName) > LicensingState.Unlicensed);
        }
        else
        {
            return Throttle.Check(key, interval, true,
                () => LicenseChecker.CheckState(SystemName) > LicensingState.Unlicensed);
        }
    }
}
```

`LicensingState` is an `enum` with following values:

| State          | Value | Decsription                                                                                                                                                            |
| -------------- | :---: | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Unlicensed** |   0   | Invalid license key or demo period expired.                                                                                                                            |
| **Demo**       |   10  | Demonstration period valid for 30 days. After expiration, the status changes to _Unlicensed_. An unlimited demonstration period is given to developers on `localhost`. |
| **Licensed**   |   20  | The license is valid.                                                                                                                                                  |

HINT: The license check is always performed against the current request URL or as a fallback against the current store URL (if there is no HTTP request object, which is usually the case in background tasks). If you want to check against another domain, you have to pass the corresponding URL via the second parameter of `CheckStateAsync`.

### License Checker

The license checker is a set of static methods for license validation within `Smartstore.Licensing.dll`.

| Method                                                                 | Description                                                                                                                                                                                                                                                       |
| ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **IsLicensableModule**                                                 | Gets a value indicating whether a module is a licensed piece of code where the user has to enter a license key that has to be activated.                                                                                                                          |
| <p><strong>GetLicense</strong><br><strong>GetLicenseAsync</strong></p> | Gets information about a license for a given module system name such as the remaining demo days.                                                                                                                                                                  |
| **ActivateAsync**                                                      | Activates a license key. This is done when the user enters their license key in administration backend.                                                                                                                                                           |
| <p><strong>Check</strong><br><strong>CheckAsync</strong></p>           | Checks the state of a license for a given module system name and returns various information about the license. If you need a stringified version of the result (German and English localization supported), call the `ToString()` method of the returned object. |
| <p><strong>CheckState</strong><br><strong>CheckStateAsync</strong></p> | Same as _Check_ or _CheckAsync_ but just returns the license state.                                                                                                                                                                                               |
| <p><strong>ResetState</strong><br><strong>ResetStateAsync</strong></p> | The license status is cached internally. Only after a certain period there is a live check against the server. With this method, the cached status is reset and immediately checked again live.                                                                   |

### LicenseRequired filter attribute <a href="#howtomakeapluginlicensable-thelicenserequiredfilterattribute" id="howtomakeapluginlicensable-thelicenserequiredfilterattribute"></a>

You can decorate either a whole controller class or just a single action method with the `LicenseRequiredAttribute`. This attribute internally calls `LicenseChecker.CheckAsync()` right before your action is processed, giving it the opportunity to block the action. If the license isn't valid, the _LicenseRequired_ view will be rendered by default, which you can override by setting the property `ViewName` on the attribute. Alternatively, you could just display a notification for which you can use the properties `NotifyOnly`, `NotificationMessage` and `NotificationMessageType`. AJAX requests will be recognized automatically, and a suitable response will be generated according to the negotiated content type (either JSON or HTML).

If you want to block certain methods or even your entire plugin when it is in demo mode, you need to set the property `BlockDemo` to _True,_ otherwise everything will be accessible in the demo mode, as the value of `BlockDemo` is _False_ by default. All properties of `LicenseRequiredAttribute`:

| Property                    | Description                                                                                                                                                                                                                                                |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **ModuleSystemName**        | Gets or sets the module system name.                                                                                                                                                                                                                       |
| **LayoutName**              | Gets or sets the name of the layout view for _License Required_ message. _\~/Views/Shared/Layouts/\_Layout.cshtml_ by default.                                                                                                                             |
| **ViewName**                | Gets or sets the name of the view for _License Required_ message. _LicenseRequired_ by default.                                                                                                                                                            |
| **Result**                  | <p>Gets or sets the action result if the module is in an unlicensed state. Possible values:<br><em>Block</em>: return a result with a license required warning.<br><em>NotFound</em>: return not found result.<br><em>Empty</em>: return empty result.</p> |
| **BlockDemo**               | A value indicating whether to block the request if license is in demo mode.                                                                                                                                                                                |
| **NotifyOnly**              | A value indicating whether to only output a notification message and not to replace the action result.                                                                                                                                                     |
| **NotificationMessage**     | Gets or sets the message to output if the module is in an unlicensed state.                                                                                                                                                                                |
| **NotificationMessageType** | Gets or sets the type of the notification message. Possible values are _error_, _success_, _info_, _warning_. Default is _error_.                                                                                                                          |

## Usage scenario <a href="#howtomakeapluginlicensable-usagescenario" id="howtomakeapluginlicensable-usagescenario"></a>

Imagine you have developed a module to communicate with an ERP system. Furthermore, your module transmits data to a web service whenever an order is placed in your shop and consumes another web service to keep your product data up-to-date. If you decide to allow the product data to be updated completely in the demo mode of your plugin, it may be sufficient for the plugin user to import the product data only once. Therefore, you should interrupt the routine that's responsible for updating product data after a certain number of products have been updated. To do so, you would use the `CheckStateAsync()` method, which checks whether the state is _Demo_ and stops the routine accordingly (see code example 1). This way, the user can see a demonstration of the actual function without getting the whole pie.

Order events should, of course, be processed and transmitted to the ERP system completely for demonstration purposes, as it's way too difficult to keep track of the number of processed orders. However, when the demonstration period is over, no more orders should be processed. Therefore, you would use the `CheckStateAsync()` method to check whether the state is _Unlicensed_ and to stop the event accordingly (see code example 2).

{% code title="Code example 1" %}
```csharp
private async Task ProcessProducts()
{
    var state = await LicenseChecker.CheckStateAsync("MyCompany.MyModule");
    // ...
    if (state == LicensingState.Demo)
    {
        // Leave after 5 products if plugin is in demo mode.
        products = products.Take(5);
    }
    
    foreach (var product in products)
    {
        await UpdateProduct(product);
    }
    // ...
```
{% endcode %}

{% code title="Code example 2" %}
```csharp
public class Events : IConsumer
{
    public async Task HandleEventAsync(OrderPlacedEvent message)
    {
        var state = await LicenseChecker.CheckStateAsync("MyCompany.MyModule");
        if (state == LicensingState.Unlicensed)
            return;
        // ...
```
{% endcode %}

## Examples and special case <a href="#howtomakeapluginlicensable-examplesandspecialcases" id="howtomakeapluginlicensable-examplesandspecialcases"></a>

In background task there is no HTTP request object, so a license is checked against the current store URL. But for an export provider this is a bit inaccurate. The provider will want to check the license against the URL of the store whose data is being exported. It can use the `ExportExecuteContext` to get the URL.

```csharp
protected override async Task ExportAsync(ExportExecuteContext context,
    CancellationToken cancelToken)
{
    var license = await LicenseChecker.CheckAsync("MyCompany.MyModule",
        (string)context.Store.Url);

    if (license.State == LicensingState.Unlicensed)
    {
        context.Log.Error(HtmlUtility.ConvertHtmlToPlainText(
            license.ToString(), true, true));
        context.Abort = DataExchangeAbortion.Hard;
        return;
    }    
    // TODO: export...
}
```

Additionally, a payment plugin can hide its payment methods at checkout if there is no active license. To enable this to work, you must override the `PaymentMethodBase.IsActive` property.

```csharp
public override bool IsActive
{
    get
    {
        try
        {
            var state = LicenseChecker.CheckState("MyCompany.MyModule");
            return (state != LicensingState.Unlicensed);
        }
        catch (Exception ex)
        {
            // _logger is an instance of ILogger
            _logger.Error(ex);
        }
        return true;
    }
}

// Or when using the mentioned throttle check simply:
public override bool IsActive
    => Module.CheckLicense(false).Await();
```
