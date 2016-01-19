---
layout: post
author: jim
title:  "Letsencrypt and Dreamhost"
date:   2016-01-18 18:39:47
tags: [dreamhost, letsencrypt, https]
---

There's a great step by step guide for setting up letsencrypt certs on dreamhost [here][blog_post]! 
This post will parrot most of the stuff that it says with a few highlighted points that tripped me up.

I'm going to avoid all the letsencrypt setup and jump straight into running the thing. 

{% highlight bash %}
$ ./letsencrypt-auto certonly --manual -d example.com
{% endhighlight %}

Now that might pop up a thing about logging your IP address. If you aren't cool with that 
then you can't have a certificate, so make sure you say yes. 

Next the command will pause waiting for you to press enter. **DO NOT PRESS ENTER YET!** 
I made that mistake and had to run the command again. Before you can proceed, you need to 
create some folders and a file under the domain you are creating the certificate for. The 
folders are `/.well-known/acme-challenge` and the file is named some random hash of characters. 
The file content is some other hash of characters. 

An example path is `/.well-known/acme-challenge/RcapSBi_ZOlYnrByap1cRRrHln1lzKOIXwg2NowrZt5`

SSH into your dreamhost server and run

{% highlight bash %}
$ mkdir -p example.com/.well-known/acme-challenge
$ echo "RcapSBi_ZOlYnrByap1cRRrHln1lzKOIXwg2NowrZt5.lvgVdH1KypqK217AlBi9qE6ZYiusmdqZrzYNqOMGp3o" > .well-known/acme-challenge/RcapSBi_ZOlYnrByap1cRRrHln1lzKOIXwg2NowrZt5
{% endhighlight %}

Finally make sure that the file is reachable in your browser by visiting http://example.com/.well-known/acme-challenge/RcapSBi_ZOlYnrByap1cRRrHln1lzKOIXwg2NowrZt5

Now, after all that, you can hit enter. 

Once the command finishes you should see a congratulations under the "IMPORTANT NOTES" output. 

The next step is to visit the [dreamhost panel][panel] and go to Domains > Secure Hosting. 
Then click the "Add Secure Hosting" button and select your desired domain, in our case example.com.
(You don't need to add a unique IP)

Dreamhost should give you a success message saying that it's using a self-signed certificate.
Instead, we want it to use the certificate letsencrypt generated so click the edit button 
next to the domain and choose "Manual configuration".

Now you should see 4 text boxes labeled "Certificate Signing Request", "Certificate", 
"Private Key", and "Intermediate Certificate" respectively.

 - **Certificate Signing Request**: Delete everything from this box.
 - **Certificate**: Copy the text from `cat /etc/letsencrypt/live/example.com/cert.pem | pbcopy` into this box.
 - **Private Key**: Run the following command `openssl rsa -in /etc/letsencrypt/live/example.com/privkey.pem` and copy the outputted result into this box.
 - **Intermediate Certificate**: Copy the text from `cat /etc/letsencrypt/live/example.com/chain.pem | pbcopy` into this box.

After that, click "Save changes now!" and you're all set. 

[blog_post]: https://lyncd.com/2015/12/letsencrypt-dreamhost-howto/
[panel]: https://panel.dreamhost.com