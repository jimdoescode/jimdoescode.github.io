---
layout: "post"
author: "jimdoescode"
title: "Feature flags or how to put the same eggs in the same baskets"
date: "2017-01-14 14:06:31"
tags: [go, golang, "feature flags"]
---

Feature flagging lets you develop new features in your application without making it visible to users until you're ready. 
Feature flags are also useful in A/B testing where you want to limit the users you show a new feature to in order to bug test
or gather information on how a particular feature will be recieved. I [made a Go  package](https://github.com/jimdoescode/feature) 
to make it super easy to flag features in your go code, but let's talk about how it works. 

Randomly flagging a feature can be useful if you just want to slowly roll out the new code and monitor it for any bugs. Doing random
feature flagging is very simple, you randomly generate a value, then see if that value meets a threshold. If so then the feature is
enabled otherwise the feature is disabled. 

Random flagging isn't terribly interesting, though it is supported in the package I wrote. The really interesting flagging involves 
making controlled samples where the same users see a feature enabled. This is where flagging gets tricky because you need to consistently
group users. Basically we need to reduce a user to a value between 0 and 1 then decide on a threshold also between 0 and 1. Any users
below the threshold have the feature enabled, any above don't. 

{% highlight go linenos %}
threshold := 0.5 // 50%
h := sha256.Sum256(user.userId)

var max uint64 = 1 << uint(len(h)) // set type to uint64 because 1 << 32 is too big for int or uint
val := 0

for _, b := range h {
    val = val << 1
    if b >= 128 {
        val += 1
    }
}

enabled := threshold > (float64(val) / float64(max))
{% endhighlight %}

The first step is hashing a value that's unique to each user, we're using `userId` in the above example. Next we figure out
what the maximum value of the hash could be by left shifting 1 by the size of the hash, for a sha256 this is `1 << 32`. Now
we loop over each byte of the hash and left shift val by 1 (this is the same as doing `val = val * 2`) if the byte is greater
than 128 (remember that bytes are uints between 0 and 255) add 1 to val. This loop reduces the hash to a single value between
0 and `max`. Finally we can divide our hash value `val` by `max` and see if that produces a value below our threshold. 

Because we create our hash from a userId we'll get the same result in `val` for a user which means we can consistently group
users by threshold. If we raise the threshold more users will be enabled, lowering the threshold will reduce the set of users.

The [feature flagging code](https://github.com/jimdoescode/feature) I wrote adds some more functionality around this, but the 
core is the algorithm I showed above.    
