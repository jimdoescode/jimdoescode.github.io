---
layout: "post"
author: "jimdoescode"
title: "Setting up the pass password manager"
date: "2017-02-18 15:41:04"
tags: [pass, "password manager", bash]
---

After years of having to remember several passwords and decide which I would use when signing up for a
new account. I've finally decided to make the switch to password manager. From now on I'll be using
[pass](https://www.passwordstore.org/). Let's talk about how I set it up. (WARNING lot's of terminal commands ahead)

First thing's first, we need to get it installed. Luckily if you have `brew` it's as simple as
```sh
$ brew install pass
```

That will install pass with all of its dependencies. Now this is where the docs get a little vague,
because while they mention that pass uses gpg they don't tell you how to set up gpg because I didn't
already have a gpg key pair created so there was no way for me to encrypt all the passwords I would be
dumping into pass. So let's go through the steps of setting up our gpg keys. First we'll run the command
to generate the keys.
```sh
$ gpg2 --gen-key
```
NOTE: I'm using gpg2 here. *Not gpg*

Running gen-key will then prompt you as follows
```sh
Please select what kind of key you want:
(1) RSA and RSA (default)
(2) DSA and Elgamal
(3) DSA (sign only)
(4) RSA (sign only)
Your selection?
```
I selected RSA/RSA because we need to encrypt our password file.

Next you'll be prompted for the size of the key.
```sh
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048)
```
I chose the max (4096) because I am kind of a tinfoil hat guy and the larger your keys the tougher they
are to crack. Also most of the stuff we'll be encrypting will be staying on my local machine so I don't
need to worry too much about large payloads being passed around. Of course the default (2048) is fine too.

Now you need to decide how long this key should live.
```sh
Please specify how long the key should be valid.
0 = key does not expire
d = key expires in n days
w = key expires in n weeks
m = key expires in n months
y = key expires in n years
key is valid for? (0)
```
Choosing (0) here is probably what you want since we don't want to have our passwords suddenly become 
inaccessible if the key expires.

Lastly you'll be prompted to verify your choices and enter your name and email address. Once you've done
all that, hit 'o' to confirm everything and enter a password for your secret key. 

Make sure you use a strong knowable password here. This will be what you have to enter to get your saved 
passwords out of pass so make sure it's something you'll remember.

Once gpg generates your keys it'll output something like this
```sh
pub  4096R/1B2AFA1C 2017-02-18
     Key fingerprint = 117C FE83 22EA B843 3E86  6486 4320 545E 1B2A FA1C
uid      [ultimate] John Q. Doe <jqdoe@example.com>
sub  4096R/CEA4B22E 2017-02-18
```
Note the second string after the slash of the first line (1B2AFA1C). We need this value to tell pass which
gpg key we want to use for encrypting our passwords.

Now our keys are generated let's set up pass. First we'll initialize pass.
```sh
$ pass init "1B2AFA1C"
```
Again note, the value we used in the init (1B2AFA1C) was what we got from that 'pub' line in the gpg set up.

Congratulations! You now have a working password manager. If you want to generate a password simply type
```sh
$ pass generate example.com 20
```
Which will produce a 20 character password to use.

The problem is that it only keeps you passwords on the computer you just set pass up on. What if you
need to use those passwords on other computers? This is where setting up pass with git comes in handy.
Run the command
```sh
$ pass git init
```
Which will create a new git repo in the directory where pass stores your encrypted passwords. Now we can
do things like add a remote and push our encrypted passwords to github or bitbucket. I personally prefer
bitbucket since it has free *private* repositories but these things are encrypted so it doesn't really
matter.

Alright, we've got pass working on one machine and it can export the encrypted passwords to a remote git
repo. But the other machines that pull from that repo won't be able to decrypt the passwords. Back to gpg.
```sh
$ gpg2 --armor --export jqdoe@example.com > public.key
$ gpg2 --armor --export-secret-keys jqdoe@example.com > private.key
```
Then copy those keys over to the other computers you want to use and use gpg to import them.
```sh
$ gpg2 --import public.key
$ gpg2 --import private.key
```

After that make sure you delete those key files and now you'll have pass working where ever you need it!

Oh and don't forget to push your saved passwords to the remote repo.
```sh
$ pass git push
```
