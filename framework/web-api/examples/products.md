# Products

### **Getting a product with name "iPhone"**

```
GET http://localhost:59318/odata/v1/Products?$top=1&$filter=Name eq 'iPhone'
```

### **Getting child products of grouped product with ID 210**

```
GET http://localhost:59318/odata/v1/Products?$filter=ParentGroupedProductId eq 210
```

### Calculating the final price of a product

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
