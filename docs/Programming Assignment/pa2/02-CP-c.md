---
title: Confidentiality Protocol
permalink: /pa2/part2-c
key: pa2-part2
layout: default
nav_order: 2
parent: Programming Assignment 2
grand_parent: Programming Assignment
---

* TOC
{:toc}

**50.005 Computer System Engineering**
Information Systems Technology and Design
Singapore University of Technology and Design
**Natalie Agus (Summer 2026)**

# The Confidentiality Protocol
{: .no_toc}

**Congratulations!** 🎉, since you have implemented AP at this point, you can now be assured that you are uploading your file to the **right** destination and not a fake, malicious server.

However, can you trust the network _path_ used for your upload?
1. It may go through many **intermediate** routers and communication links that you don't know very well (or not at all).
2. Also, could people tap the links and **steal** your data?

Unfortunately, malicious parties can eavesdrop on your conversation with the server. Although _sometimes_ confidentiality is not an issue, it is obviously not the case when you're uploading _personal_ files to a remote server.

To avoid the theft of data in transmission, you should implement a confidentiality protocol (CP) where we basically encrypt the data in transit. There are two basic ways to do this:

1. Confidentiality Protocol 1: Using **public** key cryptography
2. Confidentiality Protocol 2: Using **symmetric** key cryptography

## Confidentiality Protocol 1 (CP1)

### Task 2 (2%)

{:.task}
`TASK 2:` Design a protocol to protect the confidentiality of the <span style="color:#f77729;"><b>content</b></span> of the uploaded file using <span style="color:#f77729;"><b>public key</b></span> cryptography. For simplicity, the filename (`MODE 1`) does <span class="orange-bold">not</span> have to be encrypted.


The suggested steps are as follows:

1. The client **encrypts** the file data (in units of **byte** blocks) before sending, and SecureStore **decrypts** on receive.
2. Since the RSA key size is 1024 bits (during Server Key generation), it can encrypt/decrypt <span class="orange-bold">at most</span> 128 bytes of data at a time.
   - However you will need to use a **padding** for the data blocks and padding + data blocks must be 128 bytes at maximum.
   - There are two options:
     - `PKCS1v15`: min 11 bytes of padding, max 117 bytes data blocks
     - `OAEP` with `MGF1` and `SHA-256`: min 66 bytes of padding, max 62 bytes data blocks
   - During checkoff, **inform us** which padding option you used for the assignment. Please stick to just one for the entire assignment.

In C, the padding mode is controlled by the `use_oaep` parameter (1 for OAEP, 0 for PKCS1v15) that you pass to `rsa_encrypt_block()` and `rsa_decrypt_block()`. Conveniently, the chunk size constants are defined for you in `common.h`:

```c
#define RSA_OAEP_CHUNK  62   /* max plaintext bytes per OAEP block */
#define RSA_PKCS1_CHUNK 117  /* max plaintext bytes per PKCS1v15 block */
#define RSA_KEY_BYTES   128  /* ciphertext output size per block */
```

**Usage**:

```c
/* Encrypting one block (client side) */
size_t enc_len = 0;
unsigned char *enc_block = rsa_encrypt_block(
    server_pub_key,       /* EVP_PKEY* from X509_get_pubkey() */
    plaintext_ptr,        /* pointer to current chunk */
    chunk_len,            /* 62 or 117 bytes (or less for the last chunk) */
    &enc_len,             /* will be set to 128 */
    use_oaep              /* 1 for OAEP, 0 for PKCS1v15 */
);
/* enc_block is malloc'd, you must free() after use */

/* Decrypting one block (server side) */
size_t dec_len = 0;
unsigned char *dec_block = rsa_decrypt_block(
    private_key,          /* EVP_PKEY* from load_private_key() */
    ciphertext_ptr,       /* pointer to current 128-byte block */
    RSA_KEY_BYTES,        /* always 128 */
    &dec_len,             /* will be set to original chunk size */
    use_oaep
);
/* dec_block is malloc'd , you must free() after use */
```

Since you were able to implement AP, you already have everything you need for CP1. Just remember that in public key cryptography, we could use <span class="orange-bold">either</span> the public or private key for the encryption!

{:.note-title}
> Which key to use?
>
> Figure out **which key** to use to encrypt the data from the Client to securely upload your files to the server while protecting the **confidentiality** of the file.

### Expanding the FTP for CP1

{:.note-title}
> Instruction
> 
> Make a **copy** of Task 1 files: `ClientWithSecurityAP.c` and `ServerWithSecurityAP.c` each, and name them `ClientWithSecurityCP1.c` and `ServerWithSecurityCP1.c`.
> 
> <span class="orange-bold">Leave the original files as is.</span> You should have 6 `.c` files now under `source/` (plus the two `common.*` files). There's no need to implement any new `MODE` in this task.

Here's a recap of the `MODE` for CP1:
- `0` (`MSG_FILENAME`): client will send `M1`: size of filename, and `M2`: the filename (no need to modify, same as original)
- `1` (`MSG_FILE_DATA`): This is modified in Task 2 (CP1). Client will send `M1`: size of data block in `M2`, and `M2`: <span class="orange-bold">encrypted</span> blocks of data to the server (NEW for Task 2)
- `2` (`MSG_CLOSE`): client closes connection (no need to modify, same as original)
- `3` (`MSG_AUTH`): client begins authentication protocol (Task 1)

