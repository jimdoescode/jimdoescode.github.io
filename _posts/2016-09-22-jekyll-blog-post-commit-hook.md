---
layout: post
author: jim
title: "Jekyll Blog Post Commit Hook"
date: 2016-09-22 00:27:58
tags: [ruby, git, jekyll, makefile]
---

I'm wrapping up work on a post commit hook that will create a Jekyll markdown file based off a git commit message. I'm not really sure if there is much utility in this app but it was a fun way to dive back into ruby code. You can find all the code [here](https://github.com/jimdoescode/jekyll-blog-post-commit-hook)

Installation
------------

Download the source and run:
```sh
$ make install
```

This will make sure that the necessary gems are installed and then will prompt you for the location of the project you want to use the hook with. It then copies the hook to the `.git/hooks` directory of that project.

Usage
-----

Once you have the hook installed you can make commits as you would normally and any commit message that *ends* with `{BLOG}` will have a markdown file generated for it in the project directory. You can optionally add front matter to your posts by specifying it in the BLOG tag like this `{BLOG tags=[foo, bar]&layout=test}`.

I hope you have some fun with this!
