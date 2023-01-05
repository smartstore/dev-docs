# Prerequisites

The Smartstore Web API requires configuration by the storekeeper in order to start working. First of all, the storekeeper must install the Web API module in the backend of Smartstore. The module technology gives him the opportunity to activate or deactivate the entire Web API at any time without any impact on the online store.

The next step is to configure the API on the module's configuration page. The main thing here is to provide individual members access to the API and the data of the online store. Therefore, the storekeeper can create a public and a secret key for each registered member. Only a registered member with both keys has access to the API. To exclude a member from the API, the storekeeper can either delete the keys of the member (permanent exclusion) or disable them (temporary exclusion). A member's roles and rights are taken into consideration when data is accessed via the API.

The secret key should only be known by the storekeeper and the member accessing the Web API.

{% hint style="success" %}
To develop an own Web API consumer, implementers should be familiar with REST API consumer implementation and in particular with the OData provider.
{% endhint %}
