---
layout: default
permalink: /pa2/crypto-ref
title:  Cryptography Reference
description: Reference guide on using common.c/h
parent: Programming Assignment 2
grand_parent: Programming Assignment
nav_order:  10
nav_exclude:  true
---


* TOC
{:toc}

**50.005 Computer System Engineering**
<br>
Information Systems Technology and Design
<br>
Singapore University of Technology and Design
<br>
**Natalie Agus (Summer 2025)**

# Cryptography Reference

**50.005 Computer System Engineering**
Information Systems Technology and Design
Singapore University of Technology and Design
**Natalie Agus (Summer 2026)**


This document is a **companion** to Programming Assignment 2. It is split into two parts:

- **[Part 1: Cryptographic Concepts](#part-1-cryptographic-concepts)**: what each operation does, how it works, and why it is used.
- **[Part 2: common.c / common.h API Reference](#part-2-commonc--commonh-api-reference)**: every function and constant in the shared library, with usage notes and cross-references to Part 1.

In both the Cryptography Lab and PA2, you are provided with `common.c` and `common.h` which wrap raw OpenSSL calls. You do not need to implement or rewrite any of these functions, and they are **not tested directly** in exams. Understanding them will help you reason about the protocol, debug your code, and answer demo questions.

# Part 1: Cryptographic Concepts

## Symmetric Encryption: AES-128-CBC

**Used in:** Lab Part 1 (text encryption), PA2 Task 3 (CP2 file encryption)

**API:** [`session_encrypt`](#session_encrypt), [`session_decrypt`](#session_decrypt), [`generate_session_key`](#generate_session_key)

```c
unsigned char key[32];
generate_session_key(key);                                        /* fill with random bytes */
unsigned char *token  = session_encrypt(key, plaintext, len, &out_len);
unsigned char *result = session_decrypt(key, token, out_len, &dec_len);
```

### What the library actually does

`session_encrypt` performs the following steps:

```
ENCRYPT:
  1. Split the 32-byte key:
       hmac_key = key[0..15]      (16 bytes for HMAC-SHA256)
       aes_key  = key[16..31]     (16 bytes for AES-128-CBC)

  2. Generate a random IV:
       iv = 16 random bytes       (different every call)

  3. Pad plaintext with PKCS7:
       If plaintext is 13 bytes → pad with 3 bytes of value 0x03
       If plaintext is 16 bytes → pad with 16 bytes of value 0x10
       (always pads, even if already block-aligned)

  4. Encrypt with AES-128-CBC:
       ciphertext = AES-CBC(aes_key, iv, padded_plaintext)

  5. Compute HMAC-SHA256 over (IV || ciphertext):
       hmac = HMAC-SHA256(hmac_key, iv || ciphertext)

  6. Output token:
       token = iv (16 bytes) || ciphertext (variable) || hmac (32 bytes)

DECRYPT:
  1. Split the key (same as above)
  2. Extract from token: iv, ciphertext, received_hmac
  3. Recompute: expected_hmac = HMAC-SHA256(hmac_key, iv || ciphertext)
  4. Compare using constant-time comparison:
       if received_hmac != expected_hmac → REJECT (tampered!)
  5. Decrypt: plaintext_padded = AES-CBC-decrypt(aes_key, iv, ciphertext)
  6. Remove PKCS7 padding → plaintext
```

### Why the random IV matters

AES-CBC is **deterministic** given the same key and IV. Without a fresh random IV each time, encrypting the same file twice would produce identical ciphertext and an attacker could tell you sent the same file again (we do not want this!). The random IV ensures the same plaintext always produces **different** ciphertext.

{:.highlight}
The IV does not need to be secret. It is included in the token in plaintext. Security comes from the **key**, not the IV.

### Why the HMAC matters

AES-CBC provides **confidentiality** but not **integrity**. An attacker could flip bits in the ciphertext without knowing the key, and the decrypted result would be silently corrupted garbage. The HMAC detects this: if even one bit of the token is modified, the HMAC check fails and `session_decrypt` returns `NULL`.

{:.note}
This combination is called "authenticated encryption". AES provides confidentiality, HMAC provides integrity.

### Why constant-time comparison matters

`session_decrypt` uses `CRYPTO_memcmp()` rather than `memcmp()` to compare HMACs. A naive `memcmp` returns early on the first mismatching byte, leaking timing information that an attacker could exploit to forge MACs byte by byte. `CRYPTO_memcmp` always examines every byte regardless of where the first difference occurs.

## Block Cipher Modes: ECB vs CBC

**Used in:** Lab Part 2 (image encryption). PA2 didn't need it but it's discussed here for recap.

```c
/* raw OpenSSL EVP API */
const EVP_CIPHER *ecb = EVP_des_ede_ecb();
const EVP_CIPHER *cbc = EVP_des_ede_cbc();
EVP_EncryptInit_ex(ctx, ecb, NULL, key, NULL);   /* ECB: no IV */
EVP_EncryptInit_ex(ctx, cbc, NULL, key, iv);      /* CBC: needs IV */
EVP_EncryptUpdate(ctx, out, &len, padded_data, padded_len);
EVP_EncryptFinal_ex(ctx, out + len, &final_len);
```

### What each mode does

**ECB (Electronic Codebook):**

```
Block 1:  ciphertext_1 = Encrypt(key, plaintext_1)
Block 2:  ciphertext_2 = Encrypt(key, plaintext_2)
Block 3:  ciphertext_3 = Encrypt(key, plaintext_3)
```

Each block is encrypted **independently**. Identical plaintext blocks always produce identical ciphertext blocks.

**CBC (Cipher Block Chaining):**

```
Block 1:  ciphertext_1 = Encrypt(key, plaintext_1 XOR iv)
Block 2:  ciphertext_2 = Encrypt(key, plaintext_2 XOR ciphertext_1)
Block 3:  ciphertext_3 = Encrypt(key, plaintext_3 XOR ciphertext_2)
```

Each block is XORed with the **previous ciphertext** before encryption. The first block uses the IV. Identical plaintext blocks produce **different** ciphertext because the chaining input differs.

### Why ECB is insecure for images

In the Cryptography lab, you encrypt an image column-by-column with ECB. Columns with the same pixel data (e.g., a solid background) produce identical encrypted columns: the image structure, including edges, shapes, and uniform regions, **leaks through**. This is the well-known ["ECB penguin" problem](https://crypto.stackexchange.com/questions/14487/can-someone-explain-the-ecb-penguin).

CBC fixes this because each column's encryption depends on the previous column's ciphertext, destroying the pattern.


## PKCS7 Padding

**Used in:** Lab Part 2 (3DES blocks), and internally by `session_encrypt`

```c
/* manual implementation (in 2_encrypt_image.c) */
size_t pad = 8 - (len % 8);
memset(out + len, (uint8_t)pad, pad);
```

### How it works

PKCS7 padding ensures the data length is a **multiple of the block size** (8 bytes for 3DES, 16 bytes for AES). It appends N bytes, each with the **value** N:

```
Data: "Hello"    (5 bytes, block size 8) --> need 3 more
Padded: "Hello\x03\x03\x03"

Data: "12345678" (8 bytes, block size 8) --> already aligned, still pad a full block of 8 bytes
Padded: "12345678\x08\x08\x08\x08\x08\x08\x08\x08"  (16 bytes)
```

The pad value **is** the pad length, so the receiver can always remove exactly the right number of bytes. Even when data is already aligned, a full block of padding is added, otherwise the receiver could not distinguish between data that happens to end with `\x01` and data with one byte of padding.


## RSA Key Pair

**Used in:** Lab Part 3 (sign/verify, encrypt/decrypt), PA2 (all tasks)

**API:** [`load_private_key`](#load_private_key), [`rsa_encrypt_block`](#rsa_encrypt_block), [`rsa_decrypt_block`](#rsa_decrypt_block), [`sign_message_pss`](#sign_message_pss), [`verify_message_pss`](#verify_message_pss)

```c
/* OpenSSL EVP key generation */
EVP_PKEY_CTX *ctx = EVP_PKEY_CTX_new_id(EVP_PKEY_RSA, NULL);
EVP_PKEY_keygen_init(ctx);
EVP_PKEY_CTX_set_rsa_keygen_bits(ctx, 1024);
EVP_PKEY_keygen(ctx, &private_key);
/* private_key holds BOTH private and public parts */
```

### The core math

{:.note-title}
> Recap
> 
> RSA generates two large primes p, q and computes n = p × q. The public key is (e, n) and the private key is (d, n), where e × d ≡ 1 (mod φ(n)).

```
Encrypt:   ciphertext = plaintext^e  mod n     (uses public key)
Decrypt:   plaintext  = ciphertext^d mod n     (uses private key)

Sign:      signature  = hash(msg)^d  mod n     (uses private key)
Verify:    hash(msg) == signature^e  mod n ?   (uses public key)
```

The two operations are mathematically the **same** (modular exponentiation) but serve opposite purposes:

| Operation | Key used | Purpose | Who can do it |
|---|---|---|---|
| **Encrypt** | Public key (e) | Confidentiality: only private key holder can read | Anyone |
| **Decrypt** | Private key (d) | Read the encrypted message | Key holder only |
| **Sign** | Private key (d) | Authentication: prove you hold the key | Key holder only |
| **Verify** | Public key (e) | Check the signature is valid | Anyone |



## RSA Padding: OAEP vs PKCS1v15 vs PSS

**Used in:** Lab Part 3, PA2 (all tasks)

**API:** [`rsa_encrypt_block`](#rsa_encrypt_block), [`sign_message_pss`](#sign_message_pss), [`verify_message_pss`](#verify_message_pss)

### Why padding at all?

Raw RSA we learned in lecture  (`c = m^e mod n`) is **insecure** because:

- Encrypting the <span class="orange-bold">same</span> message twice gives the same ciphertext (deterministic)
- Small messages can be attacked **mathematically**
- Has <span class="orange-bold">no</span> integrity protection

Padding adds **randomness** and **structure** before the RSA operation.

### The three padding schemes

| Scheme | Used for | Random? | C (common.h) |
|---|---|---|---|
| **OAEP** | Encryption / decryption | Yes (random seed) | `rsa_encrypt_block(..., 1)` |
| **PKCS1v15** | Encryption (legacy) | Minimal | `rsa_encrypt_block(..., 0)` |
| **PSS** | Signing / verification | Yes (random salt) | `sign_message_pss()` |

### OAEP (for encryption)

Here are the general steps:
```
1. Generate random seed (32 bytes for SHA-256)
2. Use MGF1 (a mask generation function based on SHA-256) to expand the seed
3. XOR the plaintext with the mask
4. Combine seed + masked plaintext into a single padded block
5. RSA-encrypt the padded block: c = padded^e mod n
```

Because of the random seed, the same plaintext encrypted twice produces **different** ciphertext. The overhead is 2 × hash_length + 2 = 66 bytes for SHA-256, leaving 128 - 66 = **62 bytes** of usable plaintext per RSA block.

### PSS (for signing)

Here are the general steps:
```
SIGNING:
  1. hash = SHA-256(message)                    --> 32 bytes
  2. salt = random bytes (up to max length)     --> random!
  3. H = SHA-256(padding || hash || salt)       --> 32 bytes
  4. Encode H + salt into a padded block using MGF1
  5. signature = padded_block^d mod n           --> 128 bytes

VERIFICATION:
  1. Recover padded_block = signature^e mod n
  2. Extract H and salt from the padded block
  3. hash = SHA-256(message)
  4. H' = SHA-256(padding || hash || recovered_salt)
  5. Check: H == H' ?
```

**How can verification work if signing is random?** PSS adds a random salt during signing, so signing the same message twice gives different signatures. Verification still works because the salt is **embedded inside** the signature and recovered during verification (step 2). The verifier does not need to know the salt in advance. The randomness prevents an attacker from forging signatures but does not prevent legitimate verification.

### PKCS1v15 (for encryption, legacy)

Much simpler than OAEP: prepends `0x00 || 0x02 || random_non_zero_bytes || 0x00` before the plaintext. The overhead is minimum 11 bytes, leaving 128 - 11 = **117 bytes** of usable plaintext. Used for legacy compatibility (e.g., the CA signs certificates with this scheme for PA2 demo purposes). PSS and OAEP are strictly better for new code.


## SHA-256 Hashing

**Used in:** Lab Part 3 (explicit digest), PA2 (internally by `sign`/`verify`)

```c
EVP_MD_CTX *ctx = EVP_MD_CTX_new();
EVP_DigestInit_ex(ctx, EVP_sha256(), NULL);
EVP_DigestUpdate(ctx, data, data_len);
EVP_DigestFinal_ex(ctx, digest, &digest_len);  /* always 32 bytes */
EVP_MD_CTX_free(ctx);
```

The SHA-256 algorithm has the following properties that are crucial for authentication purposes:
- **Fixed output:** always 32 bytes, regardless of whether the input is 5 bytes or 5 GB.
- **Deterministic:** same input always produces the same hash.
- **One-way:** given a hash, you cannot recover the original input.
- **Collision-resistant:** it is computationally infeasible to find two different inputs with the same hash.

### Why hash before signing?

In CP1, you encrypted large files with RSA by chunking the data into 62-byte (OAEP) or 117-byte (PKCS1v15) blocks and RSA-encrypting each one. So technically you *could* do the same for signing: chunk the message and sign each chunk. But nobody does this, for two reasons.

1. **Speed**: RSA is modular exponentiation and is orders of magnitude slower than SHA-256. Hashing the entire file takes microseconds and signing every 62-byte chunk of a large file with RSA would be painfully slow. Hashing reduces the work to one single RSA operation regardless of file size.
2. **Correctness**: A signature is meant to prove the message as a whole has not been tampered with. Signing individual chunks does not give you that guarantee without additional structure to tie the chunks together in order.

So hashing before signing is primarily a **performance** choice, with the side effect of also reducing the input to something that trivially fits in one RSA block. `sign_message_pss()` does this internally via `EVP_DigestSignUpdate()`, so you pass it the full message and never think about chunking.

## X.509 Certificates

**Used in:** PA2 (all tasks)

**API:** [`load_cert_file`](#load_cert_file), [`load_cert_bytes`](#load_cert_bytes), [`verify_server_cert`](#verify_server_cert)

### What is in a certificate?

{:.note}
> Recap
> 
> A certificate binds a **public key** to an **identity**.


The certificate contains these information among other things. These are some example field:

```
Subject:     CN=sutd.edu.sg, O=SUTD, C=SG    (who this cert belongs to)
Public Key:  RSA 1024-bit (e, n)               (the server's public key)
Validity:    Not Before / Not After            (expiration window)
Issuer:      CN=50005, O=SUTD                  (the CA that signed this cert)
Signature:   RSA-PKCS1v15(CA_private_key, hash(above fields))
```

The verification chain works as follows:
```
1. Client has the CA cert (cacertificate.crt), shipped with the PA2 starter code.
2. Server sends its signed cert (server_signed.crt) over the network.
3. Client verifies:
     - CA_public_key.verify(server_cert.signature, server_cert.data)
     - This proves the CA vouches for the server's public key.
4. Client extracts the server's public key from the verified cert.
   - The client now has a trustworthy public key to use for the rest of the protocol.
```

Here are the general steps:
```c
X509    *server_cert   = load_cert_bytes(cert_data, cert_len);
int      ok            = verify_server_cert(server_cert, "source/auth/cacertificate.crt");
EVP_PKEY *server_pub   = X509_get_pubkey(server_cert);
```


# Part 2: common.c / common.h API Reference


## Building and Conventions

### Compile flag

```bash
gcc your_file.c common.c -o your_binary -lssl -lcrypto
```

### Memory ownership

Every function that returns an `unsigned char *` allocates with `malloc()`. The **caller must `free()`** the returned pointer. This is noted at each relevant function below.

### Error reporting

- Pointer-returning functions return `NULL` on failure.
- `int`-returning functions return `-1` or `0` (documented per function).
- Diagnostics are printed to `stderr` via `print_ssl_error()` or `fprintf(stderr, ...)`.


## Constants

All constants are `#define` macros in `common.h`.

| Constant | Value | Meaning |
|---|---|---|
| `INT_BYTES` | 8 | All wire-protocol length-prefix fields are 8 bytes, big-endian. |
| `RSA_KEY_BITS` | 1024 | RSA modulus size. Matches `generate_keys.py`. |
| `RSA_KEY_BYTES` | 128 | `RSA_KEY_BITS / 8`. Every RSA-encrypted block is exactly this many bytes. |
| `RSA_OAEP_CHUNK` | 62 | Max plaintext per RSA-OAEP/SHA-256 block: 128 - 2×32 - 2 = 62 bytes. See [Section 5](#rsa-padding-oaep-vs-pkcs1v15-vs-pss). |
| `RSA_PKCS1_CHUNK` | 117 | Max plaintext per RSA-PKCS1v15 block: 128 - 11 = 117 bytes. See [Section 5](#rsa-padding-oaep-vs-pkcs1v15-vs-pss). |
| `AES_KEY_LEN` | 16 | AES-128 key length in bytes. |
| `AES_IV_LEN` | 16 | AES-CBC initialisation vector length in bytes. |
| `AES_BLOCK` | 16 | AES block size; also the worst-case PKCS7 padding overhead. See [Section 3](#pkcs7-padding). |
| `HMAC_KEY_LEN` | 16 | HMAC signing-key length (first 16 bytes of a session key). |
| `HMAC_LEN` | 32 | SHA-256 output length; appended to every symmetric token. See [Section 6](#sha-256-hashing). |
| `SESSION_KEY_LEN` | 32 | `HMAC_KEY_LEN + AES_KEY_LEN`. Full session key length. |


## Wire Protocol Message Types

These integer codes identify the type of each frame on the wire. The same values are used by the Python reference implementation.

| Constant | Value | Purpose |
|---|---|---|
| `MSG_FILENAME` | 0 | Client sends the name of the file being transferred. |
| `MSG_FILE_DATA` | 1 | Client sends a chunk (or all) of file data. |
| `MSG_CLOSE` | 2 | Signals end of session; the peer should close the connection. |
| `MSG_AUTH` | 3 | Authentication exchange (certificate and PSS signature). |
| `MSG_SYMKEY` | 4 | CP2 only: RSA-encrypted session key exchange. |



## Integer / Byte Conversion

All length-prefix fields on the wire are 8-byte, big-endian unsigned integers. See [Section 10](#10-wire-protocol-message-types) for context on where they appear.

### `int_to_bytes`

```c
void int_to_bytes(uint64_t x, unsigned char buf[INT_BYTES]);
```

Encodes a 64-bit unsigned integer into exactly 8 bytes, most-significant byte first. The result is written directly into `buf` — no allocation, no return value. Iterates from byte index 7 down to 0, masking the lowest 8 bits and right-shifting on each step.

### `bytes_to_int`

```c
uint64_t bytes_to_int(const unsigned char buf[INT_BYTES]);
```

Decodes 8 big-endian bytes back into a `uint64_t` by left-shifting and OR-ing each byte in order.

## Socket Helpers

TCP does not guarantee that a single `send()` or `recv()` transfers all requested bytes. These helpers loop internally until exactly the right number of bytes has moved.

### `read_bytes`

```c
unsigned char *read_bytes(int sockfd, uint64_t length);
```

Allocates a `malloc`-ed buffer of `length` bytes and fills it by calling `recv()` in a loop, reading up to 1024 bytes per iteration.

{:.note}
> **Returns:** pointer to filled buffer on success, `NULL` on socket error or allocation failure.
> **Memory:** caller must `free()` the returned buffer.



### `send_all`

```c
int send_all(int sockfd, const unsigned char *buf, uint64_t length);
```

Sends exactly `length` bytes by looping over `send()` until all data is written.

{:.note}
**Returns:** `0` on success, `-1` on failure.


### `send_int`

```c
int send_int(int sockfd, uint64_t value);
```

Convenience wrapper: calls `int_to_bytes()` then `send_all()` to send an 8-byte big-endian integer in one call. Used to prefix every message with its payload length.

{:.note}
**Returns:** `0` on success, `-1` on failure.


## Key and Certificate Loading

### `load_private_key`

```c
EVP_PKEY *load_private_key(const char *filename);
```

Opens a PEM-format private key file and parses it with `PEM_read_PrivateKey()`. The returned `EVP_PKEY *` holds both private and public components.

{:.note}
> **Returns:** `EVP_PKEY *` on success, `NULL` on failure.
> **Memory:** caller must `EVP_PKEY_free()` the returned key.


### `load_cert_file`

```c
X509 *load_cert_file(const char *filename);
```

Opens a PEM-format certificate file from disk and parses it with `PEM_read_X509()`.

{:.note}
> **Returns:** `X509 *` on success, `NULL` on failure.
> **Memory:** caller must `X509_free()` the returned cert.


### `load_cert_bytes`

```c
X509 *load_cert_bytes(const unsigned char *data, int len);
```

Parses a PEM certificate from an in-memory byte buffer (e.g., one received over the socket). Uses an OpenSSL `BIO` memory buffer internally so no temporary file is needed.

{:.note}
> **Returns:** `X509 *` on success, `NULL` on failure.
> **Memory:** caller must `X509_free()` the returned cert.


## Certificate Verification

### `verify_server_cert`

```c
int verify_server_cert(X509 *server_cert, const char *ca_cert_path);
```

Verifies that `server_cert` was signed by the CA at `ca_cert_path`, and that the current time falls within the certificate's validity window. See [Section 7](#7-x509-certificates) for the full trust chain explanation.

**What it does internally:**
1. Loads the CA certificate from disk.
2. Prints the server certificate's `Not Before` and `Not After` fields to stdout.
3. Creates an `X509_STORE` with the CA cert as the single trusted root.
4. Runs `X509_verify_cert()`, which checks both the CA signature and the validity period in one call.
5. On failure, prints the OpenSSL error string (e.g., `"certificate has expired"`).

{:.note}
**Returns:** `1` on success, `0` on failure. Prints diagnostics either way.

Here's how to use it:
```c
X509    *server_cert = load_cert_bytes(cert_data, cert_len);
int      ok          = verify_server_cert(server_cert, "source/auth/cacertificate.crt");
if (!ok) { /* abort handshake */ }
EVP_PKEY *server_pub = X509_get_pubkey(server_cert);
```


## RSA-PSS Signing and Verification

See [this section](#rsa-padding-oaep-vs-pkcs1v15-vs-pss) for the PSS algorithm details and [this section](#rsa-key-pair) for the RSA math.

### `sign_message_pss`

```c
unsigned char *sign_message_pss(EVP_PKEY *priv_key,
                                const unsigned char *msg, size_t msg_len,
                                size_t *sig_len);
```

Signs an arbitrary-length message using RSA-PSS with SHA-256 and maximum salt length. There is no size limit on `msg` because OpenSSL hashes it internally before applying RSA.

**What it does internally:**
1. Initialises an `EVP_MD_CTX` with `EVP_DigestSignInit()` using `EVP_sha256()`.
2. Sets padding to `RSA_PKCS1_PSS_PADDING` with MGF1-SHA-256 and `RSA_PSS_SALTLEN_MAX`.
3. Feeds the message with `EVP_DigestSignUpdate()`.
4. Calls `EVP_DigestSignFinal()` twice: first to get the output length, then to fill a `malloc`-ed buffer.

{:.note}
> **Returns:** pointer to signature buffer (length written to `*sig_len`) on success, `NULL` on failure.
> **Memory:** caller must `free()` the returned buffer.


### `verify_message_pss`

```c
int verify_message_pss(X509 *cert,
                       const unsigned char *sig, size_t sig_len,
                       const unsigned char *msg, size_t msg_len);
```

Verifies an RSA-PSS signature using the public key extracted from `cert` via `X509_get_pubkey()`. Uses the same SHA-256 / MGF1-SHA-256 / max-salt configuration as `sign_message_pss`.

{:.note}
**Returns:** `1` if the signature is valid, `0` if invalid or on error.


## RSA Encryption and Decryption

See [this section](#rsa-padding-oaep-vs-pkcs1v15-vs-pss) for the padding details.

### `rsa_encrypt_block`

```c
unsigned char *rsa_encrypt_block(EVP_PKEY *pub_key,
                                 const unsigned char *plain, size_t plain_len,
                                 size_t *out_len, int use_oaep);
```

Encrypts a single block of plaintext with RSA using the given public key.

| `use_oaep` | Padding | Max plaintext | When to use |
|---|---|---|---|
| `1` | OAEP / SHA-256 / MGF1-SHA-256 | 62 bytes (`RSA_OAEP_CHUNK`) | All new code, CP2 key exchange |
| `0` | PKCS1v15 | 117 bytes (`RSA_PKCS1_CHUNK`) | Legacy / CA certificate compatibility |

Output is always `RSA_KEY_BYTES` (128) bytes long.

{:.note}
> **Returns:** pointer to ciphertext buffer on success, `NULL` on failure.
> **Memory:** caller must `free()` the returned buffer.


### `rsa_decrypt_block`

```c
unsigned char *rsa_decrypt_block(EVP_PKEY *priv_key,
                                 const unsigned char *cipher, size_t cipher_len,
                                 size_t *out_len, int use_oaep);
```

Decrypts a single RSA-encrypted block. The `use_oaep` flag must match the value used during encryption.

{:.note}
> **Returns:** pointer to plaintext buffer (length written to `*out_len`) on success, `NULL` on failure.
> **Memory:** caller must `free()` the returned buffer.


## Symmetric Encryption

See [this section](#symmetric-encryption-aes-128-cbc) for the full algorithm walkthrough and token layout.

**Token layout:**
```
[ IV (16 bytes) | AES-128-CBC ciphertext (PKCS7 padded) | HMAC-SHA256 (32 bytes) ]
```

**Key layout (32 bytes total):**
```
[ HMAC key bytes 0..15 | AES key bytes 16..31 ]
```


### `generate_session_key`

```c
int generate_session_key(unsigned char key_out[SESSION_KEY_LEN]);
```

Fills `key_out` with 32 cryptographically secure random bytes using `RAND_bytes()`.

{:.note}
**Returns:** `0` on success, `-1` on failure.


### `session_encrypt`

```c
unsigned char *session_encrypt(const unsigned char key[SESSION_KEY_LEN],
                               const unsigned char *plain, size_t plain_len,
                               size_t *out_len);
```

Encrypts `plain` and returns a token in the layout above. Allocates output sized `AES_IV_LEN + plain_len + AES_BLOCK + HMAC_LEN` (worst case). OpenSSL applies PKCS7 padding automatically.

{:.note}
> **Returns:** pointer to token on success, `NULL` on failure.
> **Memory:** caller must `free()` the returned buffer.

### `session_decrypt`

```c
unsigned char *session_decrypt(const unsigned char key[SESSION_KEY_LEN],
                               const unsigned char *token, size_t token_len,
                               size_t *out_len);
```

Decrypts a token produced by `session_encrypt()`. Rejects tokens shorter than 64 bytes (`AES_IV_LEN + AES_BLOCK + HMAC_LEN`). Uses `CRYPTO_memcmp()` for constant-time HMAC comparison — see [Section 1](#why-constant-time-comparison-matters) for why this matters.

{:.note}
> **Returns:** pointer to plaintext (length written to `*out_len`) on success, `NULL` on HMAC failure or decryption error.
> **Memory:** caller must `free()` the returned buffer.

## Utility Functions

### `print_ssl_error`

```c
void print_ssl_error(const char *context);
```

Fetches the most recent OpenSSL error with `ERR_get_error()`, formats it with `ERR_error_string_n()`, and prints to `stderr`:

```
[OpenSSL] <context>: <error string>
```

Called internally by all functions on failure. You can also call it directly after any raw OpenSSL API call in your own code.


### `get_time`

```c
double get_time(void);
```

Returns the current monotonic clock time as a `double` in seconds with sub-second precision, using `clock_gettime(CLOCK_MONOTONIC, ...)`. Useful for timing protocol phases.

{:.note}
`CLOCK_MONOTONIC` is used rather than `CLOCK_REALTIME` so the value is unaffected by system clock adjustments (NTP, daylight saving, etc.). It is not an absolute Unix timestamp, only the **difference** between two calls is meaningful.