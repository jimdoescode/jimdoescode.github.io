---
layout: "post"
author: "jimdoescode"
title: "A simple systemd keep-alive script"
date: "2017-08-03 22:15:14"
tags: [systemd,bash,linux]
---

After spending a few hours trying to get systemd to just keep a simple binary running if it halts, here's what I've learned.

Systemd is a linux service manager. It handles running lots of processes and I wanted it to just run a simple slackbot binary for me. Basically if the bot goes down I want systemd to restart it for me. Since this is a silly thing and doesn't really need lots of permissions or anything I want it to run for my user on the server and not as root.

First off any user level systemd scripts (called "units") should be put in `~/.config/systemd/user/`. For the slackbot I added a unit called "slackbot.service".

These units are just text files, for a simple keep-alive script here's what mine looked like:
```
[Unit]
Description=Manage SlackBot

[Service]
ExecStart=/path/to/slackbot/binary
Restart=always
RestartSec=3

[Install]
WantedBy=default.target
``` 

Then to load and start the service run
```sh
$ systemctl --user start slackbot
```

If you'd like to check that it was started appropriately
```sh
$ systemctl --user status slackbot
```

To have it run after a reboot you need to enable user lingering. To do this run the following command:
```sh
# loginctl enable-linger <username>
```

Then you need to enable the unit using systemd.
```sh
$ systemctl --user enable slackbot
```
That should output something like
```
Created symlink from ~/.config/systemd/user/default.target.wants/slackbot.service to ~/.config/systemd/user/slackbot.service.
```

Memory dump complete. Hopefully this helps.
