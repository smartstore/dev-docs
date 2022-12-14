# Customers

### **Get return requests for a customer**

```
GET http://localhost:59318/odata/v1/Customers(1)/ReturnRequests
```

### **Get email address of a customer**&#x20;

```
GET http://localhost:59318/odata/v1/Customers(1)?$select=Email
```

### Assign billing and shipping address

```
PATCH http://localhost:59318/odata/v1/Customers(1)
{ "BillingAddressId": 86384, "ShippingAddressId": 86384 }
```

### **Assign address with ID 10 to customer with ID 1**

```
POST http://localhost:59318/odata/v1/Customers(1)/Addresses(10)
```

Use the DELETE method to remove an assignment. Use 0 as address identifier if you want to remove all address assignments for a customer.

{% hint style="info" %}
It doesn't matter if one of the assignments already exists. The Web API automatically ensures that a customer has no duplicate address assignments.
{% endhint %}
