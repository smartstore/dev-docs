# Web API in detail

You can consume API services through HTTPS calls in a RESTful manner by using HTTPS methods GET, POST (insert), PUT (update), PATCH (partially update) and DELETE. OData query options (such a $expand, $filter, $top, $select, etc.) and API specific options (such as `SmApiFulfillCountry`) can be transferred through query strings.

Paging is required if you want to query multiple records. You can do that with OData query options $skip and $top. The maximum value for $top is returned in the `Smartstore-Api-MaxTop` header field. The value is configurable by storekeeper.

Due to [Basic Authentication](authentication.md), it is mandatory to send requests over HTTPS. For HTTP, a status code 421 _Misdirected Request_ is returned. The only exception is that developers can send requests over HTTP to their local development environment.

A request body needs to be UTF-8 encoded.

## Special request HTTP header fields

| Name: value                                                     | Required |                                                                          Remarks                                                                          |
| --------------------------------------------------------------- | -------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------: |
| <p><strong>Accept</strong>:<br>application/json</p>             | yes      |                                                             Only _application/json_ is valid.                                                             |
| <p><strong>Accept-Charset</strong>:<br>UTF-8</p>                | yes      |                                                                                                                                                           |
| <p><strong>Authorization</strong>:<br>Basic &#x3C;key pair></p> | yes      |                                                          See [Authentication](authentication.md).                                                         |
| <p><strong>Prefer</strong>:<br>return=representation</p>        | no       | Can be sent for PUT and PATCH requests if the API should answer with status code 200 and entity content response. Otherwise 204 _No Content_ is returned. |

## Special response HTTP header fields

| Name: example value                                                                           | Description                                                                                                                            |
| --------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| <p><strong>Smartstore-Api-AppVersion</strong>:<br>5.1.0.0</p>                                 | Smartstore version used by store.                                                                                                      |
| <p><strong>Smartstore-Api-Version</strong>:<br>1 5.0</p>                                      | The highest API version supported by the server (unsigned integer) and the version of the installed API module (floating-point value). |
| <p><strong>Smartstore-Api-MaxTop</strong>:<br>120</p>                                         | The maximum value for OData $top query option. The default value is 120 and is configurable by storekeeper.                            |
| <p><strong>Smartstore-Api-Date</strong>:<br>2022-11-11T14:35:33.7772907Z</p>                  | The current server date and time in ISO-8601 UTC.                                                                                      |
| <p><strong>Smartstore-Api-CustomerId</strong>:<br>1234</p>                                    | The customer identifier of the authenticated API user. Returned only if authentication is successful.                                  |
| <p><strong>Smartstore-Api-AuthResultId</strong>:<br>5</p>                                     | The ID for the reason why the request was denied. Returned only if the authentication failed. See table below.                         |
| <p><strong>Smartstore-Api-AuthResultDesc</strong>:<br>UserDisabled</p>                        | A short description for the reason why the request was denied. Returned only if the authentication failed. See table below.            |
| <p><strong>WWW-Authenticate</strong>:<br>Basic realm="Smartstore.WebApi", charset="UTF-8"</p> | The name of the authentication method that failed.                                                                                     |



## Reasons for denial

| Smartstore-Api-AuthResultId | Smartstore-Api-AuthResultDesc | Description                                                                                         |
| :-------------------------: | ----------------------------- | --------------------------------------------------------------------------------------------------- |
|              0              | ApiDisabled                   | The API is disabled.                                                                                |
|              1              | SslRequired                   | HTTPS is required in any case unless the request takes place in a development environment.          |
|              2              | InvalidAuthorizationHeader    | The HTTP authorization header is missing or invalid. Must include a pair of public and secret keys. |
|              2              | InvalidCredentials            | The credentials sent by the HTTP authorization header do not match those of the user.               |
|              4              | UserUnknown                   | The user is unknown.                                                                                |
|              5              | UserDisabled                  | The user is known but his access via the API is disabled.                                           |



## Query options

[OData query options](https://learn.microsoft.com/en-us/odata/concepts/queryoptions-overview) allow manipulation of data queries such as sorting, filtering, paging etc. They are sent in the query string of the request URL.

Custom query options should lighten the workload with the Smartstore Web API, especially when you work with entity relationships.

#### SmApiFulfill{property\_name}

Entities are often in multiple relationships with other entities. In most cases, an ID has to be set in order to create or change such relationships. Most of the time, however, this ID is unknown to you. To reduce the number of API round trips, this option can set an entity relationship indirectly. Example: You want to add a German address, but you don't know the ID of the German country entity which is required for inserting an address. Rather than calling the API again to get the ID, you can add the query option `SmApiFulfillCountry=DE`, and the API resolves the relationship automatically. The API can fulfill the following properties:

| property\_name | Example                      | Description                               |
| -------------- | ---------------------------- | ----------------------------------------- |
| Country        | SmApiFulfillCountry=USA      | The two or three letter ISO country code. |
| StateProvince  | SmApiFulfillStateProvince=CA | The abbreviation of a state or province.  |
| Language       | SmApiFulfillLanguage=de-DE   | The culture of a language.                |
| Currency       | SmApiFulfillCurrency=EUR     | The ISO code of a currency.               |

{% hint style="warning" %}
`SmApiFulfill` sets the relationship only if none exists yet. An existing relationship cannot be changed by this option.
{% endhint %}
