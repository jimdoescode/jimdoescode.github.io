---
layout: "post"
author: "jimdoescode"
title: "Multi-line example output for godoc"
date: "2017-05-13 17:15:14"
tags: [go,golang,godoc]
---

Adding examples to godoc is pretty [trivial](https://godoc.org/github.com/fluhus/godoc-tricks#Examples)
but no one really talks about how multi-line output works. It's also trivial but I thought I'd give an example.

According to the [godoc tricks](https://godoc.org/github.com/fluhus/godoc-tricks#example-Examples--Output) to add the output
for an example method you just add `// Output: ...` at the end of your example code. The problem is that no one really
mentions how you could do nicely formatted multi-line output like for a json object or something.

It's pretty intuitive:
```golang
// Output: {
//     "key": "value"
// }
```

Here it is on the godoc website: https://godoc.org/github.com/jimdoescode/mobilepay#example-AndroidPayToken
