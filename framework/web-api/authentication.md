# ✔ Authentication

Smartstore Web API uses Basic authentication over HTTPS authentication method to protect data from unauthorized access. It is recommended by OData protocol version 4.0 for the highest level of interoperability with generic clients.

The client sends the credentials by using the Authorization header. The credentials are formatted as the string `public key:secret key` using UTF-8 and base64 encoding. The credentials are not encrypted, so HTTPS is mandatory.

{% code title="Example" %}
```csharp
var credentials = Convert.ToBase64String(
    Encoding.UTF8.GetBytes($"{publicKey}:{secretKey}"));

using var message = new HttpRequestMessage(
    new HttpMethod("GET"),
    "http://localhost:59318/odata/v1/Customers");

message.Headers.Authorization = new AuthenticationHeaderValue("Basic", credentials);
// Authorization: Basic ZWE2NGQ0YTIyZGI1......ZDY4NGRlMDRmZEFiZGUwMmY3MTg=
```
{% endcode %}

The API responds with the status code 401 _Unauthorized_, if the user is not authorized to exchange data via the API. In this case, **Smartstore-Api-AuthResultId** (ID of the denied reason) and **Smartstore-Api-AuthResultDesc** (short description of the denied reason) response HTTP headers are sent with details of the reason for denial. In addition, the header **WWW-Authenticate** is sent with the value `Basic realm="Smartstore.WebApi", charset="UTF-8"`.

## Reasons for denial

| AuthResultId | AuthResultDesc             | Description                                                                                         |
| :----------: | -------------------------- | --------------------------------------------------------------------------------------------------- |
|       0      | ApiDisabled                | The API is disabled.                                                                                |
|       1      | SslRequired                | HTTPS is required in any case unless the request takes place in a development environment.          |
|       2      | InvalidAuthorizationHeader | The HTTP authorization header is missing or invalid. Must include a pair of public and secret keys. |
|       3      | InvalidCredentials         | The credentials sent by the HTTP authorization header do not match those of the user.               |
|       4      | UserUnknown                | The user is unknown.                                                                                |
|       5      | UserDisabled               | The user is known but his access via the API is disabled.                                           |
