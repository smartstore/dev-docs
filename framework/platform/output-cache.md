---
description: Improve performance and scalability
---

# 🥚 Output Cache

## Concept

Output caching is a technique used to improve the performance and scalability of Smartstore by storing the generated output of pages on the server side in memory, relational database or in a REDIS database. When a client requests a store page, the server first checks if a cached copy of the page exists. If it does, the server can serve the cached copy to the client without generating the page again, reducing the processing and network overhead.

Smartstore also ensures that cached content is invalidated or refreshed when it becomes stale or outdated, and that private or sensitive data is not inadvertently cached. Proper caching can significantly reduce the response time and resource usage of a store, improving the user experience and reducing server load.

### "Donut Hole" Caching

Smartstore follows the so-called _Donut Hole Caching_ strategy. In this strategy, a dynamic web page is divided into two parts: a non-dynamic or semi-static outer layer, and a dynamic inner layer. The outer layer is cached, while the inner layer is not, and is generated on each request.

The term "donut hole" refers to the inner dynamic layer, which is surrounded by the static outer layer. The outer layer is cached in its entirety, including its HTML, CSS, and JavaScript files. When a user requests the web page, the server first checks the cache for the outer layer. If it is found, it is returned to the user, which can result in a significant performance improvement since the server does not need to generate the outer layer on each request.

However, the inner dynamic layer, which contains content that changes frequently, such as user-specific data or real-time information, is not cached. This ensures that the content remains fresh and up-to-date. When a request for the inner layer is received, the server generates it dynamically and inserts it into the cached outer layer, creating the final page that is returned to the user.

Donut Hole Caching is a balance between the performance benefits of caching and the need for up-to-date, dynamic content.

### Availability

In Smartstore, output caching is handled by a commercial module that is **not** part of the open-source Community Edition. The module builds on the caching infrastructure provided by the Smartstore core and provides all the necessary implementations to actually enable and operate the output cache.

But, nevertheless, when developing custom modules for Smartstore, it's important to take output caching into account. This means that you need to ensure that your module works correctly with the output caching system. This may involve configuring the caching settings for the module or using cache tags to ensure that the cache is cleared when necessary.

## Cacheable routes

* _Terminology: page --> static outer layer, component --> dynamic inner layer (see above)_
* A cacheable route represents the stringified route to a page or a view component (route identifier)
* Output caching is always opt-in: only explicitly specified routes are candidates. Meaning: if you develop a module and do not setup any caching stuff, nothing will be cached.
* The route identifier looks like this:
  * **Full page** route pattern: `[{Module}/]{ControllerShortName}/{Action}`. Module must be omitted if controller is part of the application core. Example: `Smartstore.Blog/Blog/List`, `Catalog/Category`
  * **View component** route pattern: `vc:[{Module}/]{ComponentShortName}`. Module must be omitted if component is part of the application core. Example: `vc:SearchBox`, `vc:Smartstore.Blog/BlogSummary`.
* If any **full page** route pattern matches the current request route, the generated page will be saved in cache.&#x20;
* Several pieces of environmental information are used to compute the cache entry key (so that the cached entry varies by them):
  * Path and query string
  * Current language
  * Current currency
  * Current store
  * Current theme
  * All customer roles
  * App version
* But the output of contained view components that don't match any registered **component** route pattern will be removed from the outer layer. Their content is generated dynamically on each request later. This procedure (replacing these _donut holes_ in a deferred manner) is also called _substitution_.
  * But: for this work you have to ensure that the method arguments of the component's `Invoke` method do not depend on any parent/outer view model. Because: this model is gone in successive requests ('cause content comes from cache)
  * You also have to ensure that the arguments can be serialized to JSON
    * Because in the cached outer layer, the component output is replaced by a JSON representation of the component metadata and actual `Invoke` arguments.
  * But if the component is made cacheable, you don't have to care about parameter limitations ('cause the output is **not** removed for later substitution / is cached along with the outer layer).
  * WARN: Never ever make a component cacheable if it somehow displays user-related data (like cart for example). Another user could see it when it comes from cache.
* Here's how you specify cacheable routes for your module:
  * Add a class that implements [ICacheableRouteProvider](https://github.com/smartstore/Smartstore/blob/main/src/Smartstore.Core/Platform/OutputCache/ICacheableRouteProvider.cs). By convention we call it `CacheableRoutes` and make it internal. No need for DI registration.
  * _SAMPLE_
  * HINT: backend pages are never cached, only frontend pages.

## Display control & invalidation

Tags

## Invalidation observer
