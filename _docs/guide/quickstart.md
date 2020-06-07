---
title: Quick Start
order: 2
description: Getting started with Roblox Requests.
category: guide
---

This page gives an introduction into how to use Roblox Requests.
From this point forward, we will assume that Requests is installed in `ReplicatedStorage`.

## Make a request

Making a request is simple. We'll begin by obtaining the module:

```lua
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local http = require(ReplicatedStorage.http)
```
{: .lua}

Now, we'll make a GET request.

```lua
local r = http.get("https://api.github.com/events")  -- GitHub's public timeline
```
{: .lua}

This gives us a `Response` object containing the information we need.

What about the other HTTP methods?

```lua
local r = http.post("https://httpbin.org/post", { data={key="value"} })

local r = http.put("https://httpbin.org/put", { data={key="value"} })
local r = http.patch("https://httpbin.org/patch", { data={key="value"} })
local r = http.delete("https://httpbin.org/delete")
local r = http.head("https://httpbin.org/head")
local r = http.options("https://httpbin.org/options")
```
{: .lua}

All methods accept an options dictionary for any additional information.


## Passing Parameters in URLs

It's common to send data in a URL's query string. With HttpService, constructing key/value pairs in URLs gets very tedious.
Requests allows you to provide these arguments as a dictionary using the `query` option. For example:

```lua
local payload = {key1 = "value1", key2 = "value2"}
local r = http.get("https://httpbin.org/get", { query=payload })
```
{: .lua}

You can see the encoded URL through the `.url` attribute:

```lua
print(r.url)
-- https://httpbin.org/get?key1=value1&key2=value2
```

## Response Content

Requests makes it easy to read different kinds of response content.

```lua
local r = http.get("https://api.github.com/orgs/Roblox/repos")
print(r.text)
-- [{"id":10803524,"node_id":"MDEwOlJlcG9zaXRvcnkxMD...
```
{: .lua}

The `text` attribute provides the plaintext response body, regardless of the given content-type.
If you'd like to decode JSON data, you can use the `:json()` method:

```lua
local r = http.get("https://api.github.com/orgs/Roblox/repos")
local repos = r:json()
print(#repos)
-- 30
```
{: .lua}

In the case that JSON decoding fails, an exception will be raised. It should be noted, however, that a successful `:json()` call
does **not** indicate the success of the response. Some servers may return a JSON object in a failed response, such as error details.

To check that a response is successful, use `r.ok` or `r.status_code`.

### Response Headers

Response headers are often needed when using an API. They can be accessed as a dictionary through the `headers` attribute:

```lua
local r = http.get("https://upload.wikimedia.org/wikipedia/commons/3/38/JPEG_example_JPG_RIP_001.jpg")

print(r.headers["Last-Modified"])
-- Wed, 08 Oct 2014 23:28:44 GMT
```
{: .lua}

HTTP headers are case-insensitive, so you can access them however you like:

```lua
print(r.headers["Last-Modified"])
-- Wed, 08 Oct 2014 23:28:44 GMT

print(r.headers["content-type"])
-- image/jpeg

print(r.headers["SeRvEr"])
-- ATS/8.0.3
```
{: .lua}

## Response Status Codes

We can check the response status code and message:

```lua
local r = http.get("https://httpbin.org/get")
print(r.status_code, r.message)
-- 200 OK
print(r.ok)
-- true
```
{: .lua}

All is well.


## Custom Headers

If you'd like to add HTTP headers, just pass a dictionary to the `headers` option.

For example, API keys are often specified in the headers of a request:

```lua
local url = "https://api.github.com/some/endpoint"
local custom_headers = {Authorization = "api-key-here"}

local r = http.get(url, { headers=custom_headers })
```
{: .lua}

!!! note
    The `User-Agent` and `Roblox-Id` headers cannot be overridden.

### Cookies

To include custom cookies in a request, use the `cookies` option:

```lua
local r = http.get("http://httpbin.org/cookies", { cookies={a=1, b=2} })
print(r.text)
--   {
--     "cookies": {
--       "a": "1", 
--       "b": "2"
--     }
--   }
```
{: .lua}

## POST JSON Payloads

It's common to send JSON data via HTTP. To do this, just pass a table to the `data` argument. It will be detected as JSON and automatically
encoded when the request is made:

```lua
local payload = {key1 = "value1", list={"a", "b", "c"}}

local r = http.post("https://httpbin.org/post", { data=payload })
print(r.text)
-- {
--     ...
--     "json": {
--         "key1": "value1", 
--         "list": [
--             "a", 
--             "b", 
--            "c"
--        ]
--  }, 
--  ...
-- }
```
{: .lua}

## POST Form Data

Requests supports URL encoded and multipart encoded forms through the `FormData` class. Creating forms is simple:

```lua
local form = http.FormData({
    key1 = "value1",
    key2 = {"value2", "value3"}
})
```
{: .lua}

Fields can also be added after the form is created:

```lua
local form = http.FormData()

form:AddField("size", "large")
form:AddField("toppings", {"cheese", "pepperoni"})
```
{: .lua}

To use a form in a POST request, just set it as the `data` option:

```lua
local form = http.FormData({"key", "value"}, {"key2", "value2"})
local r = http.post("https://httpbin.org/post", { data=form })

print(r.text)
--    {
--    ...
--      "form": {
--        "key": "value", 
--        "key2": ["value2", "value3"]
--      }, 
--      ...
--    }
```
{: .lua}
### File Uploads

Requests makes it easy to upload files:

```lua
local example = http.File("example.txt", "This is an example text file.")

local form = http.FormData({
    file = example
})

local r = http.post("https://api.anonymousfiles.io/", { data=form })
print(r:json().url)
-- https://anonymousfiles.io/et78xhGK/
```
{: .lua}

Binary files can also be uploaded. This example downloads an image then uploads it to another site:

```lua
local image_url = "https://upload.wikimedia.org/wikipedia/en/a/a9/Example.jpg"

local image = http.File("example.jpg", http.get(image_url).text)

local form = http.FormData()
form:AddField("file", image)

local r = http.post("https://api.anonymousfiles.io/", { data=form })
print(r:json().url)
-- https://anonymousfiles.io/zwtSGyvZ/
```
{: .lua}

Any non-text files are encoded using Base64.

Requests will try to guess the file type from the extension. If you'd like, however, you can set it yourself:

```lua
local file = http.File("extensionless_name", "<html>But we know it's HTML</html>", "text/html")
```
{: .lua}