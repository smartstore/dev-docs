# Examples

{% code title="Partially update an address" %}
```json
PATCH http://localhost:59318/odata/v1/Addresses(1)?SmApiFulfillCountry=US&SmApiFulfillStateProvince=NY
{"City":"New York","Address1":"21 West 52nd Street","ZipPostalCode":"10021","FirstName":"John","LastName":"Doe"}
```
{% endcode %}

The example uses the `SmApiFulfillCountry` and `SmApiFulfillStateProvince` options to update the country (USA) and province (New York). This avoids extra querying of country and province records and passing its IDs in the request body.

{% code title="Get ID of store with name "my nice store"" %}
```
GET http://localhost:59318/odata/v1/Stores?$top=1&$filter=Name eq 'my nice store'&$select=Id
```
{% endcode %}

