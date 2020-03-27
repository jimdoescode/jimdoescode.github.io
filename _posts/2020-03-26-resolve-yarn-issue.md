---
layout: "post"
author: "jimdoescode"
title: "How I resolved a weird Yarn error"
date: "2020-03-26 13:50:14"
tags: [yarn,npm,javascript]
---

After an OSX update I attempted to run `yarn serve` and was greeted with an unexpected error. 
```
error An unexpected error occurred: "Failed to replace env in config: ${XDG_CONFIG_HOME}".
```

I was running the same command a day earlier without incident and now after updating it was blowing up.

If you can't tell from the error message I use the [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html) to keep my home directory as clean as I can. Doing so also gives me something to gripe about to other developers because so many ignore the specification and dump their dot files into my home directory. 

Trying to maintain XDG means constantly making sure that the right environment variables are set up. OSX updates also have a tendency to reset everything. So that was my first thought of a problem. But, after checking, all my environment variables seemed were set up correctly. Next step is to do an internet search.

After some googling (in my case duckduckgoing) I found various stackoverflow posts talking about a similar error but with the env variable `${NPM_TOKEN}` which didn't match my error. I tried a few of the suggestions anyway but they didn't seem to help.

Finally I came across a GitHub issue on the npm repo about XDG and in particular [this comment](https://github.com/npm/npm/issues/6675#issuecomment-75163496). I figured what the heck and deleted the curly braces around the variables in my local npmrc (because of XDG this resides in `~/.config/npm/npmrc`). I saved the file and attempted to run yarn. 

It worked! 

I have no idea why that caused an issue and am hesitant to say that curly braces were the actual culprit but 🤷‍♂
