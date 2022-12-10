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

| Old endpoint                               | New endpoint                                                | Remarks                                                                                                                                                                                                   |
| ------------------------------------------ | ----------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| GET MediaFiles/Download({Id})              | GET MediaFiles/DownloadFile({id})                           |                                                                                                                                                                                                           |
| POST OrderItems({id})/Infos                | GET OrderItems/GetShipmentInfo({id})                        |                                                                                                                                                                                                           |
| POST Orders({id})/Infos                    | GET Orders/GetShipmentInfo({id})                            |                                                                                                                                                                                                           |
| POST Orders({id})/Pdf                      | GET Orders/DownloadPdf({id})                                |                                                                                                                                                                                                           |
| <p>GET Payments/Methods<br>1.</p>          | GET PaymentMethods/GetAllPaymentMethods({active},{storeId}) | New method. Now returns a list of payment method system names.                                                                                                                                            |
| ProductPictures/...                        | ProductMediaFiles/...                                       | The controller name has changed.                                                                                                                                                                          |
| Products/ProductPictures                   | Products/ProductMediaFiles                                  | The navigation property name has changed.                                                                                                                                                                 |
| <p>POST Uploads/ProductImages<br>1. 2.</p> | POST Products({id})/SaveFiles                               | New method. Now returns list of **ProductMediaFile**. SKU, GTIN or MPN to identify the product can optionally be sent via query string. ContentDisposition parameter **pictureId** renamed to **fileId**. |

1. Route **/api/v1/** no longer exists.
2. The parameterization has been changed to support Swagger.

## Changed response header names

| Old name                             | New name                      | Remarks                                                |
| ------------------------------------ | ----------------------------- | ------------------------------------------------------ |
| SmartStore-Net-Api-...               | Smartstore-Api-...            | Name prefix changed.                                   |
| SmartStore-Net-Api-HmacResultId      | Smartstore-Api-AuthResultId   | [New values](web-api-in-detail.md#reasons-for-denial). |
| SmartStore-Net-Api-HmacResultDesc    | Smartstore-Api-AuthResultDesc | [New values](web-api-in-detail.md#reasons-for-denial). |
| SmartStore-Net-Api-MissingPermission | -                             | Obsolete, no longer sent.                              |
