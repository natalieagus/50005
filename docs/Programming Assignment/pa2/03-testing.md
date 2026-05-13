---
layout: default
permalink: /pa2/part-test
title: Testing
description: Implement unit tests and integration tests for Secure FTP
parent: Programming Assignment 2
grand_parent: Programming Assignment
nav_order: 3
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

# Testing
{: .no_toc}

A protocol implementation that "seems to work" on a single happy-path run is <span class="orange-bold">not</span> finished. Most network-security bugs (off-by-one on a length prefix, the wrong byte order, a truncated read, a flipped HMAC bit you never notice because the test only sends ASCII) only show up when you exercise the code with real inputs and real failure modes. It goes without saying that it is important to test your program, just like what you did in PA1.

Similarly, we need to have these tests in PA2:

```text
Unit tests:
Test one small function directly.

Integration tests:
Run the whole client and server, transfer a real file,
and verify the result byte-for-byte.
```

Example:

```text
Unit test:
Call int_to_bytes(1, buf) and check that buf is exactly
0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x01.

Integration test:
Start ./ServerWithoutSecurity in the background, run
./ClientWithoutSecurity, send files/cbc.bmp, and check that
recv_files/recv_cbc.bmp matches the original byte-for-byte.
```

{:.important}
You are to **design your code** so that important logic can be tested <span class="orange-bold">without</span> always running the full client and server.

## Project Structure

Your end project should follow this general structure:

```text
[PROJECT_DIR]/
  source/
    common.h
    common.c
    ClientWithoutSecurity.c
    ServerWithoutSecurity.c
    ClientWithSecurityAP.c
    ServerWithSecurityAP.c
    ClientWithSecurityCP1.c
    ServerWithSecurityCP1.c
    ClientWithSecurityCP2.c
    ServerWithSecurityCP2.c
    auth/
      cacsertificate.crt
      generate_keys.py

  tests/
    unit/
      test_int_bytes.c
      test_socket.c
      test_rsa_sign_verify.c
      test_session_crypto.c
      test_x509.c

    integration/
      _lib.sh
      test_exit.sh
      test_transfer.sh
      test_transfer_binary.sh
      test_multi_transfer.sh
      test_ap_handshake.sh
      test_cp1_transfer.sh
      test_cp2_transfer.sh

    unity/
      unity.c
      unity.h
      unity_internals.h

  scripts/
    gen_unit_tests.sh

  prompts/
    generate-unit-tests.md

  AGENTS.md
  Makefile
  README.md
```

Here are the main 3 locations you should work on:

```text
source/   contains your program logic.
tests/    contains code that checks whether your logic works.
prompts/  contains prompt templates for AI-assisted test drafting.
```

The starter code contains *everything* you need to run the unit tests and integration tests on `ClientWithoutSecurity` and `ServerWithoutSecurity`. The crypto helpers in `common.c` are already implemented and ready to test. As you build the AP, CP1, and CP2 variants, add your own tests in the same layout.


## Code Placement Philosophy

As usual, you should <span class="orange-bold">not</span> create one gigantic `main()` that does everything:

```c
int main(int argc, char *argv[]) {
    // open socket
    // send nonce
    // receive signature
    // load CA cert
    // verify cert
    // verify signature
    // encrypt files
    // close
}
```

Instead, you should pull all logic out of `main()` into pure functions that take inputs and return outputs:

```c
// source/auth_proto.h
#ifndef AUTH_PROTO_H
#define AUTH_PROTO_H

#include "common.h"

/* Build the bytes the client should send for an AP nonce challenge:
 *   [MSG_AUTH=3][nonce_len][nonce_bytes]
 * Returns a newly malloc'd buffer; writes its length to *out_len.
 */
unsigned char *build_ap_challenge(const unsigned char *nonce, size_t nonce_len,
                                  size_t *out_len);

/* Parse an AP response: [signature_len][signature][cert_len][cert].
 * On success, returns 0 and fills *sig, *sig_len, *cert.
 * On malformed input, returns -1.
 */
int parse_ap_response(const unsigned char *buf, size_t buf_len,
                      unsigned char **sig, size_t *sig_len,
                      X509 **cert);

#endif
```

With that design, `main()` is short and each helper is unit testable:

