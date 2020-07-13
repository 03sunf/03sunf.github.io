---
layout: post
title:  "Weird WSGI HTTP parser logic"
date:   2020-07-14 03:10:00
tags:   [Python, Flask]
---
### Description
If Flask Server exposed to an external network alone without a front server like Reverse Proxy, Remote users can Manipulate maliciously the value of `request.host` value. And also Remote user can request with invalid HTTP method.
<br/>
<br/>


### Server code
When there is a server running with the following code, a normal HTTP request returns the requested hostname. But remote user can control `request.host` value with oneline request. This weird parser logic makes the potential to be derived from another attack, such as SSRF.

```python
from flask import Flask
from flask import request

app = Flask(__name__)

@app.route("/", methods=["GET"])
def home():
    return request.host

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
```
<br/>
<br/>


### Request
#### 1. Nomal oneline request
```
$ nc 10.211.55.3 8080
GET / HTTP/1.1

HTTP/1.0 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 12
Server: Werkzeug/1.0.0 Python/3.5.2
Date: Mon, 13 Jul 2020 15:47:17 GMT

0.0.0.0:8080
```
<br/>
<br/>


#### 2. Used invalid hostname in Host header.
```
$ nc 10.211.55.3 8080
GET / HTTP/1.1
Host: google.com

HTTP/1.0 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 9
Server: Werkzeug/1.0.0 Python/3.5.2
Date: Mon, 13 Jul 2020 15:51:41 GMT

google.com
```
<br/>
<br/>


#### 3. Used invalid hostname in Path field, and also used invalid HTTP Version.
```
$ nc 10.211.55.3 8080
GET X://google.com/ HTTP/0.1337

HTTP/1.0 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 10
Server: Werkzeug/1.0.0 Python/3.5.2
Date: Mon, 13 Jul 2020 15:54:06 GMT

google.com
```
<br/>
<br/>


### Analyse
WSGI Handler passes environ of request to Flask.wsgi_app(), and request_context() creates Request object that is normally used "Flask.request". Also, the object is stored in _request_ctx_stack and used globally.
This is the part of creating WSGI environ object by WSGIRequestHandler.make_environ in serving.py.
```python
# werkzeug/serving.py
# HTTP Scheme only can be http or https.
def make_environ(self):
    # request_url created by url_parse function.
    request_url = url_parse(self.path)

    def shutdown_server():
        self.server.shutdown_signal = True

    url_scheme = "http" if self.server.ssl_context is None else "https"
    if not self.client_address:
        self.client_address = "<local>"
    if isinstance(self.client_address, str):
        self.client_address = (self.client_address, 0)
    else:
        pass
        
...

environ = {
    "wsgi.version": (1, 0),
    "wsgi.url_scheme": url_scheme,
    "wsgi.input": self.rfile,
    "wsgi.errors": sys.stderr,
    "wsgi.multithread": self.server.multithread,
    "wsgi.multiprocess": self.server.multiprocess,
    "wsgi.run_once": False,
    "werkzeug.server.shutdown": shutdown_server,
    "SERVER_SOFTWARE": self.server_version,
    "REQUEST_METHOD": self.command,
    "SCRIPT_NAME": "",
    "PATH_INFO": wsgi_encoding_dance(path_info),
    "QUERY_STRING": wsgi_encoding_dance(request_url.query),
    # Non-standard, added by mod_wsgi, uWSGI
    "REQUEST_URI": wsgi_encoding_dance(self.path),
    # Non-standard, added by gunicorn
    "RAW_URI": wsgi_encoding_dance(self.path),
    "REMOTE_ADDR": self.address_string(),
    "REMOTE_PORT": self.port_integer(),
    "SERVER_NAME": self.server.server_address[0],
    "SERVER_PORT": str(self.server.server_address[1]),
    "SERVER_PROTOCOL": self.request_version,
}

...

# Per RFC 2616, if the URL is absolute, use that as the host.
# We're using "has a scheme" to indicate an absolute URL.
if request_url.scheme and request_url.netloc:
    environ["HTTP_HOST"] = request_url.netloc
```
<br/>
<br/>


