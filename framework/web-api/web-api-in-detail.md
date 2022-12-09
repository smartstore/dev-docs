# Web API in detail

You can consume API services through HTTPS calls in a RESTful manner by using HTTPS methods GET, POST (insert), PUT (update), PATCH (partially update) and DELETE. OData query options (such a $expand, $filter, $top, $select, etc.) and API specific options (such as `SmApiFulfillCountry`) can be transferred through query strings.

Paging is required if you want to query multiple records. You can do that with OData query options $skip and $top. The maximum value for $top is returned in the `Smartstore-Api-MaxTop` header field. The value is configurable by storekeeper.

Due to [Basic Authentication](authentication.md), it is mandatory to send requests over HTTPS. For HTTP, a status code 421 _Misdirected Request_ is returned. The only exception is that developers can send requests over HTTP to their local development environment.

A request body needs to be UTF-8 encoded.

## Special request HTTP header fields

<table><thead><tr><th>Key</th><th>Value</th><th align="center">Required</th><th>Remarks</th><th data-hidden></th></tr></thead><tbody><tr><td>Accept</td><td>application/json</td><td align="center">yes</td><td>Only <em>application/json</em> is valid.</td><td></td></tr><tr><td>Accept-Charset</td><td>UTF-8</td><td align="center">yes</td><td></td><td></td></tr><tr><td>Authorization</td><td>Basic &#x3C;key pair></td><td align="center">yes</td><td>See <a href="authentication.md">Authentication</a>.</td><td></td></tr><tr><td>Prefer</td><td>return=representation</td><td align="center">no</td><td>Can be sent for PUT and PATCH requests if the API should answer with status code 200 and entity content response. Otherwise 204 <em>No Content</em> is returned.</td><td></td></tr></tbody></table>
