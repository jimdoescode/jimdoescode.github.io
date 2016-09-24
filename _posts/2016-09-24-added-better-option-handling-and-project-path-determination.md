---
layout: "post"
author: "jimdoescode"
title: "Added better option handling and project path determination."
date: "2016-09-24 11:31:57"
tags: [jekyll-post-commit, ruby, git]
---

Using the [OptionParser](http://ruby-doc.org/stdlib-2.3.1/libdoc/optparse/rdoc/OptionParser.html) library I was able to add better option handling. I don't really like that you have to specify a flag infront of each option but I guess that is kind of a standard. This gave me the ability to add a new option that lets you specify where the blog post file should be written to. That means you don't have to manually move the file into your Jekyll posts folder.

An added benefit of the new option parsing is that now there is a help option to provide better insight into the commands usages.

As a part of this change set I also made the project root determination better. The [git library](https://github.com/schacon/ruby-git) I'm using needs to have the project root specified. Previously the command assumed that it always resided in the `.git/hooks` directory and would hop up two directories to get to the project root. That became problematic for dog fooding because I had to copy my changes to the hooks directory anytime I wanted to run the command. I added the ability to specify a command argument for the project directory but decided that making that optional instead would give more versatility. Now the project directory is assumed to be the directory that the command resides in. If a `.git` folder is in that path then the command will hop up until it is on the same level as the `.git` folder which is assumed to be the project root.
