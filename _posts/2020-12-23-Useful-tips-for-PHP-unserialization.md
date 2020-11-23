---
layout: post
title:  "Useful tips for PHP unserialization"
date:   2020-12-23 22:30:00
tags:   [PHP, OI]
---

## Description
This post contains useful tips about php unserialization.
<br/>
<br/>


## PHP Serialization
PHP Serialized data follows this data type. and also you can easily understand a serialized output of a class that is under this.

```
Anatomy of a serialize()'ed value:

String
s:size:value;

String(unicode)
S:size:value;

Integer
i:value;

Boolean
b:value; (does not store "true" or "false", does store '1' or '0')

Null
N;

Array
a:size:{key definition;value definition;(repeated per element)}

Object
O:strlen(object name):object name:object size:{s:strlen(property name):property name:property definition;(repeated per property)}
```
```php
class Testclass
{
  public $p1 = 'public';
}
```
```php
O:9:"Testclass":1:{s:2:"p1";s:6:"public";}
```
<br/>
<br/>


There are three different types of properties in this class. and three different name come out when you serialize it.
```php
class Testclass
{
    public $p1 = 'public';
    protected $p2 = 'protected';
    private $p3 = 'private';
}
```
```
public    -> p1
protected -> \x00*\x00p2
private   -> \x00Textclass\x00p3
```
<br/>
<br/>


## Tips
When you face the source code like below, you can easily bypass with unicode string type.

```php
class Testclass
{
  protected $p1 = 'changeme';
}

if (strpos($input, '*') !== false) {
  exit();
}
else {
  unserialize($input);
}
```
```php
O:9:"Testclass":1:{S:5:"\x00\x2a\x00p1";s:4:"hehe";}
```