Now we know `HTTP_HOST` is created by request_url.netloc, and also request_url created by url_parse function, and this one is defined in urls.py. 
```python
# werkzeug/urls.py
def url_parse(url, scheme=None, allow_fragments=True):
    """Parses a URL from a string into a :class:`URL` tuple.  If the URL
    is lacking a scheme it can be provided as second argument. Otherwise,
    it is ignored.  Optionally fragments can be stripped from the URL
    by setting `allow_fragments` to `False`.

    The inverse of this function is :func:`url_unparse`.

    :param url: the URL to parse.
    :param scheme: the default schema to use if the URL is schemaless.
    :param allow_fragments: if set to `False` a fragment will be removed
                            from the URL.
    """
    s = make_literal_wrapper(url)
    is_text_based = isinstance(url, text_type)

    if scheme is None:
        scheme = s("")
    netloc = query = fragment = s("")
    # Part 1
    i = url.find(s(":"))
    if i > 0 and _scheme_re.match(to_native(url[:i], errors="replace")):
        # make sure "iri" is not actually a port number (in which case
        # "scheme" is really part of the path)
        rest = url[i + 1 :]
        if not rest or any(c not in s("0123456789") for c in rest):
            # not a port number
            scheme, url = url[:i].lower(), rest

    # Part 2
    if url[:2] == s("//"):
        delim = len(url)
        for c in s("/?#"):
            wdelim = url.find(c, 2)
            if wdelim >= 0:
                delim = min(delim, wdelim)
        # Part 3
        netloc, url = url[2:delim], url[delim:]
        if (s("[") in netloc and s("]") not in netloc) or (
            s("]") in netloc and s("[") not in netloc
        ):
            raise ValueError("Invalid IPv6 URL")

    if allow_fragments and s("#") in url:
        url, fragment = url.split(s("#"), 1)
    if s("?") in url:
        url, query = url.split(s("?"), 1)

    result_type = URL if is_text_based else BytesURL
    return result_type(scheme, netloc, url, query, fragment)
```
<br/>
<br/>


We can check url_parse function redefine scheme variable and url variable if there is more than one semicolon in url variable.
When we request `GET X://google.com/ HTTP/0.1337`, scheme variable be defined `http` and url variable be defined `//google.com/` in comment `Part 1`. Also comment `Part 2` compares first 2bytes with `//` and `google.com` is stored in the netloc variable.

```python
# GET / HTTP/1.1
url => /
netloc =>
url => /
netloc =>
10.211.55.2 - - [14/Jul/2020 02:24:57] "GET / HTTP/1.1" 200 -


# GET X://google.com/ HTTP/1.1
url => X://google.com/
scheme => x | url => //google.com/
after netloc => google.com | after url => /
netloc => google.com
10.211.55.2 - - [14/Jul/2020 02:26:09] "GET x://google.com/ HTTP/1.1" 200 -
```
<br/>
<br/>


This is the final WSGI environ object `GET X://google.com/ HTTP/0.1337`. I guess some Flask module referencing the Flask.request object may have an effect.
```
{
   "wsgi.multiprocess":False,
   "SERVER_PORT":"8080",
   "werkzeug.request":"<Request"   "http://google.com/"[
      "GET"
   ]   ">",
   "SCRIPT_NAME":"",
   "wsgi.errors":"<_io.TextIOWrapper name="   "<stderr>"   "mode="   "w"   "encoding="   "ANSI_X3.4-1968"   ">",
   "wsgi.run_once":False,
   "RAW_URI":"X://google.com/",
   "wsgi.input":<_io.BufferedReader name=4>,
   "REMOTE_ADDR":"10.211.55.2",
   "wsgi.multithread":True,
   "HTTP_HOST":"google.com",
   "wsgi.version":(1,
   0),
   "REQUEST_URI":"X://google.com/",
   "SERVER_SOFTWARE":"Werkzeug/1.0.0",
   "QUERY_STRING":"",
   "REMOTE_PORT":55661,
   "PATH_INFO":"/",
   "SERVER_NAME":"0.0.0.0",
   "SERVER_PROTOCOL":"HTTP/0.1337",
   "wsgi.url_scheme":"http",
   "werkzeug.server.shutdown":<function WSGIRequestHandler.make_environ.<locals>.shutdown_server at 0x7f1e0791d378>,
   "REQUEST_METHOD":"GET"
}{
   "wsgi.multiprocess":False,
   "SERVER_PORT":"8080",
   "werkzeug.request":"<Request"   "http://google.com/"[
      "GET"
   ]   ">",
   "SCRIPT_NAME":"",
   "wsgi.errors":"<_io.TextIOWrapper name="   "<stderr>"   "mode="   "w"   "encoding="   "ANSI_X3.4-1968"   ">",
   "wsgi.run_once":False,
   "RAW_URI":"X://google.com/",
   "wsgi.input":<_io.BufferedReader name=4>,
   "REMOTE_ADDR":"10.211.55.2",
   "wsgi.multithread":True,
   "HTTP_HOST":"google.com",
   "wsgi.version":(1,
   0),
   "REQUEST_URI":"X://google.com/",
   "SERVER_SOFTWARE":"Werkzeug/1.0.0",
   "QUERY_STRING":"",
   "REMOTE_PORT":55661,
   "PATH_INFO":"/",
   "SERVER_NAME":"0.0.0.0",
   "SERVER_PROTOCOL":"HTTP/0.1337",
   "wsgi.url_scheme":"http",
   "werkzeug.server.shutdown":<function WSGIRequestHandler.make_environ.<locals>.shutdown_server at 0x7f1e0791d378>,
   "REQUEST_METHOD":"GET"
}
```
