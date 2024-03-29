# ✔ Web API in detail

You can consume API services in a RESTful manner via HTTPS calls by using HTTPS methods:

* `GET`: Get the resource.
* `POST`: Create a new resource.
* `PUT`: Update the existing resource.
* `PATCH`: Update a part of the existing resource.
* `DELETE`: Remove the resource.

OData query options (such a `$expand`, `$filter`, `$top`, `$select`, etc.) and API specific options (such as `SmApiFulfillCountry`) can be passed via query strings.

Paging is required if you want to query multiple records. You can do this with the OData query options `$skip` and `$top`. The maximum value for `$top` is returned in the `Smartstore-Api-MaxTop` header field. This value is configurable by the store owner.

Due to [Basic Authentication](https://app.gitbook.com/o/jug3iI9jtm3q3KRxHi73/s/DOZxBBKmB9QIuwBDsOtV/framework/web-api/authentication), it is mandatory to send requests via HTTPS. HTTP will return a `421 Misdirected Request` status code. The only exception is that developers can send requests via HTTP to their local development environment.

A request body must be _UTF-8_ encoded.

## Request HTTP header fields

| Name: value                                                     | Required |                                                                           Remarks                                                                           |
| --------------------------------------------------------------- | -------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------: |
| <p><strong>Accept</strong>:<br>application/json</p>             | yes      |                                                              Only _application/json_ is valid.                                                              |
| <p><strong>Accept-Charset</strong>:<br>UTF-8</p>                | yes      |                                                                                                                                                             |
| <p><strong>Authorization</strong>:<br>Basic &#x3C;key pair></p> | yes      |                                                           See [Authentication](authentication.md).                                                          |
| <p><strong>Prefer</strong>:<br>return=representation</p>        | no       | Can be sent for PUT and PATCH requests if the API should answer with status code `200` and entity content response, otherwise `204 No Content` is returned. |

## Response HTTP header fields

| Name: example value                                                                           | Description                                                                                                                                                                |
| --------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| <p><strong>Smartstore-Api-AppVersion</strong>:<br>5.1.0.0</p>                                 | Smartstore version used by the store.                                                                                                                                      |
| <p><strong>Smartstore-Api-Version</strong>:<br>1 5.0</p>                                      | The highest API version supported by the server (unsigned integer) and the version of the installed API module (floating-point value).                                     |
| <p><strong>Smartstore-Api-MaxTop</strong>:<br>120</p>                                         | The maximum value for OData `$top` query option. The default value is `120` and is configurable by store owner.                                                            |
| <p><strong>Smartstore-Api-Date</strong>:<br>2022-11-11T14:35:33.7772907Z</p>                  | The current server date and time in _ISO-8601 UTC_.                                                                                                                        |
| <p><strong>Smartstore-Api-CustomerId</strong>:<br>1234</p>                                    | The customer identifier of the authenticated API user. Returned only if authentication is successful.                                                                      |
| <p><strong>Smartstore-Api-AuthResultId</strong>:<br>5</p>                                     | <p>The ID of the reason why the request was denied. Returned only if the authentication failed.</p><p>See <a href="authentication.md">authentication</a>.</p>              |
| <p><strong>Smartstore-Api-AuthResultDesc</strong>:<br>UserDisabled</p>                        | <p>A short description of the reason why the request was denied. Returned only if the authentication failed.</p><p>See <a href="authentication.md">authentication</a>.</p> |
| <p><strong>WWW-Authenticate</strong>:<br>Basic realm="Smartstore.WebApi", charset="UTF-8"</p> | The name of the failed authentication method.                                                                                                                              |

## Query options

[OData query options](https://learn.microsoft.com/en-us/odata/concepts/queryoptions-overview) allow manipulation of data queries like sorting, filtering, paging etc. They are sent in the query string of the request URL.

Custom query options should make the Smartstore Web API easier to use, especially when working with entity relationships.

#### SmApiFulfill{property\_name}

Entities often have multiple relationships with other entities. In most cases, an ID must be set to create or modify them. But often you do not know the ID. To reduce the number of API round trips, this option can set an entity relationship indirectly.

For example, imagine you want to add a German address, but you don't have the ID of the German country entity, required for address insertion. Instead of calling the API again to get the ID, you can add the query option `SmApiFulfillCountry=DE` and the API will automatically resolve the relationship.

The API can fulfill the following properties:

| Entity property   | Expected query value                      | Query string example             |
| ----------------- | ----------------------------------------- | -------------------------------- |
| **Country**       | The two or three letter ISO country code. | SmApiFulfill**Country**=USA      |
| **StateProvince** | The abbreviation of a state or province.  | SmApiFulfill**StateProvince**=CA |
| **Language**      | The culture of a language.                | SmApiFulfill**Language**=de-DE   |
| **Currency**      | The ISO code of a currency.               | SmApiFulfill**Currency**=EUR     |

{% hint style="warning" %}
`SmApiFulfill` sets the relationship only if none exists yet. An existing relationship cannot be modified with this option.
{% endhint %}

## Web API and modules

If a module extends the domain model with its own entities, these can also be integrated into the Web API using [ODataModelProviderBase](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Web.Common/Api/ODataModelProviderBase.cs) or [IODataModelProvider](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Web.Common/Api/IODataModelProvider.cs). This allows the module developer to make their entities externally accessible without having to create their own Web API.

The _Smartstore.Blog_ module registers two entities `BlogComment` and `BlogPost` through the `ODataModelBuilder` using the following `BlogODataModelProvider`.

```csharp
internal class BlogODataModelProvider : ODataModelProviderBase
{
    public override void Build(ODataModelBuilder builder, int version)
    {
        builder.EntitySet<BlogComment>("BlogComments");
        builder.EntitySet<BlogPost>("BlogPosts");
    }

    public override Stream GetXmlCommentsStream(IApplicationContext appContext)
        => GetModuleXmlCommentsStream(appContext, "Smartstore.Blog");
}
```

{% hint style="info" %}
Use plural for the name of the entity set. For example, _BlogComments_, not _BlogComment_.
{% endhint %}

Next, you need to add an API controller for each entity. It is recommended that you create a subfolder called _Api_ in the controller folder of the module. Inherit your controller from [WebApiController](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Web.Common/Api/WebApiController.cs), as shown in the example.

```csharp
/// <summary>
/// The endpoint for operations on the BlogComment entity.
/// </summary>
public class BlogCommentsController : WebApiController<BlogComment>
{
    [HttpGet("BlogComments"), ApiQueryable]
    [Permission(BlogPermissions.Read)]
    public IQueryable<BlogComment> Get()
    {
        var query = Db.CustomerContent
            .AsNoTracking()
            .OrderByDescending(x => x.CreatedOnUtc)
            .OfType<BlogComment>();

        return query;
    }

    [HttpGet("BlogComments({key})"), ApiQueryable]
    [Permission(BlogPermissions.Read)]
    public SingleResult<BlogComment> Get(int key)
    {
        return GetById(key);
    }

    [HttpGet("BlogComments({key})/BlogPost"), ApiQueryable]
    [Permission(BlogPermissions.Read)]
    public SingleResult<BlogPost> GetBlogPost(int key)
    {
        return GetRelatedEntity(key, x => x.BlogPost);
    }

    [HttpPost]
    [Permission(BlogPermissions.Create)]
    public async Task<IActionResult> Post([FromBody] BlogComment entity)
    {
        return await PostAsync(entity);
    }

    [HttpPut]
    [Permission(BlogPermissions.Update)]
    public async Task<IActionResult> Put(int key, Delta<BlogComment> model)
    {
        return await PutAsync(key, model);
    }

    [HttpPatch]
    [Permission(BlogPermissions.Update)]
    public async Task<IActionResult> Patch(int key, Delta<BlogComment> model)
    {
        return await PatchAsync(key, model);
    }

    [HttpDelete]
    [Permission(BlogPermissions.Delete)]
    public async Task<IActionResult> Delete(int key)
    {
        return await DeleteAsync(key);
    }
}
```

{% hint style="info" %}
Use the`Smartstore.Web.Api.Controllers` namespace for all Web API controllers!
{% endhint %}

The `BlogCommentsController` defines OData Web API endpoints to get, create, update, patch (partially update), and delete blog comments. It also defines an endpoint to get the associated blog post through the `BlogPost` navigation property. See the Web API module [controllers](https://github.com/smartstore/Smartstore/tree/main/src/Smartstore.Modules/Smartstore.WebApi/Controllers) for more endpoint definitions.

Override `ODataModelProviderBase.GetXmlCommentsStream` to automatically include your code comments in the Swagger documentation of the Web API. To do this, the **Documentation File** option must also be enabled in the module project settings. The path to the **XML documentation file** should be left empty. This will export your code comments to an XML file with the same name as the module at compile time (_Smartstore.Blog.xml_ in the example above), which will then be used by the Web API.
