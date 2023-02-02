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

### Properties of PaymentMethodBase

| Property                   | Description                                                                                                                                                                                                                                                                    |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **IsActive**               | A value indicating whether the payment method is active and should be offered to customers. Typically used for license checks. See also [payment method filter](creating-a-payment-provider.md#payment-method-filter).                                                         |
| **PaymentMethodType**      | See [table](creating-a-payment-provider.md#payment-method-types) below for available values. Choose a type that best suits your payment method.                                                                                                                                |
| **RequiresInteraction**    | A value indicating whether the payment method requires user input in checkout before proceeding, e.g. credit card or direct debit payment.                                                                                                                                     |
| **SupportCapture**         | A value indicating whether (later) capturing the payment amount is supported, for instance when the goods are shipped.. If `true`, then you must overwrite the method `CaptureAsync`.                                                                                          |
| **SupportPartiallyRefund** | A value indicating whether a partial refund is supported.  If `true`, then you must overwrite the method `RefundAsync`.                                                                                                                                                        |
| **SupportRefund**          | A value indicating whether a full refund is supported. If `true`, then you must overwrite the method `RefundAsync`.                                                                                                                                                            |
| **SupportVoid**            | A value indicating whether cancellation of the payment (transaction) is supported. If `true`, then you must overwrite the method `VoidAsync`.                                                                                                                                  |
| **RecurringPaymentType**   | <p>The type of recurring payment. Available values are:<br><em>NotSupported</em>: recurring payment is not supported.<br><em>Manual</em>: recurring payment is completed manually by admin.<br><em>Automatic</em>: recurring payment is processed on payment gateway site.</p> |

### Methods of PaymentMethodBase

| Method                           | Description                                                                                                                                                                                                                                                                                           |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **GetPaymentInfoWidget**         | Gets the widget invoker for payment info. The payment info is displayed on checkout's payment page. Return `null` when there is nothing to render.                                                                                                                                                    |
| **GetPaymentFeeInfoAsync**       | Gets the additional handling fee for a payment.                                                                                                                                                                                                                                                       |
| **GetPaymentInfoAsync**          | Gets a `ProcessPaymentRequest`. Called after the customer selected a payment method on checkout's payment page. Typically used to specify an `ProcessPaymentRequest.OrderGuid` that can be sent to the payment provider before the order is placed. It will be saved later when the order is created. |
| **ValidatePaymentDataAsync**     | Validates the payment data entered by customer on checkout's payment page.                                                                                                                                                                                                                            |
| **GetPaymentSummaryAsync**       | Gets a short summary of payment data entered by customer in checkout that is displayed on the checkout's confirm page. Typically used to display the brand name and a masked credit card number.                                                                                                      |
| **PreProcessPaymentAsync**       | Pre-process a payment. Called immediately before `ProcessPaymentAsync`. Can be used, for example, to complete required data such as the billing address.                                                                                                                                              |
| **ProcessPaymentAsync**          | Process a payment. Intended for main payment processing like payment authorization.                                                                                                                                                                                                                   |
| **PostProcessPaymentAsync**      | Post-process payment. Called after an order has been placed or when customer re-starts the payment (if supported). Used, for example, to redirect to a payment page to complete the payment after (!) the order has been placed.                                                                      |
| **CanRePostProcessPaymentAsync** | Gets a value indicating whether customers can complete a payment after order has been placed but not yet completed (only for redirection payment methods).                                                                                                                                            |
| **CaptureAsync**                 | Captures a payment amount.                                                                                                                                                                                                                                                                            |
| **RefundAsync**                  | Fully or partially refunds a payment amount.                                                                                                                                                                                                                                                          |
| **VoidAsync**                    | Cancels a payment (transaction).                                                                                                                                                                                                                                                                      |
| **ProcessRecurringPaymentAsync** | Processes a recurring payment.                                                                                                                                                                                                                                                                        |
| **CancelRecurringPaymentAsync**  | Cancels a recurring payment.                                                                                                                                                                                                                                                                          |

### Payment method types

| Payment method type        | Description                                                                                                                                                              |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Unknown**                | Type is unknown.                                                                                                                                                         |
| **Standard**               | All payment information is entered on the payment selection page.                                                                                                        |
| **Redirection**            | A customer is redirected to a third-party site to complete the payment after (!) the order has been placed. This type of processing is required for older payment types. |
| **Button**                 | Payment via button on cart page.                                                                                                                                         |
| **StandardAndButton**      | All payment information is entered on the payment selection page and is available via button on cart page.                                                               |
| **StandardAndRedirection** | Payment information is entered in checkout and customer is redirected to complete payment (e.g. 3D Secure) after the order has been placed.                              |

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
