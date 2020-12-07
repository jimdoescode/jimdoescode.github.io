---
layout: "post"
author: "jimdoescode"
title: "Terminal Quick Tip - Include options when debugging with lldb"
date: "2017-03-27 23:09:54"
tags: ["quicktip", "terminal", "c++", "debugger", "lldb"]
---

Took me a bit of googling to figure this out. I needed to include some options to a command
I was debugging. To do that with lldb it's a little unintuitive but you do this:

```sh
$ lldb -- ./binary --option1 --option2
```

This will start lldb with the options passed to the binary.
