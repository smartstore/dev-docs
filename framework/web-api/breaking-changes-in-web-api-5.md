# Breaking changes in Web API 5

#### Authentication

HMAC authentication is no longer supported. For the highest level of interoperability with generic clients, the Web API now uses [Basic authentication](authentication.md) over HTTPS as recommended by OData protocol version 4.0.

#### Querying related entities

The following path is no longer supported

```
GET /EntitySet({id})/RelatedEntity({relatedId})
```

Use the entpoint for the related entity directly.

{% code title="Example" %}
```
old /Customers(1)/Addresses(2)
new /Addresses(2)
```
{% endcode %}

#### Querying properties

The following path is no longer supported

```
GET /EntitySet({id})/PropertyName
```

Use the more flexible _$select_ instead.

{% code title="Example" %}
```
old /Categories(14)/Name
new /Categories(14)?$select=Name
```
{% endcode %}

#### Response of PUT and PATCH requests

For PUT and PATCH requests, the HTTP header **Prefer** with the value **return=representation** must be sent to get a status code 200 with entity content response. This is the default behavior of AspNetCore.OData v.8. Otherwise 204 _No Content_ is returned.

#### Return types of media endpoints

`/MediaFiles` returns type **FileItemInfo** which wraps and enriches the media file entity. `/MediaFolders` returns type **FolderNodeInfo** which wraps and enriches the media folder entity. Both are flattened objects and no longer entities.

#### Request parameters must be written in Camel case

{% code title="Example" %}
```
old /MediaFiles/GetFileByPath {"Path": "catalog/my-image.jpg"}
new /MediaFiles/GetFileByPath {"path": "catalog/my-image.jpg"}
```
{% endcode %}

#### Entity fulfillment

The query string parameter **SmNetFulfill** has been renamed to **SmApiFulfill**.

## Changed endpoints

| Old -> new endpoint                                                                              | Remarks                                                                                                                                                                                                   |
| ------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <p>GET MediaFiles/Download({Id}) -><br>GET MediaFiles/DownloadFile({id})</p>                     |                                                                                                                                                                                                           |
| <p>POST OrderItems({id})/Infos -><br>GET OrderItems/GetShipmentInfo({id})</p>                    |                                                                                                                                                                                                           |
| <p>POST Orders({id})/Infos -><br>GET Orders/GetShipmentInfo({id})</p>                            |                                                                                                                                                                                                           |
| <p>POST Orders({id})/Pdf -><br>GET Orders/DownloadPdf({id})</p>                                  |                                                                                                                                                                                                           |
| <p>GET Payments/Methods 1. -><br>GET PaymentMethods/GetAllPaymentMethods({active},{storeId})</p> | New method. Now returns a list of payment method system names.                                                                                                                                            |
| <p>ProductPictures/... -><br>ProductMediaFiles/...</p>                                           | The controller name has changed.                                                                                                                                                                          |
| <p>Products/ProductPictures --><br>Products/ProductMediaFiles</p>                                | The navigation property name has changed.                                                                                                                                                                 |
| <p>POST Uploads/ProductImages 1. --><br>POST Products/ProductMediaFiles 2.</p>                   | New method. Now returns list of **ProductMediaFile**. SKU, GTIN or MPN to identify the product can optionally be sent via query string. ContentDisposition parameter **pictureId** renamed to **fileId**. |

1. Route **/api/v1/** no longer exists.
2. The parameterization has been changed to support Swagger.

## Changed response header names

| Old -> new name                                                              | Remarks                                                |
| ---------------------------------------------------------------------------- | ------------------------------------------------------ |
| <p>SmartStore-Net-Api-... -><br>Smartstore-Api-...</p>                       | Name prefix changed.                                   |
| <p>SmartStore-Net-Api-HmacResultId -><br>Smartstore-Api-AuthResultId</p>     | [New values](web-api-in-detail.md#reasons-for-denial). |
| <p>SmartStore-Net-Api-HmacResultDesc -><br>Smartstore-Api-AuthResultDesc</p> | [New values](web-api-in-detail.md#reasons-for-denial). |
| <p>SmartStore-Net-Api-MissingPermission -><br>-</p>                          | Obsolete, no longer sent.                              |
