# ✔ Web API in detail

You can consume API services in a RESTful manner via HTTPS calls by using HTTPS methods:

* `GET`: Get the resource.
* `POST`: Create a new resource.
* `PUT`: Update or replace the existing resource.
* `PATCH`: Update or replace a part of the existing resource.
* `DELETE`: Remove the resource.

OData query options (such a `$expand`, `$filter`, `$top`, `$select`, etc.) and API specific options (such as `SmApiFulfillCountry`) can be passed via query strings.

Paging is required if you want to query multiple records. You can do this with the OData query options `$skip` and `$top`. The maximum value for `$top` is returned in the `Smartstore-Api-MaxTop` header field. This value is configurable by the store owner.

Due to [Basic Authentication](https://app.gitbook.com/o/jug3iI9jtm3q3KRxHi73/s/DOZxBBKmB9QIuwBDsOtV/framework/web-api/authentication), it is mandatory to send requests via HTTPS. HTTP will return a `421 Misdirected Request` status code. The only exception is that developers can send requests via HTTP to their local development environment.

A request body must be _UTF-8_ encoded.

## Request HTTP header fields

| Name: value                                                     | Required |                                                                           Remarks                                                                           |
| --------------------------------------------------------------- | -------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------: |
| <p><strong>Accept</strong>:<br>application/json</p>             | yes      |                                                              Only _application/json_ is valid.                                                              |
| <p><strong>Accept-Charset</strong>:<br>UTF-8</p>                | yes      |                                                                                                                                                             |
| <p><strong>Authorization</strong>:<br>Basic &#x3C;key pair></p> | yes      |                                                           See [Authentication](authentication.md).                                                          |
| <p><strong>Prefer</strong>:<br>return=representation</p>        | no       | Can be sent for PUT and PATCH requests if the API should answer with status code `200` and entity content response, otherwise `204 No Content` is returned. |

## Response HTTP header fields

| Name: example value                                                                                        | Description                                                                                                                                                                |
| ---------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <p><strong>Smartstore-Api-AppVersion</strong>:<br><code>5.1.0.0</code></p>                                 | Smartstore version used by the store.                                                                                                                                      |
| <p><strong>Smartstore-Api-Version</strong>:<br><code>1 5.0</code></p>                                      | The highest API version supported by the server (unsigned integer) and the version of the installed API module (floating-point value).                                     |
| <p><strong>Smartstore-Api-MaxTop</strong>:<br><code>120</code></p>                                         | The maximum value for OData `$top` query option. The default value is `120` and is configurable by store owner.                                                            |
| <p><strong>Smartstore-Api-Date</strong>:<br><code>2022-11-11T14:35:33.7772907Z</code></p>                  | The current server date and time in _ISO-8601 UTC_.                                                                                                                        |
| <p><strong>Smartstore-Api-CustomerId</strong>:<br><code>1234</code></p>                                    | The customer identifier of the authenticated API user. Returned only if authentication is successful.                                                                      |
| <p><strong>Smartstore-Api-AuthResultId</strong>:<br><code>5</code></p>                                     | <p>The ID of the reason why the request was denied. Returned only if the authentication failed.</p><p>See <a href="authentication.md">authentication</a>.</p>              |
| <p><strong>Smartstore-Api-AuthResultDesc</strong>:<br><code>UserDisabled</code></p>                        | <p>A short description of the reason why the request was denied. Returned only if the authentication failed.</p><p>See <a href="authentication.md">authentication</a>.</p> |
| <p><strong>WWW-Authenticate</strong>:<br><code>Basic realm="Smartstore.WebApi", charset="UTF-8"</code></p> | The name of the failed authentication method.                                                                                                                              |

## Query options

[OData query options](https://learn.microsoft.com/en-us/odata/concepts/queryoptions-overview) allow manipulation of data queries like sorting, filtering, paging etc. They are sent in the query string of the request URL.

Custom query options should make the Smartstore Web API easier to use, especially when working with entity relationships.

#### SmApiFulfill{property\_name}

Entities often have multiple relationships with other entities. In most cases, an ID must be set to create or modify them. But often you do not know the ID. To reduce the number of API round trips, this option can set an entity relationship indirectly.

For example, imagine you want to add a German address, but you don't have the ID of the German country entity, required for address insertion. Instead of calling the API again to get the ID, you can add the query option `SmApiFulfillCountry=DE` and the API will automatically resolve the relationship.

The API can fulfill the following properties:

| property\_name  | Example                        | Description                               |
| --------------- | ------------------------------ | ----------------------------------------- |
| `Country`       | `SmApiFulfillCountry=USA`      | The two or three letter ISO country code. |
| `StateProvince` | `SmApiFulfillStateProvince=CA` | The abbreviation of a state or province.  |
| `Language`      | `SmApiFulfillLanguage=de-DE`   | The culture of a language.                |
| `Currency`      | `SmApiFulfillCurrency=EUR`     | The ISO code of a currency.               |

{% hint style="warning" %}
`SmApiFulfill` sets the relationship only if none exists yet. An existing relationship cannot be modified with this option.
{% endhint %}
