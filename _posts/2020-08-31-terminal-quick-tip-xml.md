---
layout: "post"
author: "jimdoescode"
title: "Terminal Quick Tip - Validate XML Against XSD"
date: "2020-08-31 13:50:14"
tags: [quicktip,terminal,bash,shell]
---

If you have some XML in a file and you also have the corresponding XSD in a file, here's how you validate the XML matches the XSD.

```sh
$ xmllint --schema your.xsd your.xml --noout
```

If the XML matches the schema defined in the XSD you should see output like:

```sh
$ your.xml validates
```
