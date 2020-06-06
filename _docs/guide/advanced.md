---
title: Advances
order: 3
description: Advanced usage of Roblox Requests.
---

# Advanced Usage

This page covers some more advanced features of Roblox Requests.

## Session Objects

The Session object allows you to persist certain data across requests.
It also persists cookies across all requests made from the Session instance.

A Session object has all the same methods of the main Requests API.

Persisting cookies across requests:
```lua
local session = http.Session()

session.cookies:insert("sessioncookie", "123456789", {domain="httpbin.org", path="/cookies"})  -- add new cookie to httpbin.org/cookies
local r = session:get("https://httpbin.org/cookies")

print(r.text)
-- {"cookies": {"sessioncookie": "123456789"}}

print(session:get("https://httpbin.org/").text)  -- cookies will only be sent to their set path
-- {"cookies": {}}

```

Sessions can also be used to provide default data to requests.

```lua
local session = http.Session()
session.headers = {
	["X-Test"] = true
}

-- both "x-test" and "x-test2" are sent
session:get("https://httpbin.org/headers", { headers={["X-Test2"] = "true"} })
```

Additional headers can also be merged with the current headers:

```lua
session:set_headers({
	["x-third"] = true
})  
-- session.headers now contains both "X-Test" and "x-third"
```

Any options that you pass to a request method will be merged with the session-level values.
Method-level parameters override session parameters.

However, method-level parameters aren't persisted across requests, even if
using a session. This example will only send cookies with the first request:

```lua
local session = http.Session()

local r = session:get("https://httpbin.org/cookies", { cookies={["temp"] = "value"} })
print(r.text)
-- {"cookies": {"temp": "value"}}

local r2 = session:get("https://httpbin.org/cookies")
print(r2.text)
-- {"cookies": {}}
```

## Request Objects

When you send a request with the `http.get()` method, a `Request` object is actually created and
prepared with any data you passed. You may wish to do something else
with the request before it is sent. This is possible by creating the Request directly:

```lua
local request = http.Request("POST", "https://httpbin.org/post")

request:set_data("request body")

local response = request:send()
```

The same is possible from a session by using `Session:Request()`.

## Rate-limiting

Roblox Requests ratelimits all HTTP requests sent through the module. If a request would tip the ratelimit past 500
requests/minute, Requests will issue a warning and retry in 5 seconds.

By default, the rate-limiter allows 250 requests every 30 seconds, smoothing bursts over a 1 minute period.
You can change these settings with `http.set_ratelimit`:

```lua
local http = require(ReplicatedStorage.http)

http.set_ratelimit(10, 60)  -- allow 10 requests every 60 seconds
```

If you'd like a request to ignore the rate-limit, just set the `ignore_ratelimit` option to `true`:

```lua

if custom_ratelimit_function() then
	http.get("https://httpbin.org", { ignore_ratelimit=true })
end
```

You can also disable a session's rate-limits by setting the `ignore_ratelimit` property:

```lua
local session = http.Session()

session.ignore_ratelimit = true

-- let's break roblox!
while wait() do
	session:get("https://httpbin.org/get")
end
```

It's recommended that you only do this if you're applying some other limiting function.

### Session rate-limits

If you want to make a specific session follow a different rate-limit, you can set one:

```lua
local session = http.Session()

session:set_ratelimit(10, 60)  -- 10 requests/minute

local i = 0
while wait(1) do
	i = i + 1

	http.get("https://httpbin.org/get")  -- module level requests still follow normal rate-limit

	if i%6 == 0 then  -- only send every 6 seconds (10/min)
		session:get("https://httpbin.org/get")
	end
end
```

Unlike the global rate-limiter, this one can be changed any time you like by calling `:set_ratelimit()`.
It can also be removed by calling `:disable_ratelimit()`.

