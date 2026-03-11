---
layout: default
permalink: /labs/07-encryption-c
title: Cryptography
description:  Symmetric key encryption on images and texts
parent: Labs
nav_order:  7
---

* TOC
{:toc}

**50.005 Computer System Engineering**
<br>
Information Systems Technology and Design
<br>
Singapore University of Technology and Design
<br>
**Natalie Agus (Summer 2026)**



# C Cryptography with OpenSSL
{:.no_toc}


In NS Module 3, we examined how the security properties of <span style="color:#f77729;"><b>confidentiality</b></span> and data <span style="color:#f77729;"><b>integrity</b></span> could be protected by using symmetric key cryptography and signed message digests. In this lab exercise, you will learn how to write a C program that makes use of <span style="color:#f77729;"><b>3DES</b></span> for data encryption, <span style="color:#f77729;"><b>SHA256</b></span> for creating message digests and <span style="color:#f77729;"><b>RSA</b></span> for digital signing, all via the OpenSSL library.

At the end of this lab exercise, you should be able to:

- Understand how symmetric key cryptography can be used to encrypt data and protect its confidentiality.
- Understand how _multiple_ blocks of data are handled using different block cipher modes and padding.
- Compare the different block cipher modes in terms of how they operate.
- Understand how hash functions can be used to create fixed-length message digests.
- Understand how public key cryptography (e.g., RSA algorithm) can be used to create digital signatures.
- Understand how to create message digest using hash functions (e.g., MD5, SHA-1, SHA-256, SHA-512, etc) and sign it using RSA to guarantee data integrity.

There are 3 parts of this lab:

1. Symmetric key encryption for a <span style="color:#f77729;"><b>text</b></span> file
2. Symmetric key encryption for an <span style="color:#f77729;"><b>image</b></span> file
3. Signed message <span style="color:#f77729;"><b>digests</b></span>

# Submission 

You are to complete this lab's questionnaire on eDimension as you complete the tasks.

Once you have completed all tasks, schedule a checkoff as a Team with your Lab TA by next week Friday 6PM. 

{:.task-title}
> ✅ Checkoff
> 
> Demonstrate completion of ALL tasks in this handout: encryption and decryption of text, encryption and decryption of images using CBC and ECB mode, generation of message digest and verification. 

### System Requirements

This project requires a C11 compiler, `make`, and OpenSSL (`libssl-dev`). You should have the compiler and `make` already if you completed the previous labs. Install OpenSSL development headers if needed:

```sh
sudo apt install libssl-dev   # Ubuntu / Debian
brew install openssl          # macOS
```

## Starter Code

Download the starter code:

```
git clone https://github.com/natalieagus/cse-lab-crypto-2026-starter
```

This will result in a directory with the following structure:

```
.[PROJECT_DIR]
├── 1_encrypt_text.c
├── 2_encrypt_image.c
├── 3_sign_digest.c
├── common.c
├── common.h
├── crypto_reference.md
├── Makefile
├── original_files
│   ├── longtext.txt
│   ├── scenery.bmp
│   ├── shorttext_dec.txt
│   ├── shorttext.txt
│   ├── SUTD.bmp
│   └── triangles.bmp
└── README.md
```

## Build the Starter Code

Build all three programs at once using `make`:

```sh
make
```

This produces three executables: `1_encrypt_text`, `2_encrypt_image`, and `3_sign_digest`. Run each one individually:

```sh
./1_encrypt_text
./2_encrypt_image
./3_sign_digest
```

Output files are written to the `output/` directory, which is created automatically.

To clean up all compiled binaries and the output directory:

```sh
make clean
```


# Text Encryption

Symmetric encryption is a type of encryption where only <span style="color:#f77729;"><b>one</b></span> key (a secret key) is used to both encrypt and decrypt a message. There are many symmetric key encryption algorithms: AES, 3DES, Blowfish, Rivest Cipher 5, etc. For this task, we are going to use <span style="color:#f77729;"><b>AES-128-CBC</b></span> combined with an <span style="color:#f77729;"><b>HMAC-SHA256</b></span> for authentication. 

Open `1_encrypt_text.c` before proceeding.


## Key Generation
### Task 1-1 

`TASK 1-1:` Generate a fresh symmetric session key using `generate_session_key`. They key as been declared for you, study the file to figure out where it is.
{:.task}


