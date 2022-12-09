# Web API in detail

You can consume API services through HTTPS calls in a RESTful manner by using HTTPS methods GET, POST (insert), PUT (update), PATCH (partially update) and DELETE. OData query options (such a $expand, $filter, $top, $select, etc.) and API specific options (such as `SmApiFulfillCountry`) can be transferred through query strings.

Paging is required if you want to query multiple records. You can do that with OData query options $skip and $top. The maximum value for $top is returned in the `Smartstore-Api-MaxTop` header field. The value is configurable by storekeeper.

Due to [Basic Authentication](authentication.md), it is mandatory to send requests over HTTPS. For HTTP, a status code 421 _Misdirected Request_ is returned. The only exception is that developers can send requests over HTTP to their local development environment.

A request body needs to be UTF-8 encoded.

## Special request HTTP header fields

| Key: value                                     | Required |                                                                          Remarks                                                                          |
| ---------------------------------------------- | -------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------: |
| <p>Accept:<br>application/json</p>             | yes      |                                                             Only _application/json_ is valid.                                                             |
| <p>Accept-Charset:<br>UTF-8</p>                | yes      |                                                                                                                                                           |
| <p>Authorization:<br>Basic &#x3C;key pair></p> | yes      |                                                          See [Authentication](authentication.md).                                                         |
| <p>Prefer:<br>return=representation</p>        | no       | Can be sent for PUT and PATCH requests if the API should answer with status code 200 and entity content response. Otherwise 204 _No Content_ is returned. |

## Special response HTTP header fields

| Key: example value                                                           | Remarks                                                                                                                                |
| ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| <p>Smartstore-Api-AppVersion:<br>5.1.0.0</p>                                 | Smartstore version used by store.                                                                                                      |
| <p>Smartstore-Api-Version:<br>1 5.0</p>                                      | The highest API version supported by the server (unsigned integer) and the version of the installed API module (floating-point value). |
| <p>Smartstore-Api-MaxTop:<br>120</p>                                         | The maximum value for OData $top query option. The default value is 120 and is configurable by storekeeper.                            |
| <p>Smartstore-Api-Date:<br>2022-11-11T14:35:33.7772907Z</p>                  | The current server date and time in ISO-8601 UTC.                                                                                      |
| <p>Smartstore-Api-CustomerId:<br>1234</p>                                    | The customer identifier of the authenticated API user. Returned only if authentication is successful.                                  |
| <p>Smartstore-Api-AuthResultId:<br>5</p>                                     | The ID for the reason why the request was denied. Returned only if the authentication failed. See table below.                         |
| <p>Smartstore-Api-AuthResultDesc:<br>UserDisabled</p>                        | A short description for the reason why the request was denied. Returned only if the authentication failed. See table below.            |
| <p>WWW-Authenticate:<br>Basic realm="Smartstore.WebApi", charset="UTF-8"</p> | The name of the authentication method that failed.                                                                                     |



## Reasons for denial

| Smartstore-Api-AuthResultId | Smartstore-Api-AuthResultDesc | Remarks                                                                                             |
| :-------------------------: | ----------------------------- | --------------------------------------------------------------------------------------------------- |
|              0              | ApiDisabled                   | The API is disabled.                                                                                |
|              1              | SslRequired                   | HTTPS is required in any case unless the request takes place in a development environment.          |
|              2              | InvalidAuthorizationHeader    | The HTTP authorization header is missing or invalid. Must include a pair of public and secret keys. |
|              2              | InvalidCredentials            | The credentials sent by the HTTP authorization header do not match those of the user.               |
|              4              | UserUnknown                   | The user is unknown.                                                                                |
|              5              | UserDisabled                  | The user is known but his access via the API is disabled.                                           |

