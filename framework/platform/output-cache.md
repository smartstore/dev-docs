---
description: Improve performance and scalability
---

# 🥚 Output Cache

## Concept

Output caching is a technique used to improve the performance and scalability of Smartstore by storing the generated output of pages on the server side in memory, relational database or in a REDIS database. When a client requests a store page, the server first checks if a cached copy of the page exists. If it does, the server can serve the cached copy to the client without generating the page again, reducing the processing and network overhead.

Smartstore also ensures that cached content is invalidated or refreshed when it becomes stale or outdated, and that private or sensitive data is not inadvertently cached. Proper caching can significantly reduce the response time and resource usage of a store, improving the user experience and reducing server load.

### Donut Hole Caching

Smartstore follows the so-called _Donut Hole Caching_ strategy. In this strategy, a dynamic web page is divided into two parts: a non-dynamic or semi-static outer layer, and a dynamic inner layer. The outer layer is cached, while the inner layer is not, and is generated on each request.

The term "donut hole" refers to the inner dynamic layer, which is surrounded by the static outer layer. The outer layer is cached in its entirety, including its HTML, CSS, and JavaScript files. When a user requests the web page, the server first checks the cache for the outer layer. If it is found, it is returned to the user, which can result in a significant performance improvement since the server does not need to generate the outer layer on each request.

However, the inner dynamic layer, which contains content that changes frequently, such as user-specific data or real-time information, is not cached. This ensures that the content remains fresh and up-to-date. When a request for the inner layer is received, the server generates it dynamically and inserts it into the cached outer layer, creating the final page that is returned to the user.

Donut Hole Caching is a balance between the performance benefits of caching and the need for up-to-date, dynamic content.
