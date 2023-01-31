# Licensable modules

## Overview

A module can be marked as licensable piece of code by decorating the module class (inherited from `ModuleBase`) with the `LicensableModuleAttribute`.

HINT: There must be a dependency on the _Smartstore.Web.Common_ project for the module to use the licensing component.

To be able to fully use the module, the user must enter a license key in the administration backend, which he has previously purchased on the Smartstore marketplace and which needs to be activated. The activation is done via the administration backend.

HINT: A license key is only valid for the IP address that activated the key.

The module should perform a license check at the beginning of important functions and disable or restrict them if there is no valid license.

`LicensableModuleAttribute` has an option `HasSingleLicenseForAllStores` related to multi-stores. If set to _True_, then the license key has no domain binding and is therefore valid for all stores. Recommended is the default value _False_, i.e. a new license key is required for each store.

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

HINT: The license check is always performed against the domain of the current request. If you want to check against another domain, you have to pass the corresponding URL as a parameter when calling `CheckState`.

WARN: If the license check is done within a task, an URL should always be supplied, because no HTTP request exists there. For instance, an export provider may check as follows:

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

### Available methods

| Method                                                                 | Description                                                                                                                                                                                     |
| ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **IsLicensableModule**                                                 | Gets a value indicating whether a module is a licensed piece of code where the user has to enter a license key that has to be activated.                                                        |
| <p><strong>GetLicense</strong><br><strong>GetLicenseAsync</strong></p> | Gets information about a license for a given module system name such as the remaining demo days.                                                                                                |
| **ActivateAsync**                                                      | Activates a license key. This is done when the user enters their license key in administration backend.                                                                                         |
| <p><strong>Check</strong><br><strong>CheckAsync</strong></p>           | Checks the state of a license for a given module system name and returns various information about the license.                                                                                 |
| <p><strong>CheckState</strong><br><strong>CheckStateAsync</strong></p> | Same as _Check_ or _CheckAsync_ but just returns the license state.                                                                                                                             |
| <p><strong>ResetState</strong><br><strong>ResetStateAsync</strong></p> | The license status is cached internally. Only after a certain period there is a live check against the server. With this method, the cached status is reset and immediately checked again live. |

## Filter when unlicensed

There is also a `LicenseRequiredAttribute` available which filters an action method if the module is in an unlicensed state. In this case it blocks the request and renders a corresponding warning. Optionally, a `NotFoundResult` or an `EmptyResult` is also possible. You can decorate controllers or action methods with `LicenseRequiredAttribute` and do not need any license check inside your related code.
