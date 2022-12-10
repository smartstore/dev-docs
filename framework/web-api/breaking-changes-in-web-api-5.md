# Breaking changes in Web API 5

* HMAC authentication is no longer supported. For the highest level of interoperability with generic clients, the Web API now uses [Basic authentication](authentication.md) over HTTPS as recommended by OData protocol version 4.0.
* Querying a related entity via path `GET/EntitySet({id})/RelatedEntity({relatedId})` is no longer supported. Use the related path directly.\
  Example (old\new): `/Customers(1)/Addresses(2)` --> `/Addresses(2)`
* Querying a single, simple property value via path `GET /EntitySet({id})/PropertyName` is no longer supported. Use the more flexible _$select_ instead.\
  Example (old\new): `/Categories(14)/Name` --> `/Categories(14)?$select=Name`
* For PUT and PATCH requests, the HTTP header **Prefer** with the value **return=representation** must be sent to get a status code 200 with entity content response. This is the default behavior of AspNetCore.OData v.8. Otherwise 204 _No Content_ is returned.
* `/MediaFiles` returns type **FileItemInfo** which wraps and enriches the media file entity. `/MediaFolders` returns type **FolderNodeInfo** which wraps and enriches the media folder entity. Both are flattened objects and no longer entities.
* Request parameters are always written in camel case, for example for OData actions.\
  Example (old\new): `/MediaFiles/GetFileByPath {"Path":"catalog/my-image.jpg"}`\
  \--> `/MediaFiles/GetFileByPath {"path":"catalog/my-image.jpg"}`.
* The query string parameter **SmNetFulfill** has been renamed to **SmApiFulfill**.

## Changed endpoints

| Old --> new endpoint                                                                              | Remarks                                                                                                                                                                                                   |
| ------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <p>GET MediaFiles/Download({Id}) --><br>GET MediaFiles/DownloadFile({id})</p>                     |                                                                                                                                                                                                           |
| <p>POST OrderItems({id})/Infos --><br>GET OrderItems/GetShipmentInfo({id})</p>                    |                                                                                                                                                                                                           |
| <p>POST Orders({id})/Infos --><br>GET Orders/GetShipmentInfo({id})</p>                            |                                                                                                                                                                                                           |
| <p>POST Orders({id})/Pdf --><br>GET Orders/DownloadPdf({id})</p>                                  |                                                                                                                                                                                                           |
| <p>GET Payments/Methods 1. --><br>GET PaymentMethods/GetAllPaymentMethods({active},{storeId})</p> | New method. Now returns a list of payment method system names.                                                                                                                                            |
| <p>ProductPictures/... --><br>ProductMediaFiles/...</p>                                           | The controller name has changed.                                                                                                                                                                          |
| <p>Products/ProductPictures --><br>Products/ProductMediaFiles</p>                                 | The navigation property name has changed.                                                                                                                                                                 |
| <p>POST Uploads/ProductImages 1. --><br>POST Products/ProductMediaFiles 2.</p>                    | New method. Now returns list of **ProductMediaFile**. SKU, GTIN or MPN to identify the product can optionally be sent via query string. ContentDisposition parameter **pictureId** renamed to **fileId**. |

1. Route **/api/v1/** no longer exists.
2. The parameterization has been changed to support Swagger.

## Changed response header names

| Old --> new name                                                              | Remarks                                                |
| ----------------------------------------------------------------------------- | ------------------------------------------------------ |
| <p>SmartStore-Net-Api-... --><br>Smartstore-Api-...</p>                       | Name prefix changed.                                   |
| <p>SmartStore-Net-Api-HmacResultId --><br>Smartstore-Api-AuthResultId</p>     | [New values](web-api-in-detail.md#reasons-for-denial). |
| <p>SmartStore-Net-Api-HmacResultDesc --><br>Smartstore-Api-AuthResultDesc</p> | [New values](web-api-in-detail.md#reasons-for-denial). |
| <p>SmartStore-Net-Api-MissingPermission --><br>-</p>                          | Obsolete, no longer sent.                              |
