# Modularity & Providers

## Providers

A provider is a design pattern to integrate and swap a code extension more easily and flexibly. A good example are payment methods. If a developer wants to implement multiple payment methods of one payment company, he can create a single module (representing the payment company), which contains a provider for each payment method. Besides [IPaymentMethod](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Checkout/Payment/Service/IPaymentMethod.cs), there are other providers like [IExportProvider](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/DataExchange/Export/IExportProvider.cs), [IShippingRateComputationMethod](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Checkout/Shipping/Services/IShippingRateComputationMethod.cs), [IExchangeRateProvider](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Common/Services/IExchangeRateProvider.cs), [IExternalAuthenticationMethod](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Identity/Services/IExternalAuthenticationMethod.cs) etc.

Each of these provider interfaces is derived from the marker interface [IProvider](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore/Engine/Modularity/IProvider.cs) in order to be able to identify it uniformly as a provider. The [generic provider class](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Modularity/Provider.cs) encapsulates the respective interface and enriches it with general properties like [ProviderMetadata](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Modularity/ProviderMetadata.cs). As a result you get an abstract, handy and API friendly construct such as `Provider<IPaymentMethod>`.

### Metadata

| Name             | Implement                 | Description                                                                                                                                                                         |
| ---------------- | ------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| FriendlyName     | FriendlyNameAttribute     | The English name of the provider. Localizable by using resource key _Plugins.FriendlyName.\<ProviderSystemName>_                                                                    |
| Description      | FriendlyNameAttribute     | The English description of the provider. Localizable by using resource key _Plugins.Description.\<ProviderSystemName>_                                                              |
| DisplayOrder     | OrderAttribute            | The display order in the providers list.                                                                                                                                            |
| DependentWidgets | DependentWidgetsAttribute | Widgets which are automatically (de)activated when the provider gets (de)activated. Useful in scenarios where separate widgets are responsible for the displaying of provider data. |
| ExportFeatures   | ExportFeaturesAttribute   | Data processing types supported by an export provider.                                                                                                                              |
| IsConfigurable   | IConfigurable             | A value indicating wether the provider is configurable.                                                                                                                             |
| IsEditable       | IUserEditable             | A value indicating wether the metadata is editable by the user.                                                                                                                     |
| IsHidden         | IsHiddenAttribute         | A value indicating wether the provider is hidden. A hidden provider can only be used programmatically but not by the user through the user interface.                               |
| SystemName       | SystemNameAttribute       | Unique SystemName of the provider, e.g. _Payments.AmazonPay._                                                                                                                       |

HINT: a provider can be activated or deactivated via the provider list. For example, a deactivated payment provider does not appear in the checkout payment method list.