```cpp
/* Task 1-1: Generate a symmetric key */
memset(symmetric_key, '\0', SESSION_KEY_LEN);

// TODO: Task 1-1

/* END OF TASK 1-1 */
```

This key is used to both encrypt and decrypt messages. You need to <span style="color:#f77729;"><b>keep this safe</b></span> in real applications. Any adversary that can access this key will be able to decrypt all your supposedly confidential messages.

The helper `generate_session_key` from `common.c` fills a 32-byte buffer with cryptographically secure random bytes using OpenSSL's `RAND_bytes`.

The 32-byte key is split internally: the first 16 bytes are used as the HMAC signing key and the last 16 bytes are used as the AES-128 encryption key.

## Encryption and Decryption

### Task 1-2

`TASK 1-2:` Encrypt the raw file bytes using `session_encrypt`.
{:.task}

```cpp
/* TASK 1-2: encrypt text */
/* Encrypt using session_encrypt (AES-128-CBC + HMAC) */

size_t encrypted_len = 0;

// TODO: Task 1-2
unsigned char *encrypted_bytes = 0; // modify this assignment

/* END OF TASK 1-2 */
```

`session_encrypt` from `common.c` takes the key and a plaintext buffer, applies AES-128-CBC with a freshly generated random IV, then appends an HMAC-SHA256 over the IV and ciphertext. The resulting token has this layout:

```
IV (16 bytes) || ciphertext (variable) || HMAC-SHA256 (32 bytes)
```

Usage:

```c
size_t encrypted_len = 0;
unsigned char *encrypted_bytes = session_encrypt(
    symmetric_key,
    raw_bytes, (size_t)raw_len,
    &encrypted_len
);
```

{:.note-title}
> Token Layout
>
> The output token produced by `session_encrypt` mirrors the Fernet token format:
> * **IV**: 16 bytes, the initialization vector (randomly generated per call)
> * **Ciphertext**: variable length, the AES-128-CBC encrypted, PKCS7-padded data
> * **HMAC**: 32 bytes, HMAC-SHA256 over (IV || ciphertext), providing integrity

### Task 1-3

`TASK 1-3:` Decrypt the raw bytes using `session_decrypt`. Fill in your answer under the `TODO` for this task.
{:.task}

```cpp
/* TASK 1-3: Decrypt the text */
/* Decrypt using session_decrypt (verifies HMAC, then AES-CBC) */
size_t decrypted_len = 0;

// TODO: Task 1-3
unsigned char *decrypted_bytes = 0; // modify this assignment

/* END OF TASK 1-3 */
```

`session_decrypt` first verifies the HMAC. If the token has been tampered with, it returns `NULL` immediately. If verification passes, it decrypts the AES-128-CBC ciphertext and removes the PKCS7 padding:

```c
size_t decrypted_len = 0;
unsigned char *decrypted_bytes = session_decrypt(
    symmetric_key,
    encrypted_bytes, encrypted_len,
    &decrypted_len
);

if (!decrypted_bytes) {
    fprintf(stderr, "Decryption failed (HMAC verification error)\n");
}
```

## Creating a Printable Text Using Base64

If you attempt to write the <span style="color:#f77729;"><b>encrypted</b></span> bytes directly as text, you will see garbled or unprintable characters. That is because the ciphertext bytes are <span style="color:#f77729;"><b>no longer</b></span> valid text encoding.

In `enc_text`, the encrypted bytes are therefore Base64-encoded before saving, so the output file is readable as a plain string:

```c
/* Base64-encode for printable output */
size_t b64_len = 0;
char *encrypted_text = base64_encode(encrypted_bytes, encrypted_len, &b64_len);

FILE *out = fopen(output_filename, "w");
if (out) { fwrite(encrypted_text, 1, b64_len, out); fclose(out); }
```

Correspondingly, `dec_text` Base64-decodes the file content before decrypting:

```c
/* Base64-decode back to raw ciphertext bytes */
size_t encrypted_len = 0;
unsigned char *encrypted_bytes = base64_decode(
    encrypted_text, (size_t)text_len, &encrypted_len
);
```

You may experiment writing the encrypted bytes to the file directly without encoding, and observe the difference.
{:.note}

## Note about Encoding
This section is just for your information as there seems to be rampant misunderstanding that encoding and encryption are both the "same" thing just because they transform data from one format to another.
{:.info}
<span class="orange-bold">Encoding != Encryption</span>. Encoding schemes and encryption algorithms serve <span class="orange-bold">fundamentally different purposes</span>, even though both involve transforming data from one form to another. The key distinctions lie in their objectives, methods, and the necessity for secrecy.
{:.error}


