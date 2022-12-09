# Breaking changes in Web API 5

* HMAC authentication is no longer supported. For the highest level of interoperability with generic clients, the Web API now uses [Basic authentication](authentication.md) over HTTPS as recommended by OData protocol version 4.0.
* Querying a related entity via path GET `/EntitySet({id})/RelatedEntity({relatedId})` is no longer supported. Use the related path directly.\
  Example: old `/Customers(1)/Addresses(2)`, new `/Addresses(2)`.
* Querying a single, simple property value via path GET `/EntitySet({id})/PropertyName` is no longer supported. Use the more flexible _$select_ instead.\
  Example: old `/Categories(14)/Name`, new `/Categories(14)?$select=Name`.
* For PUT and PATCH requests, the HTTP header _Prefer_ with the value `return=representation` must be sent to get a status code 200 with entity content response. This is the default behavior of AspNetCore.OData v.8. Otherwise 204 _No Content_ is returned.
* `/MediaFiles` returns type `FileItemInfo` which wraps and enriches the media file entity. `/MediaFolders` returns type `FolderNodeInfo` which wraps and enriches the media folder entity. Both are flattened objects and no longer entities.
* Request parameters are always written in camel case, for example for OData actions. Example:\
  old `/MediaFiles/GetFileByPath {"Path":"catalog/my-image.jpg"}`\
  new `/MediaFiles/GetFileByPath {"path":"catalog/my-image.jpg"}`.

## Changed endpoints

| Old endpoint                  | New endpoint                                                | Remarks                                                        |
| ----------------------------- | ----------------------------------------------------------- | -------------------------------------------------------------- |
| GET MediaFiles/Download({Id}) | GET MediaFiles/DownloadFile({id})                           |                                                                |
| POST OrderItems({id})/Infos   | GET OrderItems/GetShipmentInfo({id})                        |                                                                |
| POST Orders({id})/Infos       | GET Orders/GetShipmentInfo({id})                            |                                                                |
| POST Orders({id})/Pdf         | GET Orders/DownloadPdf({id})                                |                                                                |
| GET Payments/Methods 1.       | GET PaymentMethods/GetAllPaymentMethods({active},{storeId}) | New method. Now returns a list of payment method system names. |
|                               |                                                             |                                                                |

