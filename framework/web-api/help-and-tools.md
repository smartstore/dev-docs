# Help & Tools

## The OData metadata document <a href="#helpandtools-theodatametadatadocument" id="helpandtools-theodatametadatadocument"></a>

The metadata document describes the entity data model (EDM) of the OData service, using an XML language called the Conceptual Schema Definition Language (CSDL). The metadata document shows the structure of the data in the OData service and can be used to generate client code. This is the recommended overview for the consumer to indicate the location of a particular resource or API endpoint. To get the metadata document, send the following request:

```http
GET http://localhost:1260/odata/v1/$metadata
```

## Swagger Web API help <a href="#helpandtools-swaggerwebapihelp" id="helpandtools-swaggerwebapihelp"></a>

Swagger is a machine readable representation of a RESTful API that enables support for interactive documentation, client SDK generation and discoverability. It is capable to send test requests to the API through a **Try it out** button. To open the Swagger help pages, send the following request:

```
GET http://localhost:1260/swagger/ui/index
```

## Client test tools <a href="#helpandtools-clienttesttools" id="helpandtools-clienttesttools"></a>

The Smartstore source code includes two client tools for testing the API: a Windows Forms application and a pure JavaScript client. Both are included in the source code package that can be downloaded at the [Smartstore Releases page](https://github.com/smartstoreag/SmartStoreNET/releases). The source code of both clients is open. It can be found on [GitHub](https://github.com/smartstoreag/SmartStoreNET) under **src/Tools**.

{% hint style="info" %}
You won't find help to each API resource in this documentation. Instead use the above tools (or even other) to explore endpoints, fields, field types etc. Even if this documentation contains several examples of frequently data exchange scenarios, it cannot make up for a detailed OData documentation.
{% endhint %}
