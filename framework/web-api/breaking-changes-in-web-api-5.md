# ✔ Breaking changes in Web API 5

#### Authentication

[HMAC authentication](https://en.wikipedia.org/wiki/HMAC) is no longer supported. For the highest level of interoperability with generic clients, the Web API now uses [Basic authentication](https://app.gitbook.com/o/jug3iI9jtm3q3KRxHi73/s/DOZxBBKmB9QIuwBDsOtV/framework/web-api/authentication) over HTTPS, as recommended by the _OData protocol version 4.0_.

#### Querying related entities

The following path is no longer supported:

```
GET /EntitySet({id})/RelatedEntity({relatedId})
```

Simply use the endpoint for the related entity directly.

{% code title="Example" %}
```
old /Customers(1)/Addresses(2)
new /Addresses(2)
```
{% endcode %}

#### Querying properties

The following path is no longer supported:

```
GET /EntitySet({id})/PropertyName
```

Use the more flexible `$select` instead:

{% code title="Example" %}
```
old /Categories(14)/Name
new /Categories(14)?$select=Name
```
{% endcode %}

#### Response of PUT and PATCH requests

For PUT and PATCH requests, the HTTP header **Prefer** with the value `return=representation` must be sent to get a status code `200` with an entity content response. This is the default behavior of _AspNetCore.OData v.8_, otherwise `204 No Content` is returned.

#### Return types of media endpoints

* _/MediaFiles_ returns the type `FileItemInfo` that wraps and enriches the media file entity.
* _/MediaFolders_ returns the type `FolderNodeInfo` that wraps and enriches the media folder entity.

Both are flattened objects without navigation properties.

#### Request parameters

Request parameters must be written in camel case.

{% code title="Example" %}
```
old /MediaFiles/GetFileByPath {"Path": "catalog/my-image.jpg"}
new /MediaFiles/GetFileByPath {"path": "catalog/my-image.jpg"}
```
{% endcode %}

#### Entity fulfillment

The query string parameter **SmNetFulfill** has been renamed to **SmApiFulfill**.

## Changed endpoints

| Old endpoint -> New endpoint                                                                                                                 | Remarks                                                                                                                                                                                                                                                         |
| -------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <p>GET MediaFiles/Download({Id}) -> <br>GET MediaFiles/DownloadFile({id})</p>                                                                |                                                                                                                                                                                                                                                                 |
| <p>POST OrderItems({id})/Infos -> <br>GET OrderItems/GetShipmentInfo({id})</p>                                                               |                                                                                                                                                                                                                                                                 |
| <p>POST Orders({id})/Infos -><br>GET Orders/GetShipmentInfo({id})</p>                                                                        |                                                                                                                                                                                                                                                                 |
| <p>POST Orders({id})/Pdf -><br>GET Orders/DownloadPdf({id})</p>                                                                              |                                                                                                                                                                                                                                                                 |
| <p><mark style="color:orange;">GET Payments/Methods</mark> -><br>GET PaymentMethods/GetAllPaymentMethods({active},{storeId})</p>             | New method. Now returns a list of payment method system names.                                                                                                                                                                                                  |
| <p>ProductPictures/... -><br>ProductMediaFiles/...</p>                                                                                       | The controller name has changed.                                                                                                                                                                                                                                |
| <p>Products/ProductPictures -><br>Products/ProductMediaFiles</p>                                                                             | The navigation property name has changed.                                                                                                                                                                                                                       |
| <p><mark style="color:orange;">POST Uploads/ProductImages</mark> -><br><mark style="color:green;">POST Products/ProductMediaFiles</mark></p> | <p>New method. Now returns a list of <strong>ProductMediaFile</strong>.</p><p>SKU, GTIN or MPN to identify the product can optionally be sent via query string. ContentDisposition parameter <strong>pictureId</strong> renamed to <strong>fileId</strong>.</p> |
| <p>POST Products/FinalPrice|LowestPrice -><br>POST Products/CalculatePrice</p>                                                               | New method. Now returns **CalculatedProductPrice**.                                                                                                                                                                                                             |



{% hint style="info" %}
Notes:

* <mark style="color:orange;">Route</mark> <mark style="color:orange;"></mark><mark style="color:orange;">**/api/v1/**</mark> <mark style="color:orange;"></mark><mark style="color:orange;">no longer exists.</mark>
* <mark style="color:green;">The parameterization has been changed to support</mark> <mark style="color:green;"></mark>_<mark style="color:green;">Swagger</mark>_<mark style="color:green;">.</mark>
{% endhint %}

## Changed response header names

| Old -> new name                                                              | Remarks                                                        |
| ---------------------------------------------------------------------------- | -------------------------------------------------------------- |
| <p>SmartStore-Net-Api-... -><br>Smartstore-Api-...</p>                       | Name prefix changed.                                           |
| <p>SmartStore-Net-Api-HmacResultId -><br>Smartstore-Api-AuthResultId</p>     | [New values](breaking-changes-in-web-api-5.md#authentication). |
| <p>SmartStore-Net-Api-HmacResultDesc -><br>Smartstore-Api-AuthResultDesc</p> | [New values](breaking-changes-in-web-api-5.md#authentication). |
| SmartStore-Net-Api-MissingPermission -> -                                    | Obsolete, no longer sent.                                      |