```c
// source/ClientWithSecurityAP.c
#include "auth_proto.h"

int main(int argc, char *argv[]) {
    /* connect, send challenge, read response, verify, send files */
    return run_ap_client(argc, argv);
}
```

```c
// tests/unit/test_auth_proto.c
TEST_ASSERT_NOT_NULL(build_ap_challenge(nonce, 16, &n));
TEST_ASSERT_EQUAL_INT(0, parse_ap_response(buf, len, &sig, &slen, &cert));
```


{:.note}
You *do not* have to extract every helper, please do not overdo it. You should extract any function that has logic you would feel uncomfortable shipping without a test.


## Unit Tests

{:.note}
Unit tests should focus on small, predictable functions that do not depend on a running server.

These are good unit test targets, drawn from `common.c` plus helpers you may extract while building the secure variants:

```text
common.c (given)
  int_to_bytes()
  bytes_to_int()
  read_bytes()
  send_all()
  send_int()
  load_private_key()
  load_cert_file()
  load_cert_bytes()
  verify_server_cert()
  sign_message_pss()
  verify_message_pss()
  rsa_encrypt_block()
  rsa_decrypt_block()
  generate_session_key()
  session_encrypt()
  session_decrypt()

your own helpers (examples; create the .h and .c as needed)
  auth_proto.c
    build_ap_challenge()
    parse_ap_response()
  wire.c
    pack_file_message()
    parse_file_message()
  session.c
    derive_keys_from_session()
```

These are good because they usually take **clear** input, produce **clear** output, and do not depend on a running TCP connection. The two unit-test files are given as sample  in the starter, `test_int_bytes.c` and `test_socket.c` for your reference.

{:.note}
Some functionalities are also repeatable, can be used in all parts: AP, CP1, and CP2 so it's good to separate the concerns.


### Example Unit Test

Given this header from the starter:

```c
// source/common.h
#define INT_BYTES 8

void     int_to_bytes(uint64_t x, unsigned char buf[INT_BYTES]);
uint64_t bytes_to_int(const unsigned char buf[INT_BYTES]);
```

A unit test looks like this:

```c
// tests/unit/test_int_bytes.c
#include "unity.h"
#include "common.h"

void setUp(void) {}
void tearDown(void) {}

void test_int_to_bytes_zero(void) {
    unsigned char buf[INT_BYTES];
    unsigned char zero[INT_BYTES] = {0, 0, 0, 0, 0, 0, 0, 0};
    int_to_bytes(0, buf);
    TEST_ASSERT_EQUAL_MEMORY(zero, buf, INT_BYTES);
}

void test_int_to_bytes_one_is_big_endian(void) {
    unsigned char buf[INT_BYTES];
    unsigned char expected[INT_BYTES] = {0, 0, 0, 0, 0, 0, 0, 1};
    int_to_bytes(1, buf);
    TEST_ASSERT_EQUAL_MEMORY(expected, buf, INT_BYTES);
}

void test_roundtrip(void) {
    const uint64_t values[] = {0, 1, 255, 256, UINT64_MAX};
    for (size_t i = 0; i < sizeof(values) / sizeof(values[0]); i++) {
        unsigned char buf[INT_BYTES];
        int_to_bytes(values[i], buf);
        TEST_ASSERT_EQUAL_UINT64(values[i], bytes_to_int(buf));
    }
}

int main(void) {
    UNITY_BEGIN();
    RUN_TEST(test_int_to_bytes_zero);
    RUN_TEST(test_int_to_bytes_one_is_big_endian);
    RUN_TEST(test_roundtrip);
    return UNITY_END();
}
```

You can then compile and run it with:

```bash
make unit
```

The Makefile auto-discovers any `tests/unit/test_*.c` you add.


## Integration Tests

Some behavior is hard to unit test because it spans two processes (client and server) and a real socket. These are usually better for integration tests (implemented as bash scripts).

```text
the wire protocol end-to-end (file actually arrives intact)
the AP handshake (server proves identity to client)
the CP1 encrypt-then-send flow (server decrypts correctly)
the CP2 session key exchange and bulk transfer
graceful exit (client sends -1 and both processes terminate)
binary files (no off-by-one or null-termination bug)
```

### Example integration test

An integration test runs your compiled client and server.