### Grading

We will **manually** check the implementation of your `MODE 1` in both Client and Server source files.

#### Commit Task 2

Save your changes and commit the changes (assuming your current working directory is the project root):

```bash
git add source/ServerWithSecurityCP1.c source/ClientWithSecurityCP1.c
git commit -m "feat: Complete Task 2"
```

## Confidentiality Protocol 2 (CP2)

### Task 3 (2%)

{:.task}
`TASK 3:` Design a protocol to protect the confidentiality of the <span style="color:#f77729;"><b>content</b></span> of the uploaded file using <span style="color:#f7007f;"><b>symmetric key</b></span> cryptography.

Although CP1 is easy to implement, it is unbearably **slow**. Try using it on a large file (>100MB) and observe its slowdown relative to no encryption (no confidentiality).

To circumvent this, we shall implement an alternative Confidentiality Protocol 2 (CP2):
- CP2 negotiates a **shared session key** (symmetric key) between the client and server, and
- Uses the session key to provide **confidentiality** of the file data thereafter.

{:.info-title}
> Key Generation
>
> You **should** use `generate_session_key()` from `common.h` to generate the session keys. This produces a 32-byte random key: first 16 bytes for HMAC-SHA256 signing, last 16 bytes for AES-128-CBC encryption. The `session_encrypt()` / `session_decrypt()` functions then use this key for AES-128-CBC with PKCS7 padding and HMAC-SHA256 integrity protection. Recall that symmetric key encryption is **much** faster than RSA.

```c
/* Generate a 32-byte session key */
unsigned char session_key[SESSION_KEY_LEN];  /* SESSION_KEY_LEN = 32 */
generate_session_key(session_key);

/* Encrypt entire file data in one call */
size_t encrypted_len = 0;
unsigned char *encrypted = session_encrypt(
    session_key,
    file_data,           /* plaintext bytes */
    (size_t)file_size,   /* plaintext length */
    &encrypted_len       /* output: IV + ciphertext + HMAC */
);
/* encrypted is malloc'd, you must free() */

/* Decrypt (verifies HMAC first, returns NULL if tampered) */
size_t decrypted_len = 0;
unsigned char *decrypted = session_decrypt(
    session_key,
    encrypted,
    encrypted_len,
    &decrypted_len
);
/* decrypted is malloc'd, you must free() */
```

{:.highlight-title}
> Question
>
> Before you proceed, you might want to ask yourself: **who** should generate the session key? Client or server? Why?

### Expanding the FTP for CP2

{:.note-title}
> Instruction
> 
> Make a **copy** of Task 2 files: `ClientWithSecurityCP1.c` and `ServerWithSecurityCP1.c` each, and name them `ClientWithSecurityCP2.c` and `ServerWithSecurityCP2.c`. Any code related to `MODE: 1` implementation must be modified for both Server and Client to utilise the **session key** instead.
> 
> <span class="orange-bold">Leave the original files as is</span>. You should have 8 `.c` files now under `source/`.

Now both client and server must implement `MODE: 4` (`MSG_SYMKEY`), which signifies the `key generation` handshake.

Upon successful AP, one of them (client or server, it is for you to figre out) must then generate a session key using `generate_session_key()` then send `MODE: 4` (via `send_int(sockfd, MSG_SYMKEY)`) to the other party, followed by two messages:
- `M1`: size of `M2` in bytes
- `M2`: generated **session key bytes**. This has to be <span class="orange-bold">kept confidential</span>.

The receipient of `MODE: 4` must receive and decrypt the session key. Then the user can proceed with keying in filename as per the regular FTP.


{:.highlight-title}
> Securely zeroing the key
>
> When your program exits, it is good practice to securely zero the session key from memory so it cannot be recovered from a memory dump. Use `OPENSSL_cleanse(session_key, SESSION_KEY_LEN);` before your program terminates.

Here's a recap of the `MODE` for CP2:
- `0` (`MSG_FILENAME`): client will send `M1`: size of filename, and `M2`: the filename (no need to modify, same as original)
- `1` (`MSG_FILE_DATA`): This is modified in Task 2 (CP1). Client will send `M1`: size of data block in `M2`, and `M2`: <span class="orange-bold">encrypted</span> blocks of data to the server. The server has to decrypt it with the **session key** as well before writing it to file (NEW for Task 3)
- `2` (`MSG_CLOSE`): client closes connection (no need to modify, same as original)
- `3` (`MSG_AUTH`): client begins authentication protocol (Task 1)
  

### Grading

We will **manually** check the implementation of your `MODE 1` and `MODE 4` in both Client and Server source files.

## Commit Task 3

Save your changes and commit the changes (assuming your current working directory is the project root):

```bash
git add source/ServerWithSecurityCP2.c source/ClientWithSecurityCP2.c
git commit -m "feat: Complete Task 3"
```

By the end of Task 3, you have 1 server-client pair for each task (8 `.c` source files in total under `source/`, plus the two `common.*` files): the original (non-secure), AP, CP1, and CP2 pairs.
