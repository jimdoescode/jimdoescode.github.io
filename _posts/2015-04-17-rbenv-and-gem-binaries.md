---
layout: post
author: jim
title:  "rbenv and gem binaries"
date:   2015-04-17 17:09:47
tags: [rbenv, ruby, gem] 
---
Figured I'd give [GitHub Pages][github-pages] a shot which means I need to set up [Jekyll][jekyll]. That means I need to install Ruby, as the version install on my Mac is pretty out of date. After some quick googling it appears that managing ruby versions with RVM is out as the version manager and [rbenv] is in. That brought about some minor confusion.

A quick Brew install took care of that:
{% highlight sh %}
$ brew install rbenv
{% endhighlight %}

I then used [rbenv] to install the current stable version of ruby (2.2.2 currently).
{% highlight sh %}
$ rbenv install 2.2.2
{% endhighlight %}

So far everything is running smoothly. Now I need to install bundler which is where I had some confusion. I made the Gemfile and installed it fine but when I tried to do the bundle install
{% highlight sh %}
$ bundle install
{% endhighlight %}
The bundle command wasn't found. I quickly realized that I don't have any gem bin directories in my path and immediately lamented that I have no way to know even where they were with rbenv. I did manage to track the bundle binary down but it was in a very convoluted place. Not ideal for just running off the cuff. After a little head scratching and google searches I figured out the solution. I need to regenerate pathes each time I do a gem install. 

Fortunately rbenv gives a nice way to do this: 
{% highlight sh %}
$ rbenv rehash
{% endhighlight %}

After doing this the bundle command was accessable as a normal binary. Doing a rehash was further re-enforced after installing jekyll. Once again it wasn't in the path, but a quick rehash did the trick. 

I need to remember that from now on if I install a gem with a binary I need to rehash. 

[jekyll]: http://jekyllrb.com
[github-pages]: https://pages.github.com/
[rbenv]: https://github.com/sstephenson/rbenv
