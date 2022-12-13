# Orders

### **Mark order with ID 145 as paid**

```
POST http://localhost:59318/odata/v1/Orders(145)/PaymentPaid
{ "paymentMethodName": "Payments.Sofortueberweisung" }
```

The example also sets the system name of the payment method to `Payments.Sofortueberweisung`for the order.

### Refund order with ID 146

```
POST http://localhost:59318/odata/v1/Orders(146)/PaymentRefund
{ "online": true }
```

The parameter **online** indicates whether to call the related payment gateway to refund the payment. `True` would refund against the payment gateway. `False` just sets the status offline without calling any payment gateway.

### Complete order with ID 147

```
POST http://localhost:59318/odata/v1/Orders(147)/CompleteOrder
```

### Download order with ID 150 as PDF

```
GET http://localhost:59318/odata/v1/Orders/DownloadPdf(id=150)
```
