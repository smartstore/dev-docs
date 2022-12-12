# Examples

{% code title="Partially update address with ID 1" %}
```
PATCH http://localhost:59318/odata/v1/Addresses(1)?SmApiFulfillCountry=US&SmApiFulfillStateProvince=NY
{"City":"New York","Address1":"21 West 52nd Street","ZipPostalCode":"10021","FirstName":"John","LastName":"Doe"}
```
{% endcode %}
