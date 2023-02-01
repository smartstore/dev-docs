# Creating a Payment provider

## Overview

A payment provider represents a payment method with which an order can be paid in the frontend. Furthermore, it contains other optional methods, e.g. to later capture the payment amount when the goods are shipped or to perform a refund. It is recommended to learn about the general functioning of a [provider](../../../framework/platform/modularity-and-providers.md#providers) before going on.

## Payment provider

The payment provider contains all important functions for a specific payment method. A module can contain any number of payment providers and thus any number of payment methods.

TIP: It is recommended to implement the payment methods of a payment company (like PayPal) in one module, not in many. In other words, the module should represent a certain payment company.

To create a payment provider add a class that inherits from `PaymentMethodBase` and optionally implements `IConfigurable`.

{% code title="A simple payment provider" %}
```csharp
[SystemName("MyCompany.MyCreditCardPayment")]
[FriendlyName("My credit card payment")]
[Order(1)]
public class MyCreditCardProvider : PaymentMethodBase, IConfigurable
{
    public RouteInfo GetConfigurationRoute()
        => new(nameof(MyPaymentAdminController.Configure), "MyPaymentAdmin", new { area = "Admin" });

    public override Widget GetPaymentInfoWidget()
        => new ComponentWidget(typeof(MyCreditCardInfoViewComponent));

    public override Task<ProcessPaymentResult> ProcessPaymentAsync(ProcessPaymentRequest processPaymentRequest)
    {
        // TODO: authorize payment...

        return new ProcessPaymentResult
        {
            NewPaymentStatus = PaymentStatus.Authorized
        };
    }
}
```
{% endcode %}

For details about `SystemName` etc. see [providers](../../../framework/platform/modularity-and-providers.md#providers). There are a number of properties and methods that the provider should override.

| Property or method         | Description                                                                           |
| -------------------------- | ------------------------------------------------------------------------------------- |
| **SupportCapture**         | Gets a value indicating whether capturing the payment amount is supported.            |
| **SupportPartiallyRefund** | Gets a value indicating whether a partial refund is supported.                        |
| **SupportRefund**          | Gets a value indicating whether a full refund is supported.                           |
| **SupportVoid**            | Gets a value indicating whether cancellation of the payment transaction is supported. |
|                            |                                                                                       |
|                            |                                                                                       |
|                            |                                                                                       |

## Payment method filter

Payment methods can be filtered using [IPaymentMethodFilter](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Checkout/Payment/Service/IPaymentMethodFilter.cs). Such an implementation is useful, for example, when a payment method should generally only be offered if a certain shopping cart amount is exceeded.

{% code title="Example of a payment filter" %}
```csharp
public partial class MyCustomPaymentFilter : IPaymentMethodFilter
{
    // Check provider acceptance once a day.
    const int CheckAcceptanceHours = 24;

    private readonly Lazy<MyPaymentProviderHttpClient> _httpClient;
    private readonly Lazy<IOrderCalculationService> _orderCalculationService;
    private readonly Lazy<ISettingFactory> _settingFactory;
    private readonly ILogger _logger;

    public MyCustomPaymentFilter(
        Lazy<MyPaymentProviderHttpClient> httpClient,
        Lazy<IOrderCalculationService> orderCalculationService,
        Lazy<ISettingFactory> settingFactory,
        ILogger logger)
    {
        _httpClient = httpClient;
        _orderCalculationService = orderCalculationService;
        _settingFactory = settingFactory;
        _logger = logger;
    }

    public async Task<bool> IsExcludedAsync(PaymentFilterRequest request)
    {
        try
        {
            if (request.PaymentMethod.Metadata.SystemName.EqualsNoCase("Payments.MyCustomPurchaseByInstallment"))
            {
                var settings = await _settingFactory.Value.LoadSettingsAsync<MyPaymentSettings>(request.StoreId);
                if (!settings.HasCredentials() || !settings.IsAcceptedByPaymentProvider)
                {
                    return true;
                }

                if (request.Cart != null)
                {
                    Money? cartTotal = await _orderCalculationService.Value.GetShoppingCartTotalAsync(request.Cart);
                    if (cartTotal == null || cartTotal.Value <= decimal.Zero || cartTotal.Value < settings.FinancingMin || cartTotal.Value > settings.FinancingMax)
                    {
                        // Cart total is not financeable.
                        return true;
                    }
                }

                if ((DateTime.UtcNow - settings.LastAcceptanceCheckedOn).TotalHours > CheckAcceptanceHours)
                {
                    try
                    {
                        await _httpClient.Value.UpdateAcceptedByPaymentProviderAsync(settings, request.StoreId);
                        if (!settings.IsAcceptedByPaymentProvider)
                        {
                            // Payment provider does not accept any further payment.
                            return true;
                        }
                    }
                    catch (Exception ex)
                    {
                        _logger.Error(ex);
                    }
                }
            }
        }
        catch (Exception ex)
        {
            _logger.Error(ex);
        }

        return false
    }
}
```
{% endcode %}