Encoding is designed to **transform** data into a specific format for efficient transmission or storage, making it convenient to process by computers and human-readable in some cases. Encoding does not aim to keep information secret but ensures that it is represented in a form suitable for various systems or applications. Common examples of encoding include **Base64**, **ASCII**, and **Unicode**.

Encryption, on the other hand, is intended to protect data confidentiality by converting the original information (plaintext) into an unreadable format (ciphertext) using an algorithm and a secret key. The primary purpose of encryption is to ensure that only authorized parties can access the original information by decrypting the ciphertext using the correct key. Examples of encryption algorithms include AES (Advanced Encryption Standard), RSA (Rivest-Shamir-Adleman), and DES (Data Encryption Standard).

### Encoding Schemes

Encoding schemes are methods used to represent data in specific formats, often for the purposes of data storage, transmission, and interpretation by computers and human operators. Here are examples of common encoding schemes, categorized by their primary use:

**Textual Data Encoding:**
- **ASCII (American Standard Code for Information Interchange):** Uses 7 bits to represent 128 characters, including English letters, digits, and control characters. It's the basis for many other encoding schemes.
- **ISO-8859-1 (Latin-1):** Extends ASCII to 8 bits, adding support for Western European languages by including characters such as ñ, ç, and ß.
- **UTF-8, UTF-16, UTF-32 (Unicode Transformation Formats):** Variable-length encodings that can represent every character in the Unicode character set, accommodating all known characters and symbols from languages around the world.

**Numeric Data Encoding:**
- **Binary:** The most basic form of encoding, representing data in two states, 0 and 1, directly corresponding to the off and on states of a computer's transistors.
- **Hexadecimal (Hex):** Uses base 16, representing numbers using sixteen distinct symbols, 0-9 to represent values zero to nine, and A-F to represent values ten to fifteen.

**Media Data Encoding:**
- **Base64:** Encodes binary data into characters selected from a set of 64, using the letters A-Z, a-z, the numbers 0-9, and characters + and /. It is used to encode binary data in contexts that primarily or solely handle textual data, like embedding images in HTML or email attachments.
- **MIME (Multipurpose Internet Mail Extensions):** An extension of the email protocol that supports sending non-text attachments like audio, video, images, and application programs.

**Data Compression and Archiving:**
- **ZIP:** A file format that supports lossless data compression. A ZIP file may contain one or more files or directories that may have been compressed.
- **GZIP:** A file format and a software application used for file compression and decompression. GZIP is often used in combination with TAR for archiving files.

**Data Integrity and Security:**
- **MD5 (Message Digest Algorithm 5):** A widely used cryptographic hash function producing a 128-bit (16-byte) hash value, typically expressed as a 32-character hexadecimal number. It's used to check the integrity of files.
- **SHA (Secure Hash Algorithm) family:** Cryptographic hash functions that can generate a hash value (digest), including SHA-1, SHA-256, and SHA-512, differing in their bit length and security level. Used for secure data transmission and digital signatures.

Each of these encoding schemes serves specific purposes, from basic representation of text and numbers to complex data integrity and security.

Encoding schemes use a <span class="orange-bold">publicly</span> known method for transforming data, while encryption algorithms require a <span class="orange-bold">secret</span> key for both encrypting and decrypting the data.
{:.important}


# Image Encryption

In this task, we are going to encrypt an <span style="color:#f77729;"><b>image</b></span> with another symmetric key cryptography called 3-DES. The 3-DES applies the DES cipher algorithm three times to <span style="color:#f77729;"><b>each</b></span> data block.

Data Encryption Standard (DES) <span style="color:#f77729;"><b>was</b></span> a US encryption standard for encrypting electronic data. It makes use of a 56-bit key to encrypt 64-bit blocks of plaintext input through repeated rounds of processing using a specialized function. Note that the DES 56-bit key is <span style="color:#f7007f;"><b>no longer considered adequate</b></span> in the face of modern computing.

