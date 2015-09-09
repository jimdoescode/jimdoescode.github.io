---
layout: post
author: jim
title:  "I'm dumb or at least naive about encryption"
date:   2015-04-19 18:19:47
tags: [go, golang, encryption, public-key-encryption]
---

I just completed my first [challenge in Go][challenge]! It turned out better than I thought it would and I'm pretty proud of it. I had a working version or, at least I thought I did, a week or so before the end of the challenge. There were just two unit tests (provided by the challenge author) that were failing and it wasn't clear to me why. Only after reading up on public key encryption did I realize that it's entirely one way. 

**The public key encrypts the private key decrypts.** 

It took me almost the entire challenge to figure that out! My original solution had the client generating both public and private keys then sharing BOTH (again I'm dumb) with the server, then both would use the same keys to encrypt and decrypt. When in reality the client should generate its public and private keys as should the server, then client and server should exchange public keys only. Luckily Go was easy enough to work with it took me about 15 minutes to enact the change and another 10 to test it. After that the unit tests all worked and I felt comfortable enough to submit. All in all it was a great experience and why I love programming. If you're interested check out [http://golang-challenge.com/][golang-challenge] I'll be participating in many of these going forward.

[challenge]: http://golang-challenge.com/go-challenge2/
[golang-challenge]: http://golang-challenge.com/
