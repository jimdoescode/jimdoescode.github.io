---
layout: "post"
author: "jimdoescode"
title: "How I resolved a weird Yarn issue"
date: "2020-03-26 13:50:14"
tags: [yarn,npm,javascript]
---

After an OSX update I attempted to run `yarn serve` and was greeted with this error: ```error An unexpected error occurred: "Failed to replace env in config: ${XDG_CONFIG_HOME}".```

Which was very unexpected since I was running the same command a day earlier before I had updated.

If you can't tell from the error message I use the [XDG Base Directory Specification](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html) to keep my home directory as clean as I can. Doing so also gives me something to gripe about to other developers because so many ignore the specification and dump their dot files into my home directory. Trying to maintain XDG means constantly making sure that the right environment variables are set up. Also OSX updates have a tendency to reset everything. So I double checked and everything was set up appropriately. 

After some googling I found various stackoverflow posts talking about a similar error but with the env variable `${NPM_TOKEN}` which didn't match my error. I tried a few of the suggestions anyway but they didn't seem to help.

Finally I came across a GitHub issue on the npm repo about XDG and in particular [this comment](https://github.com/npm/npm/issues/6675#issuecomment-75163496). I figured what the heck and deleted the curly braces around the variables in my npmrc (because of XDG this resides in `~/.config/npm/npmrc`). I saved the file and attempted to run yarn. 

Sure enough it worked. I have no idea why that caused an issue. I'm hesitant to say that those were the actual culprit but 🤷‍♂
