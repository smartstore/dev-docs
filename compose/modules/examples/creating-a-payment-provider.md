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

## Custom checkout state

In most cases, session-based data must be stored during the checkout in order to communicate with the payment provider's API. This includes, for example, the ID of the payment transaction or session. Use the `CheckoutState` property of [ICheckoutStateAccessor](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Checkout/Orders/Services/ICheckoutStateAccessor.cs) to store any custom session data during checkout. Your state object must inherit from `ObservableObject`.

{% code title="Custom checkout state object example" %}
```csharp
[Serializable]
public class MyCustomCheckoutState : ObservableObject
{
    /// <summary>
    /// Technical transaction ID used in URLs for communication between 
    /// the module and API.
    /// </summary>
    public string Id
    {
        get => GetProperty<string>();
        set => SetProperty(value);
    }

    public string TransactionDataHash
    {
        get => GetProperty<string>();
        set => SetProperty(value);
    }

    public MyPaymentDecision Decision
    {
        get => GetProperty<MyPaymentDecision>();
        set => SetProperty(value);
    }
}
```
{% endcode %}

The `ObservableObject` allows changes to the state object to be automatically applied to the session. So no `Set` or `Save` etc. has to be executed.

```csharp
public override Task<ProcessPaymentRequest> GetPaymentInfoAsync(IFormCollection form)
{
    var state = _checkoutStateAccessor.CheckoutState.GetCustomState<MyCustomCheckoutState>();
    var (id, hash) = await CreateTransaction();
    state.Id = id;
    state.TransactionDataHash = hash;

    return new ProcessPaymentRequest
    {
        OrderGuid = Guid.NewGuid()
    };
}

private Task<(string ID, string DataHash)> CreateTransaction()
{
    // TODO: create a transaction using API and get back its ID.
    var id = "1.de.4145.1-0303135329-211";
    var hash = "1c05aa56405c447e6678b7f312
    return Task.FromResult((id, hash));
}
```

WARN: the checkout state has a limited scope by design. This starts from the shopping cart page and ends with the creation of the order. On the checkout completed page, for example, the checkout state no longer exists. If you want to keep data longer, then you have to either store it in the database or put it in a separate session object.

## Storing additional order data

TODO: explain...

## Webhooks and IPNs

Webhooks and IPNs (Instant Payment Notification) are HTTP-based callback functions the payment provider uses to send payment related messages to a shop, e.g. a payment status change. It is a kind of cross web application event system. The message handler typically updates the [payment status](creating-a-payment-provider.md#payment-status) of an order according to the message. See the `AmazonPayController` as an example of an IPN handler.

WARN: In case of misuse, these messages do not originate from the payment provider. The transmitted data should not be processed or stored directly. Instead, another call to the API of the payment provider is required and the returned data is then processed. This ensures the authenticity of the data. Some providers additionally give out IP addresses from which their IPNs originate, which can then be additionally verified.

### WebhookEndpoint attribute

Each action method that receives and processes webhooks or IPNs must be decorated with the `WebhookEndpointAttribute`. It's a marker filter with the purpose to suppress customer resolution for such endpoints. The attribute prevents unnecessary creation of guest customers (as long as HTTP callbacks do not send any cookies). Instead, a system account with the system name `WebhookClient` is used.

### Testing webhooks and IPNs

We recommend using the [ngrok](https://ngrok.com/) service to test webhooks and IPNs. The service allows you to receive webhook messages and IPNs over an HTTP or HTTPS tunnel locally on localhost. It comes with a console windows app named `ngrok.exe` that you have to download. The messages then reach your local shop through this tunnel, as long as the `ngrok.exe` is running and the tunnel is alive. The advantage of this service is that it is simply attached to your development environment. You don't have to change or switching anything at your existing system like firewall, host, SSL, DDNS etc.

TIP: it will be even easier if you run `ngrok.exe` via a simple batch script which already contains your localhost URL.

{% code title="ngrok-localhost-59318.bat" %}
```actionscript
%SystemRoot%\system32\cmd /C "C:\Tools\ngrok\ngrok.exe http 59318 -host-header="localhost:59318""
pause
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/ngrok.png" alt=""><figcaption><p>Active ngrok HTTP tunnel making a local shop reachable under localhost:59318.</p></figcaption></figure>

Under **Forwarding** you see the two URLs that you can use to receive messages. You can enter them in the backend of the payment provider, together with the path to the action method that receives and processes payment messages. If the payment provider does not provide a setting for this and the URL must be supplied by code, then you can also add a setting for the ngrok URL in the configuration of your payment method, which should only be visible in developer mode.

## Appendix

### Payment method types

| Payment method type        | Description                                                                                                                                                              |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Unknown**                | Type is unknown.                                                                                                                                                         |
| **Standard**               | All payment information is entered on the payment selection page.                                                                                                        |
| **Redirection**            | A customer is redirected to a third-party site to complete the payment after (!) the order has been placed. This type of processing is required for older payment types. |
| **Button**                 | Payment via button on cart page.                                                                                                                                         |
| **StandardAndButton**      | All payment information is entered on the payment selection page and is available via button on cart page.                                                               |
| **StandardAndRedirection** | Payment information is entered in checkout and customer is redirected to complete payment (e.g. 3D Secure) after the order has been placed.                              |

### Payment status

|                       |                                                                                                                                                           |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Pending**           | The initial payment status if no further status information is available yet.                                                                             |
| **Authorized**        | The payment has been authorized (but not captured) by the payment provider. Usually this means that the payment amount is reserved for later capturing.   |
| **Paid**              | The payment has been captured against the payment gateway. It does not necessarily mean that the paid amount has been credited to the merchant's account. |
| **PartiallyRefunded** | The paid amount has been partially refunded.                                                                                                              |
| **Refunded**          | The paid amount has been fully refunded.                                                                                                                  |
| **Voided**            | The payment has been cancelled.                                                                                                                           |
