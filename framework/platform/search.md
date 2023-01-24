# 🥚 Search

## Overview

[ICatalogSearchService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Catalog/Search/ICatalogSearchService.cs) enables product searches based on search terms. The interface has two implementations. `CatalogSearchService` searches for products by implementations of [IIndexProvider](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Search/Indexing/IIndexProvider.cs), [IIndexStore](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Search/Indexing/IIndexStore.cs) and [ISearchEngine](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Search/ISearchEngine.cs). An example of such an extension is the Smartstore _MegaSearch_ module, which connects the search library Lucene.Net to Smartstore. Without such an extension, `LinqCatalogSearchService` is used, which searches the database directly for hits using LINQ.

The `CatalogSearchingEvent` is published right before a catalog search regardless of whether the search is performed with `CatalogSearchService` or `LinqCatalogSearchService`. Accordingly, `CatalogSearchedEvent` is published at the end of a search.

## Search query

`CatalogSearchQueryFactory` reads query string parameters and creates a `CatalogSearchQuery` from it, which contains all the information needed for the search engine to perform the search. Parameters are typically used to filter the search result. Possible parameters:

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

HINT: parameter tokens can be overwritten by the user in the backend via alias fields (see [ISearchAlias](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Search/ISearchAlias.cs)). For example if the manufacturer's name should appear in the URL instead of the token _m_.

`ICatalogSearchService.PrepareQuery` lets you build or modify your own catalog search query using LINQ.

## Filter

Filters are needed to limit search results, e.g. to display only products of a certain category. They are determined by `CatalogSearchQueryFactory` via the query string and passed on to the search with the help of `CatalogSearchQuery`. If you want to search programmatically, you can create a `CatalogSearchQuery` instance and define it yourself using fluent notation:

```csharp
var searchQuery = new CatalogSearchQuery()
    .VisibleOnly()
    .WithVisibility(ProductVisibility.Full)
    .HasStoreId(Services.StoreContext.CurrentStoreIdIfMultiStoreMode)
    .WithCategoryIds(null, categoryIds.ToArray())
    .BuildFacetMap(false)
    .BuildHits(false);
    
var searchResult = await _catalogSearchService.SearchAsync(searchQuery);
```

## Indexing

Search libraries like Lucene.Net determine search hits via the file system instead of accessing databases directly. These files are called index or search index. Smartstore provides a whole range of interfaces for creating and managing search indexes and a _MegaSearch_ module which performs the actual indexing. Essentially, the process is as follows.

An [IIndexingService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Search/Indexing/IIndexingService.cs) implementation creates or updates the search index, always triggered by its own task. The index scope, that is the information about what kind of index it is, is obtained via the [IIndexScopeManager](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Search/Indexing/IIndexScopeManager.cs). The indexing service uses a data collector (see [IIndexCollector](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Search/Indexing/IIndexCollector.cs)) to collect all the data to be transferred to the index. The index service does not access the index directly, but uses the [IIndexStore](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Search/Indexing/IIndexStore.cs) for this purpose. It ensures that the index can be accessed for writing (during indexing) and reading (during searching).

During an index update, the records to be updated are determined with the help of [IIndexBacklogService](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Search/Indexing/Backlog/IIndexBacklogService.cs) and [IndexBacklogItem](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Search/Indexing/Backlog/IndexBacklogItem.cs). Backlog items are previously determined and saved to the database via a hook, e.g. when a product has been modified.

The collector fires an [IndexSegmentProcessedEvent](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Search/Indexing/Events/IndexSegmentProcessedEvent.cs) each time after it has processed a segment of entities. The indexing service fires an [IndexingCompletedEvent](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/Search/Indexing/Events/IndexingCompletedEvent.cs) at the end of indexing.

## Implementing a custom search

The built-in catalog search searches for products. To implement another search for a different entity, the principle of the catalogue search must be copied and adapted accordingly.\
TODO: roughly describe what we did in the forum search...