```bash
#!/usr/bin/env bash
# tests/integration/test_transfer.sh
set -euo pipefail

source ./tests/integration/_lib.sh
trap cleanup EXIT

SRC="files/file.txt"
DST="recv_files/recv_file.txt"

reset_recv_files
start_server

printf '%s\n-1\n' "$SRC" | timeout 10s ./ClientWithoutSecurity \
  "$PORT" localhost > "$LOG_DIR/client.log" 2>&1

# Wait for server to exit cleanly.
for i in $(seq 1 20); do
  if ! kill -0 "$SERVER_PID" 2>/dev/null; then break; fi
  sleep 0.1
done

if ! cmp -s "$SRC" "$DST"; then
  echo "FAIL: $DST differs from $SRC"
  dump_logs
  exit 1
fi

echo "PASS: text file transferred byte-for-byte"
```

Save this as:

```text
tests/integration/test_transfer.sh
```

And then you compile and run using:

```bash
make integration
```

#### Adding `timeout`
The `timeout` prevents the test from hanging forever if your client or server gets stuck. The `_lib.sh` helpers (`start_server`, `reset_recv_files`, `dump_logs`, `cleanup`) handle the process lifecycle so each script stays focused on what it is actually asserting.

#### `cmp` vs `diff`
{:.note}
`cmp -s` is binary-safe and exits non-zero on the first differing byte. This is preferable to `diff` when checking file integrity, because `diff` skips or summarizes binary content on some systems. However, the decision is up to you.


## Testing the application layer protocol (wire protocol)

{:.highlight-title}
> Wire protocol
>
> A wire protocol is a set of rules defining how data is **formatted**, **encoded**, and **transmitted** between applications or systems over a network.

Functions in `common.c` like `read_bytes`, `send_all`, and `send_int` take a socket file descriptor. You can test them without spinning up a server by giving them a `socketpair`.

{:.note-title}
> Socketpair
> 
> A `socketpair` is a pair of connected sockets in the same process (duh!). You write to one end and read from the other. This isolates the function under test from TCP, the loopback interface, and port allocation.


Here's an example:

```c
// tests/unit/test_socket.c
#include "unity.h"
#include "common.h"
#include <sys/socket.h>
#include <signal.h>

static int fds[2];

void setUp(void) {
    signal(SIGPIPE, SIG_IGN);
    TEST_ASSERT_EQUAL_INT(0, socketpair(AF_UNIX, SOCK_STREAM, 0, fds));
}

void tearDown(void) {
    close(fds[0]);
    close(fds[1]);
}

void test_read_bytes_loops_across_chunks(void) {
    /* Send 9 bytes in three separate sends. read_bytes must keep
       calling recv() until it has all 9. */
    send(fds[1], "foo", 3, 0);
    send(fds[1], "bar", 3, 0);
    send(fds[1], "baz", 3, 0);

    unsigned char *got = read_bytes(fds[0], 9);
    TEST_ASSERT_NOT_NULL(got);
    TEST_ASSERT_EQUAL_MEMORY("foobarbaz", got, 9);
    free(got);
}

void test_read_bytes_returns_null_on_early_close(void) {
    /* Peer closes after 5 bytes. Asking for 100 must return NULL. */
    send(fds[1], "short", 5, 0);
    close(fds[1]);

    unsigned char *got = read_bytes(fds[0], 100);
    TEST_ASSERT_NULL(got);
}
```

A few things worth noting about this style:

```text
signal(SIGPIPE, SIG_IGN)
  Without this, writing to a closed peer kills the test process.

socketpair(AF_UNIX, ...)
  Uses an in-kernel pipe, not TCP. No port number, no race.

free(got)
  read_bytes mallocs its return value. Tests must free it,
  otherwise leak-checking tools will flag the test itself.
```


## Testing crypto helpers

**Crypto round trips** are the cleanest possible unit tests: if `decrypt(encrypt(x)) == x`, your code is at least self-consistent. They run in milliseconds and they catch the most common mistakes (wrong key length, wrong IV handling, padding errors).

Recommended structure for crypto helpers in `common.c`:

