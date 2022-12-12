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
POST http://localhost:59318/odata/v1/Products(1751)/CalculatePriceJsonJso
{ "forListing": false, "quantity": 1 }
```
