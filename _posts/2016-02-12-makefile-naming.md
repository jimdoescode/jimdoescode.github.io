---
layout: post
author: jim
title:  "Makefile naming is important/annoying"
date:   2016-02-12 00:31:47
tags: [make, makefile]
---

I just spent the better part of an hour trying to figure out why my makefile wasn't working.

Here is my dead simple makefile
```
.PHONY client
client:
	echo "Test test"
```

Yet when I ran `$ make client` the output was always: `make: Nothing to be done for 'client'.`

But why? 

Turns out it had nothing to do with the contents of the makefile, it's the file name. I 
named my makefile "MakeFile" which make can't read by default. 

**I needed to name my file "makefile".**

If I wanted to leave it named "MakeFile" then I would need to tell make using the `-f` 
argument. Who has the time for that though.

Bottom line; keep your makefiles lowercase.