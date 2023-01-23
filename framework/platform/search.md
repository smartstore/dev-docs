# 🥚 Search

## Overview

[ICatalogSearchService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Catalog/Search/ICatalogSearchService.cs) enables product searches based on search terms. The interface has two implementations. `CatalogSearchService` searches for products by implementations of [IIndexProvider](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Search/Indexing/IIndexProvider.cs), [IIndexStore](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Search/Indexing/IIndexStore.cs) and [ISearchEngine](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Search/ISearchEngine.cs). An example of such an extension is the Smartstore _MegaSearch_ module, which connects the search library Lucene.Net to Smartstore. Without such an extension, `LinqCatalogSearchService` is used, which searches the database directly for hits using LINQ.

The `CatalogSearchingEvent` is published right before a catalog search regardless of whether the search is performed with `CatalogSearchService` or `LinqCatalogSearchService`. Accordingly, `CatalogSearchedEvent` is published at the end of a search.

## Search query

`CatalogSearchQueryFactory` reads query string parameters and creates a `CatalogSearchQuery` from it. Parameters are typically used to filter the search result. Possible parameters:

| Query token | Type    | Description                                                                                                    |
| ----------: | ------- | -------------------------------------------------------------------------------------------------------------- |
|       **q** | string  | Search **query**/term.                                                                                         |
|       **i** | int     | Specifies the **page index** (starting from 1).                                                                |
|       **s** | int     | Specifies the **page size** of search hits.                                                                    |
|       **o** | int     | Specifies how to **order** the search hits. For supported values see `ProductSortingEnum`.                     |
|       **p** | decimal | Filters products by a **price range**. Supported formats: _from\~to_, _from(\~)_ or _\~to_.                    |
|       **c** | int     | Filters products by assigned **categories**. Supports a comma separated list of category identifiers.          |
|       **m** | int     | Filters products by assigned **manufacturers**. Supports a comma separated list of manufacturer identifiers.   |
|       **r** | double  | Filters products by their minimum **rating**. Supports values from 0 to 5.                                     |
|       **a** | bool    | Filters products by their **stock level**.                                                                     |
|       **n** | bool    | Filters for **newly arrived products**.                                                                        |
|       **d** | in      | Filters products by assigned **delivery times**. Supports a comma separated list of delivery time identifiers. |
|       **v** | string  | Specifies the **view mode** for search hits. Supported value are _grid_ or _list_.                             |

HINT: more parameters available for filtering products by variants and attributes if the _MegaSearchPlus_ module is installed. They are prefixed by _attr_ (product attribute), _vari_ (product variant) or _opt_ (option value of attribute or variant).

HINT: parameter tokens can be overwritten by the user in the backend via alias fields. For example if the manufacturer's name should appear in the URL instead of the token _m_.

`ICatalogSearchService.PrepareQuery` lets you build or modify your own catalog search query using LINQ.

## Implementing a custom search

The built-in catalog search searches for products. To implement another search for a different entity, the principle of the catalogue search must be copied and adapted accordingly.\
TODO: roughly describe what we did in the forum search...
