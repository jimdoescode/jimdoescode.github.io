---
layout: "post"
author: "jimdoescode"
title: "Validate a STUN binding request"
date: "2018-03-31 13:50:14"
tags: [webrtc,sdp,ice,stun]
---

In WebRTC passing the ice-ufrag and ice-pwd attributes to the client in an SDP answer will 
result in the client sending those values back as part of the STUN binding request. 

For example an answer SDP like:
```
v=0
...
a=ice-pwd:fpllngzieyoh43e0133ols
a=ice-ufrag:6k1hh2gd
...
```

Will result in a STUN binding request that looks similar to:
```
00000000  00 01 00 50 21 12 a4 42  a5 89 a1 52 2c f6 d6 1f  |...P!..B...R,...|
00000010  12 f2 55 69 00 06 00 11  36 6b 31 68 68 32 67 64  |..Ui....6k1hh2gd|
00000020  3a 38 62 63 31 64 62 61  34 00 00 00 00 25 00 00  |:8bc1dba4....%..|
00000030  00 24 00 04 6e 7f 00 ff  80 2a 00 08 e4 53 2a 52  |.$..n....*...S*R|
00000040  ba 06 bd e9 00 08 00 14  bd e0 f2 7b f7 d8 71 70  |...........{..qp|
00000050  67 44 86 9a a2 d9 a3 a0  27 d6 a9 d1 80 28 00 04  |gD......'....(..|
00000060  04 36 14 1e 
```

## Finding the USERNAME

According to the STUN protocol the USERNAME is defined by the bytes `0x0006` followed by a 
two byte pad `0x00` and the length of the user name (`0x11` or 17). Everything in STUN is in TLV format (type, 
length, value). So the bytes of the USERNAME attribute in this STUN binding request are
```
0x00 0x06                               # This is a USERNAME 
0x00                                    # Padding
0x11                                    # The length is 17 bytes long
0x36 0x6b 0x31 0x68 0x68 0x32 0x67 0x64 # User name 6k1hh2gd
0x3a                                    # Colon separator
0x38 0x62 0x63 0x31 0x64 0x62 0x61 0x34 # Nonce 8bc1dba4
```

The nonce after the colon (8bc1dba4) is to make sure that the MESSAGE-INTEGRITY attribute is 
unique on each request. It also makes the ice-pwd harder to determine.

## Finding the MESSAGE-INTEGRITY

To verify no one has tampered with this STUN binding request we need to attempt to compute 
the same MESSAGE-INTEGRITY hash that's in the request. The MESSAGE-INTEGRITY attribute is 
identified by the bytes `0x0008` followed by a two byte pad `0x00` and the length of the hash 
(`0x14` or 20). So the MESSAGE-INTEGRITY attribute of our request is
```
0x00 0x08                               # This is a MESSAGE-INTEGRITY hash 
0x00                                    # Padding
0x14                                    # The length is 20 bytes long
0xbd 0xe0 0xf2 0x7b 0xf7 0xd8 0x71 0x70 # This is the hash
0x67 0x44 0x86 0x9a 0xa2 0xd9 0xa3 0xa0 
0x27 0xd6 0xa9 0xd1
```

## Verifying the request

We calculate this by HMAC-SHA1ing all the bytes up to the MESSAGE-INTEGITY attribute with 
the ice-pwd that was we sent to the client in the SDP answer (fpllngzieyoh43e0133ols).
```
HMAC-SHA1(
    0x00 0x01 0x00 0x50 0x21 0x12 0xa4 0x42  
    0xa5 0x89 0xa1 0x52 0x2c 0xf6 0xd6 0x1f
    0x12 0xf2 0x55 0x69 0x00 0x06 0x00 0x11  
    0x36 0x6b 0x31 0x68 0x68 0x32 0x67 0x64
    0x3a 0x38 0x62 0x63 0x31 0x64 0x62 0x61  
    0x34 0x00 0x00 0x00 0x00 0x25 0x00 0x00
    0x00 0x24 0x00 0x04 0x6e 0x7f 0x00 0xff  
    0x80 0x2a 0x00 0x08 0xe4 0x53 0x2a 0x52
    0xba 0x06 0xbd 0xe9,
    fpllngzieyoh43e0133ols
)
```

Doing that results in the hash
```
0xbd 0xe0 0xf2 0x7b 0xf7 0xd8 0x71 0x70 
0x67 0x44 0x86 0x9a 0xa2 0xd9 0xa3 0xa0 
0x27 0xd6 0xa9 0xd1 
```

Which is equivalent to the request's MESSAGE-INTEGRITY hash. 👍