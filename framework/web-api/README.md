# Web API

The Smartstore Web API allows direct access to the data of an online store. It is built on the most recent technologies Microsoft offers for web based data consuming, the ASP.NET Core Web API and the OData provider. This documentation gives you an overview of how to build an API client, also known as API consumer. It uses C# code, but you can build your consumer in any common programming language. As OData is a standardized protocol, there are a lot of frameworks and toolkits available for various platforms.

The ASP.NET Core Web API is a framework for building web APIs on top of the ASP.NET Core framework. It uses HTTPS as an application protocol (rather than a transport protocol) to return data based on the client requests. The Open Data Protocol (OData) is a standardized web protocol that provides an uniform way to expose, structure, query and manipulate data using REST practices. OData also provides a uniform way to represent metadata about the data, allowing clients to know more about the type system, relationships and structure of the data.

The Web API is primarily used to exchange raw data. In addition, it contains service functions and methods similar to those in the store backend.

{% hint style="info" %}
When different data is to be exchanged on a large scale or requires special processing, the more flexible solution may be to develop an individual module. Such a module acts as middleware between Smartstore and your application and can in principle access all service functions of the Smartstore core.
{% endhint %}
