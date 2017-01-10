---
layout: post
author: jim
title:  "Type conversion VS type assertion in go"
date:   2015-04-23 18:19:47
tags: [go, golang, type-conversion, type-assertion]
---

Just learned an interesting subtlety about Go. There are two ways to "cast" values. I put cast in quotes because one isn't really casting but it's just the general term I always use for saying "change on type to another one".

The first one I was pretty used to, [type conversion](http://golang.org/ref/spec#Conversions):
{% highlight go linenos %}
var x int = 5
var x64 int64 = int64(x)
{% endhighlight %}

The one I was unaware of is [type assertion](http://golang.org/ref/spec#Type_assertions):
{% highlight go linenos %}
var x interface{} = int64(5)
var x64 int64 = x.(int64)
{% endhighlight %}

So what's the difference? Well **conversion** should be used when you are dealing with a **type**, whether that be a constant or struct or whatever. **Assertion** is used when when you're dealing with an **interface**. Say for example a method returns an interface instead of a struct, there's still a value associated with the return but it has a generic type. You can use assertion to convert the return value to any type that implements that interface.

Pretty nifty!

> ######small aside
> I had to wrap the number 5 in an int64 when doing `var x interface{} = int64(5)`. Check out [this post]() to find out why 
