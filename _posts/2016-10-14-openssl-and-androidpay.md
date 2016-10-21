---
layout: "post"
author: "jim"
title: "OpenSSL and AndroidPay"
date: "2016-10-14 18:12:01"
tags: ["C", "OpenSSL", "AndroidPay"]
---

This is my first deep dive into OpenSSL and boy oh boy is it complicated and not well documented. Unfortunately that means I can't explain a ton of the reasoning
behind the decisions I made in my code. It's mostly like that because that's what I found that works.

AndroidPay sends a JSON payload to a server. In that payload are 3 things the *tag*, the *ephemeralPublicKey*, and the *encryptedMessage* (the data). On the server side we
need the private key which is the companion to the public key that AndroidPay used to encrypt everything. With those 4 things we can then decrypt the *encryptedMessage*.

Step 1
------

We need to get the private key that resides on the server into a format that OpenSSL can use. It, along with the *ephemeralPublicKey*, will be used to generate a shared key
that can decrypt the message. The Java example Google provides has a private key which is created with the [PKCS8](https://tools.ietf.org/html/rfc5208) format. Luckily
OpenSSL has a handy container for that called `PKCS8_PRIV_KEY_INFO` there are also methods that allow us to use `BIO` which is OpenSSL's IO wrapper to convert the base64 encoded key into
the `PKCS8_PRIV_KEY_INFO` format.

```c
PKCS8_PRIV_KEY_INFO *p8;
BIO *bio_64 = BIO_new(BIO_f_base64());
BIO *bio_mem = BIO_new(BIO_s_mem());

// Read merch_privkey_b64 into BIO
BIO_set_flags(bio_64, BIO_FLAGS_BASE64_NO_NL);
BIO_write(bio_mem, merch_privkey_b64, (int)merch_privkey_b64_len);
bio_mem = BIO_push(bio_64, bio_mem);

// Read the key from the BIO
p8 = d2i_PKCS8_PRIV_KEY_INFO_bio(bio_mem, NULL);

BIO_free_all(bio_64);
```

Once we have the private key into the `PKCS8_PRIV_KEY_INFO` container there's a nice method that will convert it into an `EVP_PKEY` which is what we want to use for our public
and private key pairs.

```c
EVP_PKEY * private_key = EVP_PKCS82PKEY(p8);
PKCS8_PRIV_KEY_INFO_free(p8);
```

After we free up our `p8` variable, we have our private key ready to go.

Step 2
------

Now we need to get the *ephemeralPublicKey* into that `EVP_PKEY` format. This key is a point on a special elliptic curve named NIST P-256 or in OpenSSL the curve is known as prime256v1.
The *ephemeralPublicKey* is base64 encoded so you need to decode it first (not shown). Then, with the decoded octet string we can set the point using an o2i method which
means octet to something (maybe input??) I don't really know what the 'i' stands for but it does set the point correctly.

```c
EC_KEY *pubkey = EC_KEY_new_by_curve_name(NID_X9_62_prime256v1);
if (o2i_ECPublicKey(&pubkey, ephemeral_pubkey_octet, ephemeral_pubkey_octet_len) == NULL) {
    ERROR;
}
```

This method deals with a different type than the `EVP_PKEY` type we saw in step 1, so we'll need to convert this `EC_KEY` type to `EVP_PKEY` to get our public and
private keys to play nice together. From what I understand `EC_KEY` is a lower level abstraction in OpenSSL and it's advisable to only use them if you really know what you're doing.
I don't, so we need to convert it to the higher level `EVP_PKEY`. I believe `EVP_PKEY`s actually use `EC_KEY`s under the hood and there's actually a nice method to set an `EC_KEY`
in an `EVP_PKEY`.

```c
EVP_PKEY * public_key = EVP_PKEY_new();
if (!EVP_PKEY_set1_EC_KEY(public_key, pubkey)) {
    ERROR;
}
```

Now we have a public key and a private key, and both are in the same format.

Step 3
------

We now need to use our public and private keys to generate a shared secret key. This shared secret and the *ephemeralPublicKey* will get concatenated into a single buffer that we
can use to generate our decryption key. In order to create the shared secret we need to use some `EVP_PKEY` methods. First we'll create a context container to hold both keys
and then derive the shared secret using that context.

```c
EVP_PKEY_CTX *kctx = EVP_PKEY_CTX_new(state->private_key, NULL);
size_t shared_secret_len;

if (!EVP_PKEY_derive_init(kctx) ||
    !EVP_PKEY_derive_set_peer(kctx, public_key) ||
    !EVP_PKEY_derive(kctx, NULL, &shared_secret_len)) {
    ERROR;
}
```

That call to `EVP_PKEY_derive` above will tell us how large the shared secret buffer needs to be. We then allocate a new buffer to that size and insert the shared secret.

```c
unsigned char *shared_secret = OPENSSL_malloc(shared_secret_len);
if (!EVP_PKEY_derive(kctx, shared_secret, &shared_secret_len)) {
    ERROR;
}
EVP_PKEY_CTX_free(kctx);
```

As mentioned above the shared secret is concatenated with the *ephemeralPublicKey*. This makes up what's called the "input keying material" for the HKDF algorithm that's used to generate our
mac and decryption keys.

```c
size_t input_keying_material_len = shared_secret_len + ephemeral_pubkey_octet_len;
unsigned char *input_keying_material = OPENSSL_malloc(input_keying_material_len);
memcpy(input_keying_material, ephemeral_pubkey_octet, ephemeral_pubkey_octet_len);
memcpy(input_keying_material + ephemeral_pubkey_octet_len, shared_secret, shared_secret_len);
OPENSSL_free(shared_secret);
```

There is some slight pointer arithmetic going on to perform the concatenation. I originally tried to use `strcat` but these are raw byte buffers not real strings so it didn't result in the
desired value.

After this step we should have all the things necessary to generate our AndroidPay decryption key.

Step 4
------

In order to generate the encryption key we need to use the [HKDF algorithm](https://tools.ietf.org/html/rfc5869) with a SHA256 hash. This means we'll need to use the SHA256 message digest (MD)
constant provided by OpenSSL. Since it's a constant we don't need to worry about cleaning it up at the end which is nice. We'll then take that constant and run it through an HMAC 
as per the HKDF algorithm's extract step. [According to AndroidPay](https://developers.google.com/android-pay/integration/payment-token-cryptography#decrypting-the-payment-token) 
we specify a NULL salt and use our input keying material that was generated previously in the HMAC.

```c
const EVP_MD *sha256 = EVP_sha256();
unsigned char *prc;
unsigned int prc_len = EVP_MD_size(sha256);

if (HMAC(sha256, NULL, 0, input_keying_material, input_keying_material_len, prc, &prc_len) == NULL) {
    ERROR;
}
```

This gives us some pseudo-random characters that need to be put into an `EVP_PKEY` container. We then feed `EVP_PKEY` through the HKDF expand step. As per 
the [AndroidPay docs](https://developers.google.com/android-pay/integration/payment-token-cryptography#decrypting-the-payment-token) we use the ASCII 
character string "Android" as the info component.

One thing to keep in mind is that we're dealing with a SHA256 hash for HKDF. This results in only a single iteration through the HKDF expansion 
step which greatly simplifies our code.

```c
unsigned char T[EVP_MD_size(sha256)];
size_t T_len;

EVP_PKEY *prk = EVP_PKEY_new_mac_key(EVP_PKEY_HMAC, NULL, prc, prc_len); //pseudorandom key
EVP_MD_CTX *hmac = EVP_MD_CTX_new();

// Once we've generated the pseudo random key we run
// the HKDF expand step passing "Android" in for info.
unsigned char byte = 1;
if (!EVP_DigestSignInit(hmac, NULL, sha256, NULL, prk) ||
    !EVP_DigestSignUpdate(hmac, "Android", 7) ||
    !EVP_DigestSignUpdate(hmac, &byte, 1) ||
    !EVP_DigestSignFinal(hmac, T, &T_len)) {
    ERROR;
}

EVP_MD_CTX_free(hmac);
EVP_PKEY_free(prk);
```

After a single iteration through the HKDF expansion we should have a 256 bit filled buffer. The first 128 bit, or 16 byte, half is the symmetric encryption key, the second half 
is the mac key. We'll use the mac key to validate the *tag* value and make sure that we are decrypting something that actually came from
our AndroidPay client. The symmetric encryption key is used to decrypt the *encryptedMessage*. 

```c
unsigned char *encryption_key = OPENSSL_malloc(16);
unsigned char *mac_key = OPENSSL_malloc(16);

memcpy(state->encryption_key, T, 16);
memcpy(state->mac_key, T + 16, 16);
```

We're almost ready to decrypt the message now.

Step 5
------

Before we do the decryption we want to make sure that the *tag* value that was sent with the AndroidPay payload is valid. That will indicate that this is a valid request and we are
clear to decrypt it. The *tag* value should be the result of an HMAC with a SHA256 hash of the previously generated mac key and the *encryptedMessage*. We'll once again use the `EVP_MD` 
for the SHA256 message digest. Make sure the *encryptedMessage* is base64 decoded (not shown).

```c
const EVP_MD *sha256 = EVP_sha256();
unsigned char tag_comparison[EVP_MD_size(sha256)];
unsigned int tag_comparison_len;

if (HMAC(sha256, mac_key, 16, encrypted_message, encrypted_message_len, (unsigned char *)&tag_comparison, &tag_comparison_len) == NULL) {
    ERROR;
}
```

Once we have the tag_comparison we then have to compare it but we need to be mindful of timing attacks here. Luckily OpenSSL provides a memcmp method that guards against timing
attacks.

```c
if (CRYPTO_memcmp(tag, &tag_comparison, tag_len)) {
    INVALID;
}
```

We've now validated the AndroidPay client's data so we're clear to decrypt.

Step 6
------

Decrypting the *encryptedMessage* is pretty straight forward if you've made it to this point. We just need to make sure the *encryptedMessage* is base64 decoded (not shown) then run it through
the appropriate cipher. According to [the documentation](https://developers.google.com/android-pay/integration/payment-token-cryptography#decrypting-the-payment-token) the cipher is AES128 CTR 
mode with a zero IV. I don't know what that means but OpenSSL has functions and parameters that share that name so we can use those.

```c
EVP_CIPHER_CTX *decode_ctx = EVP_CIPHER_CTX_new();
unsigned char * unencrypted_message = OPENSSL_malloc(encrypted_message_b64_len);
size_t unencrypted_len = 0, final_unencrypted_len = 0;

if (!EVP_CIPHER_CTX_init(decode_ctx) ||
    !EVP_DecryptInit_ex(decode_ctx, EVP_aes_128_ctr(), NULL, encryption_key, 0) ||
    !EVP_DecryptUpdate(decode_ctx, unencrypted_message, (int *)&unencrypted_len, encrypted_message, (int)encrypted_message_len) ||
    !EVP_DecryptFinal_ex(decode_ctx, unencrypted_message + unencrypted_len, (int *)&final_unencrypted_len)) {
    ERROR;
}

unencrypted_len += final_unencrypted_len;
unencrypted_message[unencrypted_len] = '\0';
```

A few things we need to be mindful of, the unencrypted lengths used in `EVP_DecryptUpdate` and `EVP_DecryptFinal_ex` need to be set to something, otherwise it causes errors. Also 
you can't just reuse the same length variable for both those methods. `EVP_DecryptFinal_ex` isn't additive so you need to make sure you're moving the pointer and using a new length 
variable. Finally, the base64 length we use to allocate the unencrypted_message buffer is always going to be larger than the unencrypted message data itself so we need to make sure that we 
properly terminate the data at the correct spot. Otherwise you could get garbage at the end of the unencrypted message buffer.

Conclusion
----------

I'm not entirely sure why some of these things are the way they are but once it's all together the decryption process seems pretty logical. This was my first dive into the OpenSSL
codebase and I'm not completely convinced that I got everything correct, but, it works and hopefully doesn't have any major issues.
