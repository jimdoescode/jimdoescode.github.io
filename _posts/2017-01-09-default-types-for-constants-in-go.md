---
layout: "post"
author: "jimdoescode"
title: "Default types for constants in Go"
date: "2017-01-09 20:23:43"
tags: [go,golang,type-conversion,type-assertion,constants]
---

It was recently brought to my attention that I had an error in a [previous post]({% post_url 2015-04-23-type-conversion-vs-type-assertion-in-go %}). The error centered around setting a constant value of
5 to an interface. Here's what I was doing.

{% highlight go linenos %}
var x interface{} = 5
var x64 int64 = x.(int64)
{% endhighlight %}

The second line would result in the error `panic: interface conversion: interface is int, not int64`. The reason for this is that just the
number `5` by itself is a constant in Go. Constants in Go, even though they may appear to be untyped, are implicitly assigned a default type based on
their syntax. For example the constant `5.0` has an implicit type of `float64` while `5` has an implicit type of `int`. Hopefully you can see
what happened now.

When the code above runs, `5` is dumped into the interface variable `x` but the underlying type is `int`. Then when we try to assert that it's
`int64`, Go panics because there's a type mismatch.

To fix the issue we need to tell Go what the type of the constant 5 should be with type conversion.

{% highlight go linenos %}
var x interface{} = int64(5)
var x64 int64 = x.(int64)
{% endhighlight %}

Now everything runs as expected.
