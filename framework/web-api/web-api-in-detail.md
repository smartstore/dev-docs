# Web API in detail

You can consume API services through HTTPS calls in a RESTful manner by using HTTPS methods GET, POST (insert), PUT (update), PATCH (partially update) and DELETE. OData query options (such a $expand, $filter, $top, $select, etc.) and API specific options (such as SmApiFulfillCountry) can be transferred through query strings.

Paging is required if you want to query multiple records. You can do that with OData query options $skip and $top. The maximum value for $top is returned in the `Smartstore-Api-MaxTop` header field. The value is configurable by storekeeper.

Due to [Basic Authentication](authentication.md), it is mandatory to send requests over HTTPS. For HTTP, a status code 421 **Misdirected Request** is returned. The only exception is that developers can send requests over HTTP to their local development environment.

A request body needs to be UTF-8 encoded.