```text
Unit test:
  generate a key in-memory
  encrypt a known plaintext
  decrypt and assert it matches the original
  flip one bit in the ciphertext and assert decrypt fails
  flip one bit in the HMAC and assert decrypt fails

Integration test:
  run the full CP1 or CP2 client and server
  transfer a real file
  check the file arrives byte-for-byte
```

Example unit test for the session-key path:

```c
// tests/unit/test_session_crypto.c
#include "unity.h"
#include "common.h"

void setUp(void) {}
void tearDown(void) {}

void test_session_roundtrip(void) {
    unsigned char key[SESSION_KEY_LEN];
    TEST_ASSERT_EQUAL_INT(0, generate_session_key(key));

    const unsigned char plain[] = "hello secure world";
    size_t ct_len, pt_len;

    unsigned char *ct = session_encrypt(key, plain, sizeof(plain) - 1, &ct_len);
    TEST_ASSERT_NOT_NULL(ct);

    unsigned char *pt = session_decrypt(key, ct, ct_len, &pt_len);
    TEST_ASSERT_NOT_NULL(pt);
    TEST_ASSERT_EQUAL_UINT(sizeof(plain) - 1, (unsigned)pt_len);
    TEST_ASSERT_EQUAL_MEMORY(plain, pt, pt_len);

    free(ct);
    free(pt);
}

void test_session_decrypt_rejects_tampered_hmac(void) {
    unsigned char key[SESSION_KEY_LEN];
    generate_session_key(key);

    size_t ct_len;
    unsigned char *ct = session_encrypt(key,
        (const unsigned char *)"data", 4, &ct_len);

    /* Flip the last byte (inside the HMAC tag). */
    ct[ct_len - 1] ^= 0x01;

    size_t pt_len;
    unsigned char *pt = session_decrypt(key, ct, ct_len, &pt_len);
    TEST_ASSERT_NULL(pt);

    free(ct);
}
```

{:.important}
A test that *decrypt* fails on a flipped bit is more valuable than three tests that decrypt succeeds on slightly different plaintexts because it is the test that catches a developer <span class="orange-bold">accidentally</span> skipping the HMAC check.


## Testing X.509 certificate verification

The starter code includes `source/auth/cacsertificate.crt` and a helper `generate_keys.py` that mints server keys and CSRs. You can use these for unit tests on `verify_server_cert`:

```text
Unit test:
  Load a certificate that is signed by the CA. Assert verify succeeds.
  Load a certificate that is self-signed. Assert verify fails.
  Pass a non-existent CA path. Assert verify fails.
```

**Recommended approach**: before running the tests, generate one CA-signed cert and one self-signed cert and commit them under `tests/fixtures/`. Then:

```c
// tests/unit/test_x509.c
void test_verify_server_cert_accepts_ca_signed(void) {
    X509 *cert = load_cert_file("tests/fixtures/server_signed.crt");
    TEST_ASSERT_NOT_NULL(cert);
    TEST_ASSERT_EQUAL_INT(1,
        verify_server_cert(cert, "source/auth/cacsertificate.crt"));
    X509_free(cert);
}

void test_verify_server_cert_rejects_self_signed(void) {
    X509 *cert = load_cert_file("tests/fixtures/server_selfsigned.crt");
    TEST_ASSERT_NOT_NULL(cert);
    TEST_ASSERT_EQUAL_INT(0,
        verify_server_cert(cert, "source/auth/cacsertificate.crt"));
    X509_free(cert);
}
```

{:.note-title}
> Beginner mistake
> 
> Do <span class="orange-bold">not</span> commit the matching private keys. Only commit the public certificates. Keys are generated locally per developer.


## Testing AP, CP1, CP2 handshakes

The protocol variants are by definition <span class="orange-bold">multi-step state machines</span> that span two processes. The best test is the **integration** test following these: start the matching server, run the matching client, transfer a file, and check the result.

Here's a recommended pattern:

```bash
# tests/integration/test_ap_handshake.sh
set -euo pipefail
source ./tests/integration/_lib.sh
trap cleanup EXIT

# Start the AP server in the background.
./ServerWithSecurityAP "$PORT" localhost > "$LOG_DIR/server.log" 2>&1 &
SERVER_PID=$!
# (port-wait code omitted; see _lib.sh)

printf 'files/file.txt\n-1\n' | timeout 10s ./ClientWithSecurityAP \
  "$PORT" localhost > "$LOG_DIR/client.log" 2>&1

# Wait for server to exit, then check the transferred file.
for i in $(seq 1 20); do
  if ! kill -0 "$SERVER_PID" 2>/dev/null; then break; fi
  sleep 0.1
done

if ! cmp -s files/file.txt recv_files/recv_file.txt; then
  echo "FAIL: file did not survive AP transfer"
  dump_logs
  exit 1
fi

# Also check the server actually performed the handshake.
if ! grep -q "Authentication successful" "$LOG_DIR/server.log"; then
  echo "FAIL: server log shows no successful authentication"
  dump_logs
  exit 1
fi

echo "PASS: AP handshake + file transfer"
```

A passing test here proves:

```text
1. The server loaded its private key.
2. The server produced a signature over the client's nonce.
3. The client loaded the CA cert and verified the server's certificate.
4. The client verified the signature.
5. The file transferred byte-for-byte after authentication.
```

{:.note}
Now, we know that writing tests are tedious in itself, so you can offload it to an AI. Thinking about proper test cases (including edge cases) are valuable and you should take the time to do so. Think about end-to-end usage and where things might go wrong. You might do this better than an AI because *you know your project best*. You own it.


## Using the AI unit-test generator


{:.highlight}
This project include an optional AI-assisted unit-test generator. The generator is <span class="orange-bold">not</span> automatic. It runs only when you type a command. If there's an agent, it will instruct the agent what to do, otherwise it will print prompts to `stdout` for you to process.

You can call them using:

```bash
make ai-unit-tests MODULE=[module-to-test]
```

If an agent is set, it should read `AGENTS.md` to understand the project rules. The agent is insructed such that it may only create or modify files under:

```text
tests/unit/
```

Your agent most not modify anything else, otherwise it will be a headache to track what it changed.


### `git add && git commit` first then `git diff`

This is very **important**. You should add and commit your changes first before asking the agents to modify your test files.




### AI workflow

When you add a new module or function:

```text
1. Write the function in source/.
2. Declare it in a matching header (source/<name>.h).
3. Run:

   make ai-unit-tests MODULE=<name>

4. Review the generated file in tests/unit/.
5. Run:

   make unit

6. Fix your code or tests.
7. Commit only tests you understand.
```

Useful commands:

```bash
make ai-unit-tests MODULE=common
make ai-unit-tests MODULE=auth_proto
make unit
git diff
```



### Suggested AI prompt

If your tool asks for a prompt, use something like this:

```text
Generate Unity unit tests for the new or modified functions in this C project.

Rules:
1. Only create or edit files under tests/unit/.
2. Do not modify source/.
3. Use the public function declarations in source/<name>.h.
4. Write tests for normal cases, edge cases, and invalid input.
5. For socket helpers, use socketpair() in setUp() rather than a real TCP listener.
6. For crypto helpers, generate keys in-memory or load fixtures from source/auth/.
7. Do not write tests that bind to a TCP port, spawn a server, call fork(), or read from stdin.
8. Mark generated tests with a comment saying they are AI-generated drafts.
9. Keep the tests simple and readable.
```

For a specific file:

```text
Generate Unity unit tests for the helpers in source/common.c using source/common.h.
Create or update tests/unit/test_session_crypto.c, covering only the
session_encrypt / session_decrypt / generate_session_key trio.
Cover empty plaintext, single-block plaintext, multi-block plaintext,
and a flipped-bit tamper that must cause session_decrypt to return NULL.
Do not modify production code.
```


## Summary of testing commands

```bash
make unit
```


```bash
make integration
```


```bash
make test
```


```bash
make ai-unit-tests MODULE=name
```



The integration test server binds to port 14321 by default. Override it if that port is in use:

```bash
PA2_TEST_PORT=15555 make integration
```




## Summary

As you know from PA1, if a function is hard to test, that is often a sign that it is doing too much. You should split large functions into smaller functions but do not overdo it.

No matter what, keep `main()` small and keep operating-system behavior **separate** from parsing, formatting, and crypto logic.
- Use unit tests for small logic and crypto round trips.
- Use integration tests for full-program behavior and end-to-end file transfer.

Remember that although AI may help draft tests, but you must understand, review, and run every test you submit.