In 2016, a major security vulnerability in DES and 3DES was [revealed](https://nvd.nist.gov/vuln/detail/CVE-2016-2183). As a result, NIST (National Institute of Standards and Technology) <span style="color:#f7007f;"><b>deprecated</b></span> DES and 3DES for <span style="color:#f7007f;"><b>all</b></span> applications by 2023. It has been <span style="color:#f7007f;"><b>replaced</b></span> by AES which is more secure and more robust.
{:.info}

Nevertheless, we are going to experiment using 3-DES for this task and observe how image encryption brings about different issues.

Open `2_encrypt_image.c` before proceeding.

## Overview

The file contains a single function `enc_img` that reads a 24-bit BMP file, processes it column by column, and writes an encrypted BMP file. BMP structs, PKCS7 padding, and the raw OpenSSL EVP cipher API are all handled directly in this file — unlike Part 1, there is no `common.c` wrapper for 3DES. This gives you direct visibility into the cipher API before PA2 abstracts it.

We will <span style="color:#f77729;"><b>encrypt</b></span> the image <span style="color:#f77729;"><b>column by column</b></span>.

Although we can encrypt the whole image as a flattened array, or row by row, but for the sake of this lab we do it column by column so that we can observe a special form of output.
{:.note}

### Reading Pixel Values per Column

The BMP pixel buffer is read into memory. Each column is then extracted as a byte array of 3-byte RGB values: 3 bytes per pixel, `height` pixels tall.

```c
size_t col_len = (size_t)(height * 3);
unsigned char *col_bytes = malloc(col_len);

for (int r = 0; r < height; r++) {
    int src_row = top_down ? r : (height - 1 - r);
    unsigned char *px = GET_PIXEL(c, src_row);
    col_bytes[r * 3 + 0] = px[0];
    col_bytes[r * 3 + 1] = px[1];
    col_bytes[r * 3 + 2] = px[2];
}
```

### Bottom-up vs Top-down

The `top_down` flag controls the order in which rows are packed into the column byte array. BMP files store rows bottom-up by default, but we can choose to iterate top-down or bottom-up when assembling column bytes for encryption. This will <span style="color:#f77729;"><b>affect</b></span> the encryption result:

- **Top-down:** row 0 (top of image) is packed first.
- **Bottom-up:** row `height-1` (bottom of image) is packed first.

The order _within_ each pixel tuple (RGB) is always preserved. Only the row packing order changes.

## Key Generation

The 3DES key and IV are already given in the code. They are hardcoded for this lab's purpose.

```c
const unsigned char key[8] = {0xb6, 0x11, 0xd5, 0xd7, 0x83, 0xb2, 0x2c, 0x6d};
const unsigned char iv[8]  = {0x94, 0x6b, 0xae, 0x83, 0x40, 0x44, 0xfc, 0x63};
```

## Create a Cipher

{:.note-title}
> Cipher
>
> Creating a Cipher just means instantiating an *object* or *context* that represents the encryption algorithm you want to use, configured with your chosen key and mode. It is not doing any encryption yet. It is just setting up the machinery so you can feed data into it afterward.

### Task 2-1

`TASK 2-1:` Study how a cipher context is created using the raw OpenSSL EVP API for 3DES in either ECB or CBC mode then answer the questions on eDimension.
{:.task}

The `des3_encrypt` helper in `2_encrypt_image.c` selects the cipher based on the `use_cbc` flag:

```c
const EVP_CIPHER *cipher = use_cbc ? EVP_des_ede_cbc() : EVP_des_ede_ecb();

EVP_CIPHER_CTX *ctx = EVP_CIPHER_CTX_new();
EVP_CIPHER_CTX_set_padding(ctx, 0); /* we pad manually */
EVP_EncryptInit_ex(ctx, cipher, NULL, key, iv);
```

`EVP_des_ede_ecb()` selects 3DES in ECB mode (no IV needed).
`EVP_des_ede_cbc()` selects 3DES in CBC mode (IV required).

### ECB Mode

ECB stands for 'electronic codebook'. When using ECB mode, <span style="color:#f77729;"><b>identical</b></span> input blocks always encrypt to the <span style="color:#f77729;"><b>same</b></span> output block.

<img src="{{ site.baseurl }}/assets/contentimage/ns-lab-2/ecb.png"  class="center_seventy"/>

### CBC Mode

CBC stands for 'cipher block chaining'. In CBC mode, the current block is XORed with the previous ciphertext block before encryption (thus the word chained). Decryption is the reverse: decrypt the current ciphertext, then XOR with the previous ciphertext block.
<img src="{{ site.baseurl }}/assets/contentimage/ns-lab-2/cbc.png"  class="center_seventy"/>

{:.note-title}
> Initialization Vectors (IVs)
> 
> IVs are used in encryption to ensure that the same plaintext encrypted with the same key will result in different ciphertexts each time. This randomness is crucial for security in modes like CBC. An IV is generated for **each** encryption operation and is typically included with the ciphertext to be used during decryption.
>
> <span class="orange-bold">They don't have to be secret</span>. The security of encryption does not rely on the secrecy of the IV. Instead, it relies on the secrecy of the symmetric key. The IV's purpose is to introduce randomness to ensure that identical plaintexts do not produce identical ciphertexts.
>
> In Part 1 (text encryption), the IV is included at the front of the token. When decrypting, `session_decrypt` extracts the IV from the token and uses it to decrypt the data.


## Pad Column Bytes

### Task 2-2

`TASK 2-2:` Implement the function `pkcs7_pad`.
{:.task}

```c
static unsigned char *pkcs7_pad(const unsigned char *data, size_t len, size_t *padded_len)
{
    unsigned char *out = malloc(*padded_len);
    /*  TASK 2-2: Implement pkcs7 padding */

    // TODO: Task 2-2

    /* END OF TASK 2-2 */
    return out;
}
```

3DES encrypts <span style="color:#f77729;"><b>64-bit (8-byte) blocks</b></span>. The column byte array length must be a multiple of 8 before it can be encrypted. The `pkcs7_pad` helper appends N bytes each with the value N, where N is the number of bytes needed to reach the next 8-byte boundary:

```c
size_t padded_len = 0;
unsigned char *padded = pkcs7_pad(col_bytes, col_len, &padded_len);
```

For example, if `col_len` is 5:

```
Input:   b'\x01\x02\x03\x04\x05'         (5 bytes)
Padded:  b'\x01\x02\x03\x04\x05\x03\x03\x03'  (8 bytes, padded with 0x03)
```

If `col_len` is already a multiple of 8, a full block of `0x08` bytes is appended so the receiver can always determine exactly how many padding bytes to remove.

PKCS7 padding is a <span style="color:#f77729;"><b>generalization</b></span> of PKCS5 padding. PKCS7 appends N bytes of value N, where N = `block_size - (len % block_size)`.
{:.note}

### Task 2-3

`TASK 2-3:` Encrypt the padded column bytes using `des3_encrypt`. 
{:.task}

```c
    /*
    * TASK 2-3: Encrypt with 3DES (ECB or CBC).
    */
    size_t enc_len = 0;

    // TODO: Task 2-3

    unsigned char *encrypted = 0; // modify this assignment

    /* END OF TASK 2-3 */
```

Note that if `ECB` mode is used, the same column bytes will <span style="color:#f77729;"><b>always</b></span> produce the same encrypted output.
{:.note}

## Observe the Output Images

When you have successfully completed all tasks, there will be 8 output images in `output/`. <span style="color:#f77729;"><b>Observe them</b></span> and think about the answers to these questions.

Look at all `ECB` outputs.

1. Are there any differences between `top-down` and `bottom-up` output?
2. What kind of <span style="color:#f77729;"><b>security vulnerabilities</b></span> does `ECB` mode have? Would you use this to encrypt images in real life?

Then look at all `CBC` outputs:

1. Compare the result of `CBC` mode with `ECB` mode. What differences do you see? Is it an _improvement_ (more secure)?
2. Why do you think the outputs have the "smearing" effect?
3. Does it still have some kind of security vulnerability?
4. Do we want to prioritise `top-down` or `bottom-up` encryption, or does it depend on the image?

{:.note}
> If you're working on a headless Ubuntu server (i.e., no GUI environment), you cannot view images directly using graphical viewers. To inspect or view image files, you might want to transfer them to your local machine. You can use the following command on your local machine: `scp username@server-ip:/path/to/image.bmp .`, and that will transfer the image to your current working directory.
>
> You can use the command `readlink -f [FILENAME]` in the file's working directory to print out its full path.

<img src="{{ site.baseurl }}//docs/Labs/images/07-Lab7-Encryption/2025-06-16-11-50-36.png"  class="center_seventy no-invert"/>



# Message Digest 

In class, we also learned how a signed message digest could be used to guarantee the integrity of a message. Signing the <span style="color:#f77729;"><b>digest</b></span> instead of the message itself gives much better <span style="color:#f77729;"><b>efficiency</b></span>.

In the final task, we will create and sign a message digest, and <span style="color:#f77729;"><b>verify</b></span> it with the corresponding public key.

Open `3_sign_digest.c` before proceeding.

## Generate RSA Key-Pair

### Task 3-1

`TASK 3-1:` Before we can sign any digest, we need to first generate the asymmetric key pair and store it in `key_pair`. We will do it here.
{:.task}

At the beginning of the file, we declare the `key_pair`, which will hold *both* the public and private key.


```cpp
/* Global RSA key pair:  generated once, used by both functions */
static EVP_PKEY *key_pair = NULL;
```

We will set its value here:
```c
    EVP_PKEY_CTX *ctx = EVP_PKEY_CTX_new_id(EVP_PKEY_RSA, NULL);
    /*
     * Task 3-1: Generate RSA key pair.
     * In C/OpenSSL, EVP_PKEY holds both private and public key.
     */

    // TODO: Task 3-1

    /* END OF TASK 3-1 */
    EVP_PKEY_CTX_free(ctx);
```

In OpenSSL's EVP API, a single `EVP_PKEY` object holds both the private and public components. You can generate the pair as follows:
1. Create a context using `EVP_PKEY_CTX_new_id` with parameter `EVP_PKEY_RSA` (given)
2. Call `EVP_PKEY_keygen_init` with the context as parameter
3. Call `EVP_PKEY_CTX_set_rsa_keygen_bits` giving the context and size (we shall use `1024` bits)
4. Call `EVP_PKEY_keygen` by passing the context and address to `key_pair` 
5. Free the context

{:.note}
`EVP_PKEY_CTX_set_rsa_keygen_bits` sets the key size (1024 bits for this task). The public exponent is implicitly set to `65537` by OpenSSL, matching the standard recommendation. The resulting `key_pair` object contains both halves of the key pair.

## Using the Keys

Afterwards, we can use the private or public key for encryption or decryption. However, there are some terminologies that you need to know.

### Encrypt and Decrypt

Encryption of a message with a <span style="color:#f77729;"><b>public key</b></span> is normally labeled as `encrypt` in the API, and <span style="color:#f77729;"><b>decryption</b></span> with a private key is labeled as `decrypt`. The `common.c` wrappers `rsa_encrypt_block` and `rsa_decrypt_block` handle the OAEP padding internally:

```c
unsigned char *rsa_encrypt_block(EVP_PKEY *pub_key,
                                 const unsigned char *plain, size_t plain_len,
                                 size_t *out_len, int use_oaep)

unsigned char *rsa_decrypt_block(EVP_PKEY *priv_key,
                                 const unsigned char *cipher, size_t cipher_len,
                                 size_t *out_len, int use_oaep)
```

Note that the `private_key` object passed here is the same one generated above; OpenSSL uses the public component for encryption and the private component for decryption automatically. It is called `key_pair` in this file.

For `encrypt` with `OAEP` padding, the length of plaintext that fits in a single RSA block depends on the <span style="color:#f77729;"><b>padding</b></span> scheme:

- With a 1024-bit RSA key, the block size is 128 bytes.
- OAEP with SHA-256 has 66 bytes of padding overhead, leaving at most **62 bytes** of plaintext per block.
- PKCS1v15 has 11 bytes of minimum overhead, leaving at most **117 bytes** of plaintext per block.

You will encounter these limits again in Programming Assignment 2.
{:.important}

### Sign and Verify

If you want to <span style="color:#f77729;"><b>encrypt</b></span> a message using the <span style="color:#f77729;"><b>private key</b></span>, the operation is called `sign`, and the output is a <span style="color:#f77729;"><b>signature</b></span>. The corresponding operation to verify (i.e., decrypt) using the public key is called `verify`.

- Conversely, if you want to <span style="color:#f77729;"><b>decrypt</b></span> the output signature, the keyword in the API is `verify`. Verification succeeds silently; a failure raises an error.
- But don't be fooled! Verification _is a decryption_, and signing _is an encryption_. They just have different names tied to their _purpose_.

The `sign_message_pss` wrapper from `common.c` computes an RSA-PSS signature with SHA-256:

```c
unsigned char *sign_message_pss(EVP_PKEY *priv_key,
                                const unsigned char *msg, size_t msg_len,
                                size_t *sig_len)
```

Verification uses `EVP_DigestVerify` from OpenSSL liberary directly (since `verify_message_pss` in `common.c` requires an X.509 certificate, which we don't have here. You will use it in Programming Assignment 2 instead):

```c
EVP_MD_CTX *md_ctx = EVP_MD_CTX_new();
EVP_PKEY_CTX *pkey_ctx = NULL;

EVP_DigestVerifyInit(md_ctx, &pkey_ctx, EVP_sha256(), NULL, key_pair);
EVP_PKEY_CTX_set_rsa_padding(pkey_ctx, RSA_PKCS1_PSS_PADDING);
EVP_PKEY_CTX_set_rsa_pss_saltlen(pkey_ctx, RSA_PSS_SALTLEN_MAX);
EVP_DigestVerifyUpdate(md_ctx, file_data, (size_t)file_len);
int ok = (EVP_DigestVerifyFinal(md_ctx, signature, sig_len) == 1);
```

Note that `sign_message_pss` hashes the data internally before signing. You pass the raw `file_data`, not the digest. The library computes SHA-256 internally before applying RSA-PSS. This means the input data length can be <span style="color:#f77729;"><b>arbitrarily long</b></span>.
{:.note}

We use a <span style="color:#f77729;"><b>different</b></span> padding scheme for signatures (`PSS`) rather than `OAEP` because of the nature of the operation. You may read more about it [here](https://www.cryptosys.net/pki/manpki/pki_rsaschemes.html), but it is out of our syllabus.

### Salting in Hashing

{:.info}
A salt is a random value added to the input (e.g., a password) before hashing.

It ensures that the same input will produce **different** hash outputs. This prevents attackers from using precomputed hash tables (rainbow tables) to reverse-engineer the hashed values.

Characteristics of Salt:
* **Uniqueness**: Each input (e.g., each password) should have a unique salt to prevent identical inputs from producing the same hash.
* **Randomness**: The salt should be randomly generated to ensure unpredictability.
* Non-**Secret**: The salt does <span class="orange-bold">not</span> need to be kept secret. It is typically stored **alongside** the hash in a database. From Lab 3 (TOCTOU), you have seen how salt is kept alongside the hashed password in `/etc/shadow`.

## Creating a Digest

### Task 3-2

`TASK 3-2:` Implement `compute_sha256` function. In order to create a message digest, use `EVP_DigestInit_ex`, `EVP_DigestUpdate`, and `EVP_DigestFinal_ex` to compute a SHA-256 hash.
{:.task}

```c
/**
 * TASK 3-2: Compute SHA-256 hash of data.
 * Returns a 32-byte buffer (caller must free).
 */
static unsigned char *compute_sha256(const unsigned char *data, size_t len, unsigned int *digest_len)
{
    unsigned char *digest = malloc(EVP_MAX_MD_SIZE);

    // TODO: Implement the computation of SHA-256 here

    return digest; /* 32 bytes for SHA-256 */

    /* END OF TASK 3-2 */
}
```

You can use the following steps:

1. Create a new context using `EVP_MD_CTX_new()`
2. Initialise using `EVP_DigestInit_ex` with the context and `EVP_sha256()` as arguments
3. Update the digest using `EVP_DigestUpdate`
4. Produce final digest using `EVP_DigestFinal_ex` by passing `digest` and its length
5. Finally, free the context created in (1)

The resulting `digest_len` is always 32 bytes for SHA-256, regardless of input size. The Base64 representation printed to stdout looks like:

```sh
Original hash bytes: Aw3B+TbDQVr/PzNXFjUVGQ00eijnWOH3F9F7rkU1QcmYQ==
Length of hash bytes of original_files/shorttext.txt is 32
```

{:.highlight}
> Confirm that the length of the digest always the same (32 bytes), regardless of the length of the input data.

### Task 3-3 to 3-6

`TASK 3-3:` Compute SHA-256 hash of the file data by calling `compute_sha256`.
{:.task}

```c
    /*
     * Task 3-3: Compute SHA-256 hash of the file data.
     *
     * Python:
     *   hash_function = hashes.Hash(hashes.SHA256())
     *   hash_function.update(file_data)
     *   message_digest_bytes = hash_function.finalize()
     */
    // TODO: Task 3-3
    unsigned int digest_len = 0;
    unsigned char *digest = 0; // modify this assignment
    /* END OF TASK 3-3 */
```

`TASK 3-4:` Encrypt the digest with the public key (inside `key_pair`, OpenSSL combined both) by calling `rsa_encrypt_block`.
{:.task}

```c
    /*
     * Task 3-4: Encrypt the digest with the PUBLIC key (OAEP padding)
     *
     * We use rsa_encrypt_block with use_oaep=1.
     */
    /* Note: in OpenSSL 3.x, we can use the private EVP_PKEY for public operations too,
       but conceptually we're using the public key here. */
    size_t enc_len = 0;
    // TODO: Task 3-4
    unsigned char *encrypted = 0; // modify this assignment
    /* END OF TASK 3-4*/
```


`TASK 3-5:` Decrypt the digest with the private key (inside `key_pair`) by calling `rsa_decrypt_block`.
{:.task}

```c
    /*
     * Task 3-5: Decrypt the digest back.
     */
    size_t dec_len = 0;
    // TODO: Task 3-5
    unsigned char *decrypted = 0; // modify this assignment
    /* END OF TASK 3-5 */
```

`TASK 3-6:` Sign the data with `key_pair` using `sign_message_pss`. It will automatically extract the private key from that.
{:.task}

```c
    /*
     * Task 3-6: Sign the data with the private key (RSA-PSS, SHA-256).
     *
     * Note: sign() hashes the data internally so you should pass the raw file_data,
     * not the digest. The library computes SHA-256(file_data) then signs that.
     */
    size_t sig_len = 0;
    // TODO: Task 3-6
    unsigned char *signature = 0; // modify this assignment
```

## The `main` function

If you scroll to `main` you will find these four function calls:

```c
enc_digest("original_files/shorttext.txt");
enc_digest("original_files/longtext.txt");
sign_digest("original_files/shorttext.txt");
sign_digest("original_files/longtext.txt");
```

Here we test encryption of a digest using the public key (`enc_digest`) and signing of a digest using the private key (`sign_digest`), each tested with two files of differing length. Study the output carefully and head to eDimension to answer the rest of the questionnaires.

# Summary

This lab introduced the three cryptographic building blocks that form the foundation of Programming Assignment 2.

**Part 1** covered symmetric encryption using AES-128-CBC combined with HMAC-SHA256 via `session_encrypt` and `session_decrypt` from `common.c`. You generated a session key, encrypted plaintext into a token with the layout `IV || ciphertext || HMAC`, and <span class="orange-bold">verified</span> that tampering causes `session_decrypt` to reject the token. In PA2 CP2, this is exactly how file data is protected in transit after the session key exchange.

**Part 2** covered block cipher modes using 3DES directly through the raw OpenSSL EVP API, without the `common.c` wrappers. You implemented PKCS7 padding manually and observed the structural difference between ECB and CBC outputs on real images. ECB leaks patterns because identical plaintext blocks always produce identical ciphertext; CBC chains each block to the previous ciphertext and destroys those patterns. You will not use 3DES in PA2, but the padding logic and the EVP cipher API pattern (`EncryptInit` / `EncryptUpdate` / `EncryptFinal`) are the same ones `session_encrypt` uses internally with AES.

**Part 3** covered RSA key generation, SHA-256 digests, and the two distinct uses of RSA: encrypting a digest with the public key (`rsa_encrypt_block`) and signing raw data with the private key (`sign_message_pss`). You saw that signing is faster than chunked RSA encryption because the data is hashed to a fixed 32-byte digest first, and that RSA-PSS is the correct padding scheme for signatures while OAEP is used for encryption.

## What will be used in Programming Assignment 2

| Lab concept | PA2 usage |
|---|---|
| `generate_session_key` / `session_encrypt` / `session_decrypt` | CP2: encrypting the file payload after key exchange |
| `rsa_encrypt_block` / `rsa_decrypt_block` (OAEP) | CP1: direct file encryption; CP2: encrypting the session key |
| `sign_message_pss` | All tasks: client authenticates itself to the server |
| `verify_message_pss` | All tasks: server verifies the client signature (uses X.509 cert, not a raw key pair as in this lab) |
| `load_cert_file` / `load_cert_bytes` / `verify_server_cert` | All tasks: client verifies the server's identity before sending anything |
| PKCS7 padding | Applied automatically inside `session_encrypt`; you will not call it directly |

The key difference in PA2 is that trust is established through **X.509 certificates** rather than a *pre-shared key pair* like in this lab. The server sends its signed certificate over the network; the client verifies it against the CA certificate shipped with the starter code, then extracts the server's public key from the verified cert. Only after that trust step does any encryption or signing take place. The `common.c` functions you used in this lab are the same ones that implement all of that.