# Products

### **Get a product with name "iPhone"**

```
GET http://localhost:59318/odata/v1/Products?$top=1&$filter=Name eq 'iPhone'
```

### **Get child products of grouped product with ID 210**

```
GET http://localhost:59318/odata/v1/Products?$filter=ParentGroupedProductId eq 210
```

### Calculate the final price of a product

```
POST http://localhost:59318/odata/v1/Products(123)/CalculatePrice
{ "forListing": false, "quantity": 1 }
```

{% code title="Response" %}
```json
{
    "@odata.context": "http://localhost:59318/odata/v1/$metadata#Smartstore.Web.Api.Models.Catalog.CalculatedProductPrice",
    "ProductId": 123,
    "CurrencyId": 5,
    "CurrencyCode": "EUR",
    "FinalPrice": 43.07800000,
    "RegularPrice": null,
    "RetailPrice": 47.58810000,
    "OfferPrice": null,
    "ValidUntilUtc": null,
    "PreselectedPrice": null,
    "LowestPrice": null,
    "DiscountAmount": 0,
    "Saving": {
        "HasSaving": true,
        "SavingPrice": 47.58810000,
        "SavingPercent": 9,
        "SavingAmount": 4.51010000
    }
}
```
{% endcode %}

### **Assign category with ID 9 to product with ID 1**

```
POST http://localhost:59318/odata/v1/Products(1)/ProductCategories(9)
{ "DisplayOrder": 5, "IsFeaturedProduct": true }
```

### **Assign manufacturer with ID 12 to product with ID 1**

```
POST http://localhost:59318/odata/v1/Products(1)/ProductManufacturers(12)
{ "DisplayOrder": 1, "IsFeaturedProduct": false }
```

{% hint style="info" %}
* The request body is optional but sending the content type header with _application/json_ is required otherwise 404 _Not Found_ is returned_._
* Use the DELETE method to remove an assignment.
* Omit the category/manufacturer identifier if you want to remove all related assignments for a product.
* It doesn't matter if one of the assignments already exists. The Web API automatically ensures that a product has no duplicate categories or manufacturer assignments.
* Such navigation links are only available for a few navigation properties at the moment.
{% endhint %}

### **Delete assigment of image 66 to product 1**

```
DELETE http://localhost:59318/odata/v1/Products(1)/ProductMediaFiles(66)
```

### **Update display order of image assignment 66 at product 1**

```
PATCH http://localhost:59318/odata/v1/ProductMediaFiles(66)
{ "DisplayOrder": 5 }
```

{% hint style="info" %}
66 is the ID of _ProductMediaFile_, not the ID of _MediaFile_. _ProductMediaFile_ is a mapping between a product and a media file (image).
{% endhint %}

### Upload product images

Multiple images can be uploaded for a product by a single multipart form data POST request. The product ID can be 0 and the product can be identified by query string parameter _sku_, _gtin_ or _mpn_.

```
POST http://localhost:59318/odata/v1/Products(1)/SaveFiles
Content-Type: image/jpeg
Content-Disposition: form-data; name="my-file-1"; filename="my-file1.jpg"

<Binary data for my-file1.jpg here (length 503019 bytes)…>
Content-Type: image/jpeg
Content-Disposition: form-data; name="my-file-2"; filename="my-file2.jpg"

<Binary data for my-file2.jpg here (length 50934 bytes)…>
Content-Type: image/jpeg
Content-Disposition: form-data; name="my-file-3"; filename="my-file3.jpg"

<Binary data for my-file3.jpg here (length 175939 bytes)…>

```

{% hint style="info" %}
It doesn't matter if one of the uploaded images already exists. The Web API automatically ensures that a product has no duplicate images by comparing both binary data streams.
{% endhint %}

It's also possible to update/replace an existing image. To do so simply add the file identifier as `fileId` attribute in the content disposition header of the file. The example updates the picture entity with the Id 6166.

```
POST http://localhost:59318/odata/v1/Products(0)/SaveFiles?sku=p9658742
Content-Type: image/jpeg
Content-Disposition: form-data; name="new-image"; filename="new-image.jpg"; fileId="6166"

<Binary data for new-image.jpg here (length 4108730 bytes)…>
```
