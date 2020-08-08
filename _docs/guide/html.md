---
title: HTML/XML Parsing
order: 5
description: Parsing and navigating HTML and XML documents with Roblox Requests.
category: guide
---

Roblox Requests allows you to generate a DOM from HTML or XML directly from your response.
This allows you to make use of many useful APIs that use the XML format, as well as fetch data
from any webpage.

## DOM From Response Content

### HTML

In the Quick Start, we learned to parse JSON response content using the `:json()` method:

```lua
local r = http.get("https://httpbin.org/get")
local data = r:json()
```

Parsing an HTML or XML body is just as easy. To generate a DOM from an HTML response, we'll use the `:html()` method.

``` lua
local r = http.get("https://github.com/")
local root = r:html()
```

Requests will check the response's Content-Type before parsing it, and make sure the response is actually HTML. If the content type
doesn't match, it'll throw an error:

``` lua
local r = http.get("https://httpbin.org/get")
local root = r:html()
-- error: [http] Response is not specified as HTML.
```

If a server isn't specifying the content type correctly, you can ignore this with the optional `ignore_content_type` argument:

``` lua
local root = r:html(true)  -- set ignore_content_type to true
```

### XML

Parsing XML content is just as easy, using the `:xml()` method. Similar to `:html()`, this will throw an error if the content-type is not XML based.

``` lua
local r = http.get("https://www.w3schools.com/xml/note.xml")
local my_xml = r:xml()
```

## Parse From String

Requests also allows you to generate a DOM from a string"

``` lua
local html = http.parse_html("<html> </html>")
local xml = http.parse_xml("<xml> </xml>")
```


## Navigating the Document

You can find specific elements in a document using a subset of jQuery's selector strings:

``` lua
local elements = root:select(selectorstring)
```

Or in shorthand:

``` lua
local elements = root(selectorstring)
```

This will return a list of elements, all of which are of the same type as the root element, and thus support selecting as well, if ever needed:

``` lua
for _, element in ipairs(elements) do
    print(element.name)
    local subs = element(subselectorstring)
    for _, sub in ipairs(subs) do
        print(sub.name)
    end
end
```

### Selectors

Supported selectors are a subset of jQuery's selectors:

- `"*"` - all contained elements
- `"element"` - elements with the given tagname
- `"#id"` - elements with the given id attribute value
- `".class"` - elements with the given classname in the class attribute
- `"[attribute]"` - elements with an attribute of the given name
- `"[attribute='value']"` - equals: elements with the given value for the given attribute
- `"[attribute!='value']"` - not equals: elements without the given attribute, or having the attribute, but with a different value
- `"[attribute|='value']"` - prefix: attribute's value is given value, or starts with given value, followed by a hyphen (`-`)
- `"[attribute*='value']"` - contains: attribute's value contains given value
- `"[attribute~='value']"` - word: attribute's value is a space-separated token, where one of the tokens is the given value
- `"[attribute^='value']"` - starts with: attribute's value starts with given value
- `"[attribute$='value']"` - ends with: attribute's value ends with given value
- `":not(selectorstring)"` - elements not selected by given selector string
- `"ancestor descendant"` - elements selected by the `descendant` selector string, that are a descendant of any element selected by the `ancestor` selector string
- `"parent > child"` - elements selected by the `child` selector string, that are a child element of any element selected by the `parent` selector string

Selectors can be combined; e.g. `".class:not([attribute]) element.class"`

## Elements

All tree elements have the following properties and methods:

### Properties
- `.name`: the element's tagname
- `.attrs`: a table with keys and values for the element's attributes; `{}` if none
- `.id`: the value of the element's id attribute; `nil` if not present
- `.classes`: an array with the classes listed in element's class attribute; `{}` if none
- `.children`: an array with the element's child elements
- `.parent`: the element that contains this element
- `.level`: how deep the element is in the tree; root level is `0`
- `.root`: the root element of the tree
- `.descendants`: a **Set** containing all elements in the tree beneath this element

Some element properties are specified as Sets. You must call `:to_list()` on these properties to get their values.
{: .warning}

### Methods
- `:content()`: the raw text between the opening and closing tags of the element
- `:text()`: the complete element text, starting with `"<tagname"` and ending with `"/>"` or `"</tagname>"`
- `:links()`: an array with all the links in the document. Links are not guaranteed to be fully qualified.
- `:absolute_links()`: an array with all fully qualified links in the document.

Source for the parser can be found at [the project's GitHub](https://github.com/jpatrickdill/rbx-html).

## Example Usage: Finding All Links

``` lua
local document = http.get("https://github.com"):html()

for _, url in ipairs(document:absolute_links()) do
    print(url)
end

-- https://github.com/#start-of-content
-- https://help.github.com/articles/supported-browsers
-- https://github.com/
-- https://github.com/join?ref_cta=Sign+up&amp;ref_loc=header+logged+out&amp;ref_page=%2F&amp;source=header-home
-- ...
```

Powered by Vadim A. Misbakh-Soloviov's [lua-htmlparser](https://github.com/msva/lua-htmlparser).