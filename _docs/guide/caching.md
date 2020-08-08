---
title: Caching Responses
order: 4
description: How to cache responses from static data sources.
category: guide
---

## When/why should I cache?

Many CDNs automatically cache their responses for you. Local caching, however, will reduce ratelimit and network usage by using locally
stored responses.

You should only cache responses from sources that will *always give you the same content.* Don't cache any API
endpoints that return different data when using the same request options (query string, body, etc.)

## Cache Patterns

In order to enable caching, we have to mark certain URLs as safe to cache. To do this, we use URL patterns.

Example patterns:
- `"github.com"` - Matches all subdirectories for GitHub
- `"*.github.com"` - Matches all subdomains and subdirectories for GitHub
- `"github.com/"` - Matches ONLY "http(s)://github.com/"
- `"github.com/*/stars"` - Matches stars directory for any GitHub user

URL patterns use the asterisk (*) as a wildcard that allows any data. This allows you to cache a large range of URLs.

### Special Cases

If you provide a domain name such as `"github.com"` with no trailing slash, it will be assumed that you want to cache every URL
on GitHub. However, using `"github.com/"` will only cache the front page for GitHub.

Passing `"*.github.com"` to include subdomains will implicitly include `"github.com"`.

## Setting up Cache

To set up caching for a URL, we'll use the `http.cache` object. Here, we set up the cache to store every `"github.com"` URL:

```lua
http.cache.cache_locally("github.com")
```

We can also pass multiple URLs in one call:

```lua
http.cache.cache_locally({"github.com", "scratch.mit.edu"})
```

To check if a response was accessed from the cache, use the `Response.from_cache` attribute:

```lua
http.cache.cache_locally("github.com")

print(http.get("https://github.com/").from_cache)  
-- false

print(http.get("https://github.com/").from_cache)
-- true, using initial response that was cached
```


### Cache Expiry

If you have data that you only want to store for a certain amount of time, you can set an expiry:

```lua
http.cache.cache_locally("openstreetmap.org", {
    expires = 5*60  -- expire after 5 minutes
})
```

If the cached response is more than 5 minutes old, it will send a new request and replace the old one.

## Global Caching

Requests also supports global caching, backed by Data Stores and MessagingService. This is only recommended if your game is accessing very large static resources.
To enable global caching for a set of URLs, just use the `cache_globally` function:

```lua
http.cache.cache_globally("osmbuildings.org")

-- with expiry
http.cache.cache_globally("osmbuildings.org", {expires=60})

```

Global caching works by keeping an index of cached URLs in each server that is updated in realtime via MessagingService. This way, Requests knows if
a URL is cached without having to check the Data Store. If a response is cached, it downloads it from the Data Store instead of the actual server.
