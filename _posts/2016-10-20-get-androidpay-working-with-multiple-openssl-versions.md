---
layout: "post"
author: "jim"
title: "Get AndroidPay working with multiple OpenSSL versions"
date: "2016-10-20 21:48:04"
tags: [c,openssl,androidpay]
---

In a [previous post]({% post_url 2016-10-14-openssl-and-androidpay %}) we went over how to 
use the OpenSSL API in C to decrypt AndroidPay data. There was something I didn't realize 
in my implementation. I was using version 1.1.0 over the OpenSSL API and there are a lot of 
folks still running versions earlier than that.

Unfortunately there are some API differences between 1.1.0 and earlier versions and even 
more unfortunately, we do hit a couple of them.

One of the biggest differences is in creating a new `EVP_MD_CTX` which we do when
generating the symmetric encryption key and the mac key. In OpenSSL versions >= 1.1.0 the
function to create a new `EVP_MD_CTX` is `EVP_MD_CTX_new()` this lines up with many of the 
other new instance creation functions in OpenSSL. However prior to 1.1.0 the way to get a 
new `EVP_MD_CTX` instance is with a call to `EVP_MD_CTX_create()`.

We need to reconcile this and ideally we need a solution that works in both instances. This 
is where we can use a preprocessor conditional.

```c
EVP_MD_CTX *hmac;

#if OPENSSL_VERSION_NUMBER >= 0x10100000L 
    hmac = EVP_MD_CTX_new();
#else
    hmac = EVP_MD_CTX_create();
#endif
```

This code snippet will only surface the method call that corresponds to the correct OpenSSL 
API. Now since there are different `EVP_MD_CTX` creation functions of course that means 
there are different functions to clean up `EVP_MD_CTX` instances so we need to use the same 
preprocessor conditionals.

```c
#if OPENSSL_VERSION_NUMBER >= 0x10100000L 
    EVP_MD_CTX_free(hmac);
#else
    EVP_MD_CTX_destroy(hmac);
#endif
```

The next incompatibility deals with how the `HMAC()` function works. In versions >= 1.1.0 
we could use `NULL` and `0` if we didn't want to specify a salt for the HKDF algorithm.

```c
const EVP_MD *sha256 = EVP_sha256();
unsigned char *prc;
unsigned int prc_len = EVP_MD_size(sha256);

if (HMAC(sha256, NULL, 0, input_keying_material, input_keying_material_len, prc, &prc_len) == NULL) {
    ERROR;
}
```

In prior version that call results in an error because a key (what we call a salt in HKDF) 
has to be specified and the length has to be greater than zero. Luckily the
[AndroidPay documentation](https://developers.google.com/android-pay/integration/payment-token-cryptography#decrypting-the-payment-token) 
covers this by telling us that not providing a salt is equivalent to 32 zeroed bytes. So
we'll create a key that's 32 zeroed bytes and make the length of our key 32.

```c
const EVP_MD *sha256 = EVP_sha256();
unsigned char *prc;
unsigned int prc_len = EVP_MD_size(sha256);
const unsigned char salt[33] = "\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0";
const int salt_len = 32;

if (HMAC(sha256, salt, salt_len, input_keying_material, input_keying_material_len, prc, &prc_len) == NULL) {
    ERROR;
}
```

Remember that `0` is different from `\0` we basically want to use `NULL` values and `0` 
isn't a `NULL` value. Doing this will work in all versions of OpenSSL so we don't need any 
preprocessor conditionals.

Those two changes are really all that's needed to update the code from the previous post so
that it works with older versions of OpenSSL. I should note that the lowest version I tested
with is 1.0.1. I can't guarantee that anything older than that will work.
