---
title: API Reference
order: 2
description: Complete reference for the Roblox Requests API.
category: reference
---

## Module

### http.send

`http.send(method, url, [options]) -> Response`

Creates and sends an HTTP request with the given method, URL, and options.

Options:

| Name             | Type                        | Description                                                                            |
|------------------|-----------------------------|----------------------------------------------------------------------------------------|
| headers          | dictionary                  | Headers to be used with the Request.                                                   |
| query            | dictionary                  | Query string parameters to add to the URL.                                             |
| data             | string OR table OR FormData | Data to send in the body of the request. Tables will automatically be encoded as JSON. |
| cookies          | dictionary OR CookieJar     | Cookies to send with request.                                                          |
| ignore_ratelimit | bool                        | If true, the rate-limit will be ignored for this request.                              |

### http.get, post, head, put, delete, patch, options

`http.get(url, [options]) -> Response`

Shortcut methods for `http.send`.

### http.promise_send

`http.promise_send(method, url, [opts]) -> Promise`

This has the same functionality as `http.send` but returns a Promise.

### http.promise_get, promise_post, promise_...

`http.promise_get(url, [opts]) -> Promise`

Shortcut methods for `http.promise_send`

### http.set_ratelimit

`http.set_ratelimit(requests, period)`

Changes global rate limit. Default is 250 requests / 30 seconds.

## http.Request

`http.Request(method, url, [options]) -> Request`

Creates and prepares a Request object.

### Attributes

Note: These attributes are all read-only. Use the provided methods instead of setting them directly.

- **method** (string) - HTTP verb used for request
- **url** (string) - URL request is sent to
- **headers** (dictionary) - Headers used for this request
- **query** (dictionary) - Parameters to add to URL
- **data** (string) - Raw data sent with request

### Request:update_headers

`Request:update_headers(headers)`

Updates request headers based on a dictionary of new headers.

### Request:update_query

`Request:update_query(params)`

Updates URL query string based on a dictionary of parameters.

### Request:set_data

`Request:set_data(data)`

Sets new data for request. The data can be a string, table (JSON), or FormData.

### Request:send

`Request:send() -> Response`

Sends the request and returns a `Response` object.

### Request:promise

`Request:promise() -> Promise`

Returns a [Promise](https://eryn.io/roblox-lua-promise/).
The Promise will resolve with the response object if the response has a 2xx status code. Otherwise, the Promise will reject with this rejection table:

| Name             | Type                        | Description                                                                            |
|------------------|-----------------------------|----------------------------------------------------------------------------------------|
| request_sent     | bool                        | Whether the request was sent successfully.                                             |
| response         | bool                        | The response object. This value is only present if `request_sent` is `true`.           |
| error            | string                      | The error thrown during preparation of the request if `request_sent` is `false`.       |


## Response

### Attributes

- **request** (Request) - Request that generated this response
- **url** (string) - Requests' URL
- **method** (string) - Request's HTTP method
- **ok** (bool) - `true` if response has OK status code (200 \<= code \< 300)
- **status_code** (number) - Status code of reponse
- **message** (string) - Status message of response
- **response_time** (number) - Time the request took in seconds
- **timestamp** (number) - Epoch time the request was sent at
- **headers** (dictionary) - Headers sent in response
- **text** (string) - Response body
- **cookies** (CookieJar) - New cookies sent in this response

### Response:json

`Response:json() -> Variant`

Converts JSON response to a Lua object.

## http.Session

`http.Session([base_url]) -> Session`

Creates a session with an optional base URL.

All main module methods also apply to sessions.

### Attributes

- **headers** (dictionary) - Default headers used with each request
- **cookies** (CookieJar) - Current session cookies. Updates automatically from Set-Cookie header
- **base_url** (string) - Base URL to prefix each request with. If a request URL contains the HTTP protocol (http(s)://), this will be ignored
- **ignore_ratelimit** (bool) - If true, all requests will ignore the global rate-limit

### Session:set_headers

`Session:set_headers(new_headers)`

Updates headers dictionary with values of new_headers.

### Session:set_ratelimit

`Session:set_ratelimit([requests[, seconds]])`

Sets custom rate-limit for the Session. Requests sent by the session must follow this ratelimit and the global ratelimit.

If no values are provided, this will disable the session ratelimit. This will not disable the global ratelimit.

This can be changed at any time by calling the function again.

### Session:send

`Session:send(method, url, [opts]) -> Response`

Creates and sends a request prepared with session defaults.
Options specified here will override the session values.

### Session:get, post, head, put, delete, patch, options

`Session:get(url, [opts]) -> Response`

Shortcut methods for `Session:send`

### Session:promise_send

`Session:promise_send(method, url, [opts]) -> Promise`

This has the same functionality as `Session:send` but returns a Promise.

### Session:promise_get, promise_post, promise_...

`Session:promise_get(url, [opts]) -> Promise`

Shortcut methods for `Session:promise_send`

### Session:Request

`Session:Request(method, url, [opts]) -> Request`

Creates a Request prepared with session defaults.
Options specified here will override the session values.

## http.FormData

`http.FormData(fields) -> FormData`

Creates form data based on given fields. Fields should be passed in a dictionary.

### FormData:AddField

`FormData:AddField(name, value)`

Adds field with given name and value.

## http.File

`http.File(content) -> File`

`http.File(filename, content, [content_type]) -> File`

Creates a file for use in a FormData field. If no content type is specified, it will
be guessed from the file extension.

## http.CookieJar

CookieJars are used to store and persist cookies. This is the object referenced by `session.cookies`.

### CookieJar:insert

`CookieJar:insert(name, value, options)`

Creates a new cookie with an optional domain and path

Options:

| Name   | Type   | Description                                                                 |
|--------|--------|-----------------------------------------------------------------------------|
| domain | string | If specified, the cookie will only apply to URLs on this domain.            |
| path   | string | If specified, the cookie will only apply to URLs that begin with this path. |

### CookieJar:delete

`CookieJar:delete(name)`

Deletes a cookie.
