---
layout: default
title: 50.003 x 50.005 CoreStack Challenge
permalink: /pa/tetrish
nav_order: 8
parent: Programming Assignment
---

- TOC
  {:toc}

**50.005 Computer System Engineering**
<br>
Information Systems Technology and Design
<br>
Singapore University of Technology and Design
<br>
**Natalie Agus (Summer 2026)**

# 50.003 x 50.005 CoreStack Challenge

{:.highlight}
**CoreStack** is a cross-course alternative track jointly offered by 50.005 and 50.003. A group that opts in builds one shared systems core (the `corestack` library: shell, concurrency utilities, IPC primitives, secure session, logging client) and uses it to deliver **two** separate applications. The 50.005 application is `tetriSH`. The 50.003 application is of the group's choosing (web app, terminal app, SSH-based service, replay viewer, anything reasonable) and must depend on the same `corestack` in a non-trivial way.

{:.note}
It is up to you to define what the `core` is. This should be part of your proposal in Week 3.

**This handout describes the 50.005 side only.** The 50.003 side is documented separately in the 50.003 (ask your instructor). Read both before opting in.

## tetriSH

{:no-toc}

{:.highlight}
tetriSH is the 50.005 half of CoreStack. It replaces PA1 and PA2 for accepted groups.

This is not a bonus-mark assignment. tetriSH replaces:

1. **PA1: shell, daemon, process management, signals, and basic IPC**
2. **PA2: authenticated and confidential client-server communication**

A group that takes tetriSH does <span class="orange-bold">not</span> need to separately submit PA1 and PA2.

Instead, the group submits one integrated system that demonstrates the required PA1 and PA2 competencies through the tetriSH architecture. A polished game interface alone is <span class="orange-bold">not</span> sufficient, and this is an assessment on systems engineering. You will be interviewed and asked to implement live features during the checkoff.

{:.important}
The project component remains <span class="orange-bold">capped</span> at the same mark as the normal PA1 + PA2 path. There are **no additional bonus marks** for attempting tetriSH. However, there's a **prize** for the top 2 groups.

Sections below are tagged so you can tell what is required and what is yours to design.

{:.important-title}

> **Required**
>
> This is a hard requirement. You will be graded on whether your code does this and no deviation is allowed.

{:.highlight-title}

> **Open design**
>
> A decision you must make and document neatly. We give you example approaches and you should pick, invent your own, but defend it in your `README` and at the live demo Q&A.

### tl;dr

Indicate your interest to embark on this track by Week 3, Monday 6PM to **both** your 50.005 instructor and your 50.003 instructor. All three group members must be enrolled in both 50.003 and 50.005 in the current term. You may still withdraw by the end of Week 5 Friday, 6PM, but withdrawal is all-or-nothing across both courses.

Once you're committed to this track, here's the rule (50.005 side):

- **To avoid zero:** baseline MVP must work
- **To complete tetriSH properly**: baseline MVP + Battle Royale must work
- **To get good marks**: system must be correct, clean, documented, and understood, also pass live QnA and live patching
- **To win prize**: full required system + strong engineering quality + flawless Q&A + shippable bundle that the cohort can play on + a git history that proves your team actually built this

Refer to the individual sections below for clarifications. The 50.003 side is graded separately by the 50.003 instructor on the 50.003 schedule.

## Overview

`tetriSH` is a single integrated project that combines [PA1]({{ site.baseurl }}/pa1/intro) (shell + daemon) and [PA2]({{ site.baseurl }}/pa2/intro-c) (authenticated, confidential client-server protocol) into one <span class="orange-bold">terminal-based</span> Battle-Royale Tetris system written entirely in C.

You should deliver at least these **five binaries** and **three libraries**:

| Component        | Type    | Role                                                        |
| ---------------- | ------- | ----------------------------------------------------------- |
| `tetrish`        | binary  | Interactive shell, reads `.tetrishrc`, launches the daemons |
| `tetrisd`        | binary  | Concurrent game server (the body of the system)             |
| `tetrislogd`     | binary  | Dedicated logger daemon (separate process)                  |
| `tetrisctl`      | binary  | Admin CLI for the running game daemon                       |
| `tetrisu`        | binary  | Terminal-based game client (the user-facing program)        |
| `libtetrissh`    | library | Secure session (cert auth, RSA-wrapped AES, framing)        |
| `libhtttp`       | library | HTTTP protocol parser and serialiser                        |
| `libtetrisbrain` | library | Tetris game logic (board, pieces, gravity, line clear)      |

You may complete this challenge in **groups of 3 only**. This is an _opt-in replacement_ for PA1 + PA2. Selection is competitive and is described in the later section below.

### Project Constraints

This entire project **must** solely be terminal-based and it must be written in C.

## Layering

{:.important-title}

> **Required**
>
> The system has exactly **three** layers above the kernel and you should implement the upper two.

```
+---------------------------------------------+
|  Application: HTTTP messages                 |
|  (HyperText Tetris Transfer Protocol)        |
+---------------------------------------------+
|  Secure session                              |
|  (cert auth, RSA-wrapped AES, framed)        |
+---------------------------------------------+
|  Transport: TCP via POSIX sockets            |
+---------------------------------------------+
```

{:.note}
TCP is provided by the kernel through the socket API. We are not here to reimplement reliability, ordering, retransmission, or congestion control as it is out of scope and not the point of this course.

## Binaries

### `tetrish`: the shell

{:.important-title}

> **Required**
>
> All PA1 shell behaviour: REPL, `fork()` plus `execvp()`, builtins (`cd, help, exit, usage, env, setenv, unsetenv`), `.tetrishrc` execution on startup, background process tracking and spawning using `sys`, `dspawn`, `dcheck`, no crashes on bad input.

`tetrish` is the <span class="orange-bold">entry</span> point. From inside `tetrish`, the user can launch `tetrisd` and `tetrislogd` in the background, run `tetrisctl` queries (server side), or start `tetrisu` (client side) to play.

{:.highlight-title}

> **Open design**
>
> Whether you add `tetrish`-specific builtins (e.g. `tetris-status` as a wrapper around `tetrisctl status`, or `tetris-up` as a one-shot to launch the daemon and the logger together) is your call.

### `tetrisd`: the game daemon

{:.important-title}

> **Required**
>
> The daemon must:
>
> - Detach from the controlling terminal when launched in background from `tetrish`
> - Bind to the TCP port configured in `.tetrishrc`
> - Accept multiple concurrent clients
> - Establish a secure session (via `libtetrissh`) with each client before any HTTTP traffic
> - Parse and serialise HTTTP messages (via `libhtttp`)
> - Maintain rooms with multiple players, run game logic (via `libtetrisbrain`), broadcast state
> - Handle `SIGTERM` (graceful shutdown), `SIGHUP` (reload config), `SIGUSR1` (dump state to log)
> - Ignore `SIGPIPE`; detect broken connections via `write()` returning `EPIPE`
> - Forward all log records to `tetrislogd` over IPC, with non-blocking enqueue from game-critical threads (see Section 7)
> - Expose a control plane to `tetrisctl` (see Section 6)

`tetrisd` is the **process** that runs the **server**. It contains threads. The threads inside `tetrisd` should orchestrate the listeners, per-client handlers, per-room game ticks, the signal handler, and the channel out to `tetrislogd`. The libraries (`libtetrissh`, `libhtttp`, `libtetrisbrain`) should provide the heavy lifting.

{:.highlight-title}

> **Open design**
>
> The internal architecture of `tetrisd` is yours. We have three reference design for your perusal.

#### Thread-per-client, room-ticker per room

```
   listener_thread  ---->  client_thread (one per client)
                              |
                              v
                          room_t (mutex-protected)
                              ^
                              |
                          ticker_thread (one per active room)

   logshipper_thread (one)    signal_thread (one)
   ctl_listener_thread (one)
```

- Listener `accept()`s, hands the `fd` to a **fresh** client thread.
- Client thread calls into `libtetrissh` to do the handshake, then loops on HTTTP requests parsed by `libhtttp`.
- Each active room runs a ticker thread at fixed Hz that mutates board state (via `libtetrisbrain`) under the room mutex and enqueues `STATE` broadcasts to each player's send queue.
- A dedicated log-shipper thread drains the in-process log buffer and forwards records to `tetrislogd` over IPC.

{:.note-title}

> Tradeoffs
>
> Thread count grows linearly with clients.

#### Event loop with `epoll` / `kqueue`

```
   main_thread (epoll loop)  <----  all client fds
                                    listener fd
                                    ctl socket fd
                                    timer fd (for room ticks)
                                    logshipper fd (out to tetrislogd)

   signal_thread (one)
```

- Single thread services all sockets via non-blocking I/O.
- Room ticks come from `timerfd_create` (Linux) or `EVFILT_TIMER` (BSD/macOS).
- Synchronisation problem mostly disappears because there is one mutator.

{:.note-title}

> Tradeoffs
>
> Scalable, but requires heavy synchronisation.

#### Master plus per-room worker processes

```
   master_process  ----fork()---->  room_worker (one per room)
        |                                  ^
        | shm_open + mmap                  | POSIX mq
        v                                  v
   shared lobby table                shared garbage queue (battle royale)
```

- Master accepts connections, spawns a worker process per room.
- File descriptors for clients are passed to workers via `SCM_RIGHTS` over a Unix socket.
- Global state (room directory, scoreboard) lives in shared memory.
- Cross-room events (battle royale garbage) flow through POSIX message queues.

{:.note-title}

> Tradeoffs
>
> This utilise IPC and is the most elegant solution but hard to debug.

### `tetrislogd`

This is your logger daemon.

{:.important-title}

> **Required**
>
> `tetrislogd` is a separate <span class="orange-bold">process</span>, not a thread inside `tetrisd`. It receives log records over an IPC channel and writes them to disk.

Required behaviour:

- Accept log records from `tetrisd` over IPC
- Write records to the log file specified in `.tetrishrc`
- Maintain a "dropped records" counter that increments when the IPC channel cannot keep up, and emit a summary line periodically (e.g. `dropped 47 records in last 30s`)
- Handle `SIGTERM` (flush buffered records, close file, exit) and `SIGHUP` (reopen log file, for log rotation)
- Survive a `tetrisd` restart without dying (i.e. accept reconnections, do not exit when its IPC peer disappears)

The reason why this should be a separate process is because:

1. **Separation of failure domains:** A bug in the logger (file system full, slow disk, etc.) does not bring down the game daemon. The game keeps playing even if logs are momentarily lost.
2. **Real IPC requirement:** This forces a real producer-consumer IPC channel between `tetrisd` and `tetrislogd`, exercising the OS chapter on IPC in a non-trivial way. Good to challenge yourself.
3. **Production realism:** This is how `systemd-journald` and `rsyslogd` work. Daemons forward log records via a socket; the logger is its own process.

{:.highlight-title}

> **Open design**
>
> The IPC mechanism between `tetrisd` and `tetrislogd` is yours. Reasonable options:
>
> - **Unix domain socket** (datagram or stream, most flexible): reconnect logic must be written on `tetrisd` side if `tetrislogd` restarts.
> - **POSIX message queue** (`mq_open`): built-in bounded buffer, drop policy is explicit.
> - **Named pipe (FIFO)**: simplest in code but need careful lifecycle handling because there's no reconnection
> - **Shared memory ring buffer plus a notification channel**: highest performance but hardest to debug and get right.
>
> Wire format on the channel (line-based, length-prefixed binary, JSON, custom) is yours and you should document it properly.

{:.highlight-title}

> **Open design (lifecycle)**
>
> Whether `tetrislogd` is launched by `tetrish` independently (user types `tetrislogd &` then `tetrisd &`), or `tetrisd` spawns `tetrislogd` automatically, or some other arrangement. Document the startup sequence in your `README`.

### `tetrisctl`

This is the admin CLI of your system.

{:.important-title}

> **Required**
>
> `tetrisctl` is a separate binary that runs administrative actions against a running `tetrisd`. At minimum it must support a status query and a graceful shutdown trigger. The channel it uses must be a real IPC mechanism, not a network connection to the public TCP port.
>
> The control plane must remain available even when the public TCP listener is saturated.

{:.highlight-title}

> **Open design**
>
> The additional commands you offer here, the IPC mechanism (Unix domain socket, named pipe, POSIX message queue, signals plus state files, or a combination), and the wire format on that channel follows `HTTTP`, and it is fixed. See [this](#hypertext-tetris-transfer-protocol) section for details. Document whatever additional things you add for `tetrisctl` in your README.

Useful additional commands you may consider: `rooms`, `players`, `kick <player>`, `reload` (often implemented as sending `SIGHUP` to `tetrisd`), `log-level <level>` (forwards to `tetrislogd`), `dropped-logs` (queries the dropped-records counter from `tetrislogd`). None are mandatory.

### `tetrisu`

This binary is the game client.

{:.important-title}

> **Required**
>
> Terminal-based. Must:
>
> - Connect via TCP, complete the secure session handshake (via `libtetrissh`)
> - Send HTTTP requests for game actions (via `libhtttp`)
> - Receive and render server-pushed `STATE` frames
> - Handle keyboard input non-blocking (so it can read input and network simultaneously)
> - Exit cleanly on `q` or `SIGINT`

{:.highlight-title}

> **Open design**
>
> Rendering technique: raw ANSI escape codes, `ncurses`, or anything else that renders in a terminal. You are free to determine the key bindings beyond the basics or whether the client supports reconnection.

## Libraries

The libraries are the durable, testable, reusable parts of the codebase. They are **statically** linked into the binaries that need them. Each library has one responsibility and you must stick to clean coding practice in this project.

### `libtetrissh`

This library implements the secure session handshake between client and server.

{:.note}
`libtetrissh` is one of the libraries that lives in the shared `corestack` repository, since the 50.003 side may also need a secure session. The 50.005-side details below are unchanged. See the 50.003 handout for what the 50.003 application does with it.

{:.important-title}

> **Required**
>
> Every byte of HTTTP traffic flows inside an authenticated, confidential session established at connection time. The session protocol follows the PA2 pattern:
>
> 1. Client connects, sends a fresh nonce.
> 2. Server sends its X.509 certificate.
> 3. Client verifies the certificate against the bundled CA (`cacsertificate.crt`).
> 4. Server signs the client nonce with its private key (RSA-PSS).
> 5. Client verifies the signature using the public key from the certificate.
> 6. Client generates a 32-byte AES-256 session key, RSA-OAEP encrypts it with the server's public key, sends it.
> 7. From this point on, every frame is `[4-byte big-endian length][AES ciphertext]` carrying one HTTTP message.
>
> Frame size limit: 64 KiB. Larger HTTTP messages must be split by the application or rejected with `413 Payload Too Large`.
>
> Cryptographic primitives come from PA2's `common.c`. You are <span class="orange-bold">not</span> allowed to modify `common.c` or use any cryptographic library other than OpenSSL.

`libtetrissh` is linked into `tetrisd` (server-side handshake) and `tetrisu` (client-side handshake). Linking the same library into both ends <span class="orange-bold">prevents protocol drift</span> between client and server.

{:.highlight-title}

> **Open design**
>
> The shape of the API. An easy API would be something like: `session_handshake_server()`, `session_handshake_client()`, `session_send()`, `session_recv()`, `session_close()`. The exact signatures, the data structure that holds session state, error reporting conventions are yours to determine.

### `libhtttp`

This is the HTTTP protocol library.

{:.important-title}

> **Required**
>
> `libhtttp` parses incoming HTTTP messages and serialises outgoing ones. Both `tetrisd` and `tetrisu` should link against it.
>
> The wire format is fixed and details are described the section [below](#hypertext-tetris-transfer-protocol). `libhtttp` must conform to it exactly.

`libhtttp` is made as a separate library:

1. **Testability**: The protocol parser can be unit-tested in isolation, without needing the daemon running or the client connected.
2. **Reuse**: The same parser code is used on both sides of the connection and there won't be any drift (disagreement, outdated interface)
3. **Separation of concerns**: The dispatcher in `tetrisd` reasons in terms of parsed **messages**, not **bytes**. The renderer in `tetrisu` reasons in terms of `STATE` structures, not strings. They belong in two different level of abstraction.

{:.highlight-title}

> **Open design**
>
> You can freely design: whether parsing is one-shot ("_here is a buffer, give me a parsed message_") or streaming ("_feed me bytes, tell me when a message is complete_"), and whether the parsed-message data structure is heap-allocated, arena-allocated, or stack-friendly. The error codes and how they map to HTTTP status codes returned to the client is also yours to determine.

### `libtetrisbrain`

This library implements the main Tetris game logic,

{:.important-title}

> **Required**
>
> `libtetrisbrain` implements the Tetris game rules. Pieces, rotation, gravity, line clear, scoring, lock delay, soft and hard drop, game over detection. It is **pure logic with no I/O**, no networking, no file system access and no side-effects.
>
> `tetrisd` links against it for authoritative server-side game state. `tetrisu` may link against it for client-side prediction (optional) or just render server state directly.

It is common to have game logic separated out from the entire system supporting it as:

1. **The game logic is non-trivial**: we need to separate it from the rest of the system for ease of development and testing, and also grading.
2. **Pure functions are testable**: a board state plus a move produces a new board state, so you should have no mocking, no fixtures, and simply clean inputs and outputs.
3. **Clean integration boundary**: `tetrisd` knows what `libtetrisbrain` exposes and nothing about how it works internally. The brain can be rewritten without touching the daemon.

{:.highlight-title}

> **Open design**
>
> You can design the data structures for the board and pieces, the rotation system (SRS, ARS, classic Atari, your own invention), the scoring rules, the gravity curve, the lock delay rules. Please Document your choices.

## Hypertext Tetris Transfer Protocol

This is an application-layer protocol.

{:.important-title}

> **Required**
>
> The following wire format is fixed.

### Message grammar

```
REQUEST       ::= REQUEST-LINE *(HEADER CRLF) CRLF [BODY]
REQUEST-LINE  ::= METHOD SP PATH SP "HTTTP/1.0" CRLF

RESPONSE      ::= STATUS-LINE *(HEADER CRLF) CRLF [BODY]
STATUS-LINE   ::= "HTTTP/1.0" SP STATUS-CODE SP REASON-PHRASE CRLF
```

<span class="orange-bold">We shall use `HTTTP/1.0` literal.</span>

### Required methods

| Method   | Path                      | Purpose                                                 |
| -------- | ------------------------- | ------------------------------------------------------- |
| `JOIN`   | `/room/<id>`              | Join or create a room                                   |
| `LEAVE`  | `/room/<id>`              | Leave a room                                            |
| `START`  | `/room/<id>`              | Begin the game (room owner only)                        |
| `MOVE`   | `/room/<id>/player/<pid>` | Body: `LEFT` or `RIGHT`                                 |
| `ROTATE` | `/room/<id>/player/<pid>` | Body: `CW` or `CCW`                                     |
| `DROP`   | `/room/<id>/player/<pid>` | Body: `SOFT` or `HARD`                                  |
| `STATE`  | `/room/<id>`              | **Server-originated.** Pushed broadcast of board state. |

`STATE` is the only server-originated message. Clients must read these unprompted while interleaving with their own request-response cycles.

{:.highlight-title}

> **Open design**
>
> You may add methods (`CHAT`, `PAUSE`, `WHISPER`, `SPECTATE`, etc.). Document them. The live extension pool may ask you to add a method on the day; designing your `libhtttp` dispatcher to make that easy is a strategic choice.

### Required status codes

Use HTTP-like status codes with their conventional meanings. The following must be reachable:

`200`, `201`, `400`, `401`, `403`, `404`, `409`, `429`, `500`.

You may add others if you wish.

### Required headers

- `Content-Length` on every message with a body
- `Content-Type: application/tetris-command` on client requests with a body
- `Content-Type: application/tetris-state` on server `STATE` broadcasts
- `Player-Id` on every authenticated request
- `Date` on every response (RFC 1123 format)

### Sample

HTTTP JOIN Request:

```
JOIN /arena/main HTTTP/1.0
Host: tetrish.local
User: alice
Mode: battle-royale
Client-Version: 1.0
Content-Length: 35

{"skin":"cyan","start_level":0}
```

HTTTP JOIN Response:

```
HTTTP/1.0 200 OK
Session-Id: s-8f31a2
Player-Id: p17
Tick-Rate: 20
Content-Type: application/json
Content-Length: 94

{"arena":"main","player_id":"p17","board_width":10,"board_height":20,"next_tick":48122}
```

HTTTP rejected move sample:

```
HTTTP/1.0 409 INVALID_MOVE
Seq: 43
Server-Tick: 48124
Content-Type: application/json
Content-Length: 82

{"accepted":false,"reason":"collision","authoritative_x":3,"authoritative_y":14}
```

HTTTP Battle Royale status poll:

```
GET /arena/main/state HTTTP/1.0
Session-Id: s-8f31a2
Accept: application/json
```

## The control plane

{:.important-title}

> **Required**
>
> `tetrisctl` and `tetrisd` communicate via a real IPC mechanism, not the public TCP socket and not function calls. The mechanism must be <span class="orange-bold">local-only</span>.
>
> The control plane must remain available even when the public TCP listener is congested. Imagine the server is being overwhelmed by client requests and you need to issue `tetrisctl shutdown`. <span class="orange-bold">It must still work.</span>

{:.highlight-title}

> **Open design**
>
> Suggested mechanism: Unix domain socket, named pipe (FIFO), POSIX message queue, or signals plus a state file. Wire format on that channel: line-based, length-prefixed, JSON, or whatever else you want, just don't forget to document it.

If you use Unix domain sockets, the path lives in `.tetrishrc`. If you use signals, **document** which signals trigger which actions and how state is returned to the user.

## Logging

{:.important-title}

> **Required**
>
> All logging is performed by `tetrislogd`, the dedicated logger daemon explained above. `tetrisd` and other binaries forward log records to `tetrislogd` over **IPC**.
>
> Game-critical threads (the listener, client handlers, room tickers) **must not block** on the logger. If the IPC channel is <span class="orange-bold">full</span>, log records may be dropped, but the dropped count must be observable. `tetrislogd` exposes the dropped-records count, and `tetrisd` may also track its own local-side drops if it buffers internally.
>
> Every connection event, secure session establishment, HTTTP request and response, room state change, and admin action <span class="orange-bold">must</span> be logged with a timestamp.

{:.highlight-title}

> **Open design**
>
> The IPC mechanism between `tetrisd` and `tetrislogd` is free for you to design. So is the internal buffering inside `tetrisd` (a ring buffer drained by a dedicated shipper thread is one common pattern), the exact log line format and level tagging system and whether `tetrisctl` or `tetrisu` also log to `tetrislogd` (recommended, but your call).

A typical pipeline looks like this:

```
[client_thread]                                      [tetrislogd]
   log("connection from %s", ...)                          ^
       |                                                   |
       v                                                   |
   ring_buffer_push() (non-blocking, drops on full)        |
       |                                                   |
       v                                                   |
   [logshipper_thread] drains buffer                       |
       |                                                   |
       v                                                   |
   IPC send  --------------------------------------------->|
                                                           |
                                                  writes to disk
```

{:.note}

> Why so complex?
>
> This pattern matches production loggers (`systemd-journald` drops with `Suppressed N messages` lines when overwhelmed). Your system does not need to be identical, but the principles (non-blocking on the producer, observable drops, separate process for the writer) are <span class="orange-bold">required</span>.

## Concurrency

{:.important-title}

> **Required**
>
> - Your daemon must handle multiple concurrent clients correctly. You should not suffer from data races, torn reads, double frees.
> - You must document your locking strategy in the README: which mutex protects which structure, and the global lock acquisition order.
> - Code must conform to the documented order. We will read the code with the README open.
> - Never hold a mutex across a blocking syscall.
> - If you use `fork()` from a multi-threaded process, the child must call `execve()` immediately or use only async-signal-safe functions.

{:.highlight-title}

> **Open design**
>
> You can choose to implement it using threads vs processes vs hybrid. The Lock granularity is also up to you (the tradeoff is performance): one big lock, per-room locks, per-structure locks, lock-free queues. Synchronisation primitives used are also yours to pick: `pthread_mutex`, semaphores, condition variables, or anything else POSIX provides.

## Battle Royale Mode

Battle Royale mode is a <span class="orange-bold">required</span> part of tetriSH.

However, you should implement it only _after_ the baseline Tetris system is working: independent terminal client, server-authoritative game state, secure session, HTTTP messages, logging, and control plane.

When a player clears N lines in a single move with N >= 2, those rows minus 1 are converted to **garbage** and inserted at the bottom of a random other player's board in a different room.

{:.important-title}

> **Required**
>
> Battle Royale mode is required for tetriSH completion.
>
> Garbage transfer between rooms must be server-side managed and must use a **real IPC mechanism**. Direct function call into another room's state while holding its mutex from the source room's thread does **not** count, even if it works.

{:.highlight-title}

> **Open design**
>
> Suggested mechanism: POSIX shared memory plus semaphore, POSIX message queue, named pipe, Unix domain socket carrying serialised events, or a combination.

### Worked Example with `shm`

This is a worked example with shared memory plus message queue:

```
shm: garbage_pool[N]   <- ring of pending garbage events
mq:  /tetris-garbage   <- notification on new event
```

{:.highlight}
A room that clears lines writes the event to the shared ring, signals via the message queue. Other rooms read events targeted at them and inject garbage into their own boards under their own room mutex.

## The `.tetrishrc` file

{:.important-title}

> **Required directives** (the daemons cannot start without these):
>
> - `listen_port`: TCP port for clients
> - `cert_path`, `key_path`, `ca_path`: certificate paths
> - `log_path`: path where `tetrislogd` writes log records
> - `log_ipc`: address of the IPC channel between `tetrisd` and `tetrislogd` (the format depends on your chosen mechanism: a Unix socket path, an mq name, etc.)
>
> The shell side requires no specific directives but must execute every line as it does in PA1.

{:.highlight-title}

> **Open design**
>
> Tunable directives such as `max_rooms`, `max_players_per_room`, `tick_hz`, `log_level`, the control plane address, the prompt string, custom aliases. Add what you need. Sensible defaults if a directive is absent.

## File system layout

This is a suggested design. You are free to reorganise.

{:.highlight-title}

> **Open design**
>
> All paths must be configurable via `.tetrishrc`. Every path should be relative to the project root. No hard-coded path should be there.

```
project/
    bin/
        tetrish
        tetrisd
        tetrislogd
        tetrisctl
        tetrisu
    lib/
        libtetrissh.a
        libhtttp.a
        libtetrisbrain.a
    include/
        tetrissh.h
        htttp.h
        tetrisbrain.h
    auth/
        server.crt
        server.key
        cacsertificate.crt
        generate_keys.sh
    sample.tetrishrc
    var/
        log/                 (created at runtime)
        run/                 (sock, pid)
    src/
        tetrish/
        tetrisd/
        tetrislogd/
        tetrisctl/
        tetrisu/
        libtetrissh/
        libhtttp/
        libtetrisbrain/
    tests/
    Makefile
    README.md
```

## Out of scope

There's absolutely no need to implement all these. You will not gain special considerations.

- Reimplementing TCP, UDP, or any reliability protocol on top of UDP. Use TCP from the kernel.
- Implementing your own crypto primitives. Use OpenSSL via PA2's `common.c` only.
- Web frontend, GUI client, mobile client. Terminal only.

## Grading

This section explains how tetriSH is graded, how groups indicate that they are taking this track, what the live checkoff looks like and how the prize is awarded.

Students who complete tetriSH are assessed against the same broad learning objectives as PA1 and PA2, but through one larger integrated system.

{:.note}
The prize is separate from course marks.

### Features to Demo

#### Baseline Tetris requirement

{:.important}

> Baseline is a requirement to avoid zero
>
> To pass the tetriSH track, the system **must** include a working baseline Tetris game. This does not need to be a polished commercial-quality Tetris implementation, but it must be playable enough to prove that the client, server, protocol, secure session, game logic, concurrency, and logging are integrated correctly. <span class="orange-bold">Failure to fulfil the baseline demo results in 0 marks for your both PA</span>, so please consider carefully.

The baseline game must demonstrate:

1. a player can connect using `tetrisu`,
2. the client completes the secure session handshake before any HTTTP traffic,
3. the player can `JOIN` a room,
4. the player can `START` a game,
5. the server creates and maintains the game state,
6. a Tetris piece falls over time,
7. the player can move the piece left and right,
8. the player can rotate the piece,
9. the player can soft drop and hard drop,
10. pieces lock when they reach the bottom or collide with existing blocks,
11. completed lines are cleared,
12. the score or line count updates after line clears,
13. the game detects game over,
14. the server sends `STATE` messages to the client,
15. `tetrisu` renders the board in the terminal,
16. the client can quit cleanly,
17. the server logs game events through `tetrislogd`.

The server is **authoritative**. `tetrisu` may render state and collect keyboard input, but the real game state must live in `tetrisd`, using `libtetrisbrain`.

The client _must not_ simply update its own local board and pretend the server accepted the move. Player commands must go through the full stack:

```text
tetrisu
  -> libhtttp
  -> libtetrissh
  -> TCP socket
  -> tetrisd
  -> libtetrisbrain
  -> STATE response back to tetrisu
```

From the system architexture point of view, here's what you should have:

1. `tetrisd` accepting multiple concurrent clients,
2. `tetrisu` rendering a user friendly playable terminal client,
3. `libhtttp` parsing and serialising HTTTP messages and handle error messages properly,
4. `libtetrisbrain` implementing core Tetris game logic efficiently,
5. `tetrislogd` receiving log records over IPC and writing them to disk in an organised manner (logs are labeled warning, debug, error, info, etc),
6. `tetrisctl` communicating with `tetrisd` over a local IPC control plane,
7. **clean** build instructions from a fresh checkout,
8. a README explaining architecture, concurrency, IPC choices, security assumptions, and known limitations.

#### Battle Royale Requirement

Battle Royale mode is not part of the baseline MVP demo, but it is still required for full tetriSH completion.

The baseline MVP exists to define the minimum integrated system that must work <span class="orange-bold">before</span> Battle Royale can be meaningfully assessed. A group that cannot demonstrate the baseline MVP <span class="orange-bold">cannot</span> pass tetriSH, even if some Battle Royale code exists.

#### Group formation and Week 3 indication

tetriSH is completed in **groups of 3**, the same group for your labs. All three members must be enrolled in both 50.003 and 50.005. By Monday, 6PM of **Week 3**, groups intending to take the CoreStack track must notify **both** instructors (50.005 and 50.003) of their intent.

You should submit a short document on eDimension detailing:

1. group member names and confirmation that all three are enrolled in both 50.003 and 50.005,
2. declared role for each member,
3. a short architecture sketch,
4. intended thread or process model,
5. intended IPC mechanisms,
6. intended division of work,
7. repository link, if available.

{:.important}
If there are too many participants, for logistics reasons, your instructor has the right to filter the selection based on your proposal.

After Week 3 Monday 6PM, there's no more chance to switch to this track.

### Roles

Each group declares one member for each role. This **should be decided up front**. Here's a sample:

| Role                            | Owns                                                                                                                   |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| **Systems**                     | `tetrish`, `tetrisd` process model, concurrency, `tetrislogd`, `tetrisctl`, signal handling, and IPC channels          |
| **Networking and Security**     | `libtetrissh`, `libhtttp`, secure session correctness, HTTTP parser and serialiser, threat model, and request dispatch |
| **Application and Integration** | `tetrisu`, `libtetrisbrain`, room lifecycle, game loop, build system, and integration tests                            |

Members may help one another. That is expected. However, each member <span class="orange-bold">must be able to answer live Q&A questions</span> about their declared role.

The role declaration is not meant to stop collaboration but it is meant to make individual responsibility clear.

### How grading works

The tetriSH project is graded as a **group programming assignment**. Group work management is expected to be **professional**, and hence the role declaration above.

{:.note}
All three members receive the **same** project mark. The grades are separated into 3 parts: functionality (5%), live extension (5%), live QnA (10%).

The grading focuses on whether the submitted system demonstrates the learning objectives of PA1 and PA2 through the tetriSH system. Battle Royale mode is part of the <span class="orange-bold">required</span> tetriSH system. However, the baseline MVP is _assessed first_ because it proves that the core shell, daemon, networking, security, protocol, logging, control plane, and server-authoritative game loop are working.

A group that cannot demonstrate the baseline MVP cannot rely on Battle Royale features to compensate for missing core systems behaviour. A group that completes only baseline MVP without Battle Royale feature will gain some marks but <span class="orange-bold">NOT</span> full marks.

| Area                             | What is assessed                                                                    |
| -------------------------------- | ----------------------------------------------------------------------------------- |
| **Shell and process management** | `tetrish`, process spawning, background jobs, daemon lifecycle                      |
| **IPC and control plane**        | `tetrislogd`, `tetrisctl`, local IPC, log forwarding, shutdown and reload behaviour |
| **Networking**                   | TCP server, concurrent clients, request handling, server-pushed state               |
| **Security**                     | certificate verification, nonce proof, RSA-wrapped AES session, encrypted frames    |
| **Protocol design**              | HTTTP parser, serialiser, headers, status codes, malformed input handling           |
| **Concurrency**                  | thread or process model, lock discipline, avoiding races and deadlocks              |
| **Application logic**            | terminal client, game loop, Tetris rules, room state consistency                    |
| **Code quality**                 | modularity, memory management, error handling, build cleanliness                    |
| **Documentation**                | README clarity, architecture explanation, known limitations                         |

{:.important}
The project <span class="orange-bold">does not need</span> to be the most feature-rich submission to receive full project marks. It must be correct, demonstrable, understandable, and aligned with the required systems objectives.

## Commitment Policy

CoreStack is an optional alternative track. Groups may withdraw and return to the normal PA1 + PA2 path on the 50.005 side, but only before the fallback deadline, and withdrawal must happen on both courses at the same time.

{:.important}
The fallback deadline is **Week 5**, Friday, 6PM.

Before this deadline, a group <span class="orange-bold">must</span> notify both the 50.005 and 50.003 instructors that it is _withdrawing_. The group will then complete PA1 and PA2 in the normal way and will be graded using the normal PA1 and PA2 rubrics on the 50.005 side.

There is **no penalty** for falling back before the deadline.

After the fallback deadline, the group is committed to the CoreStack track. The group must submit and demonstrate tetriSH at the final 50.005 checkoff.

{:.important}
Failing to deliver the MVP during the final checkoff warrants 0 marks for both PA1 and PA2 components on the 50.005 side.

## Live checkoff

{:.highlight}
Each tetriSH group must attend a live checkoff. Only tetriSH groups that can complete at least the MVP can schedule the checkoff. Otherwise, they would obtain zero regardless of the state of the code.

The live checkoff is used to **verify** that the submitted system works and that the group understands the system.

A typical checkoff is **60 to 90 minutes**, depending on the number of groups.

The grades are separated into two segments: **functionality and live extension + QnA.**

| Phase                   | Activity                                                                                                                |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **Baseline demo**       | The group demonstrates the required system running from a clean build.                                                  |
| **Systems inspection**  | Instructor inspect selected parts of the implementation, logs, IPC behaviour, and failure handling. Also play the game. |
| **Live Q&A**            | Each member answers questions about their declared role.                                                                |
| **Prize consideration** | For prize-eligible groups, we consider engineering quality, code quality, product quality and possible extensions.      |

### Functionality (5%)

The group should be prepared to run:

1. `tetrish`,
2. `tetrisd`,
3. `tetrislogd`,
4. `tetrisctl`,
5. at least two `tetrisu` clients,
6. packet inspection or logging evidence showing that post-handshake traffic is encrypted using wireshark.

{:.highlight}
The demo must be reproducible from the submitted repository.

Two sections below. Take what you want.

### Live Extension (5%)

The live extension tests whether you can navigate and modify your own codebase under time pressure. The purpose of this activity is to ensure that not only you "understand" what your code is doing but also own its architecture. It demonstrates long-term planning as well since clean architecture would make making these small changes easy.

Your group have approximately **20 minutes** to implement a small extension on your running system, plus 5 minutes to walk the diff.

{:.notes-title}

> AI Tool Policy
>
> No agentic or autonomous coding tool. Chat-based assistants are allowed, but no copy-pasting is allowed. Every line you accept from the AI must be one you can explain. Instructor can interrupt and ask why this line, and you should be able to explain fully.

#### Task pool

Here are the sample task pool. We may select from these OR ask you to implement adjacents.

**Networking and Security (`libhtttp`, `libtetrissh`)**

1. Add a new HTTTP method (e.g. `PAUSE` on `/room/<id>`), including parser handling, dispatch in `tetrisd`, a reachable `409` if the game is not active, and a log entry on every call.
2. Add a new required header (e.g. `X-Client-Build`) and reject any request missing it with the right status code.
3. Add a `Retry-After` header on every `429` response and have the client honour it.
4. Swap RSA-OAEP key wrapping for RSA-PKCS1v15 (or vice versa). Explain the security difference.
5. Change the symmetric key derivation: instead of the client generating a raw 32-byte AES key, derive it via HKDF-SHA256 over a fresh client nonce + server nonce, using primitives already exposed by `common.c`.
6. Add HMAC over the ciphertext frame (if your current frame has no auth tag) and have the receiver verify before decrypting.
7. Add per-session replay protection: a monotonic counter header on every request, server rejects out-of-order or duplicate counters with `400`.
8. Reduce the AES key size from 256 to 128 bits and explain whether this materially changes your threat model.

**Systems (`tetrish`, `tetrisd`, `tetrislogd`, `tetrisctl`)**

1. Add a new `tetrisctl` subcommand that returns a structured snapshot of one room (player count, current piece, score, tick rate).
2. Add `SIGUSR2` to `tetrisd` for a one-shot full state dump to log without restart.
3. Implement size-based log rotation in `tetrislogd` without dropping records during the rotation.
4. Add a per-IP connection limit configurable in `.tetrishrc`, enforced at `accept()`, rejection logged at warning level.
5. Implement `tetrisctl drain`: stop accepting new connections, let in-flight games finish, then exit cleanly on the next `tetrisctl shutdown`.
6. Add a `kick <player>` admin command that closes that player's session and broadcasts the room update.
7. Swap your logger IPC mechanism from one option to another (e.g. Unix socket to POSIX message queue, or named pipe to socket). The drop counter must still work.
8. Add a `reload` path that re-reads `.tetrishrc` on `SIGHUP` and applies a subset of directives at runtime (e.g. log level, tick rate) without dropping connections.

**Application and Integration (`tetrisu`, `libtetrisbrain`)**

1. Add a hold-piece feature (one swap per piece, key-bound on the client, preview rendered).
2. Switch rotation system (SRS to classic, or whatever you implemented to a different one). Demonstrate a kick that worked before and not after, or vice versa.
3. Change the scoring rule for tetrises (four-line clear) and demonstrate it changing across two rounds.
4. Add a ghost piece projection at the landing position. Defend whether the server needs to send extra state for this.
5. Add a hard-drop preview line in the client.
6. Change Battle Royale targeting from "random other room" to "the room with the highest current score." This is a real integration task: brain, broadcast, cross-room IPC.
7. Add a configurable starting level (faster initial gravity) read from `.tetrishrc`.
8. Add a "spectate" mode in the client that subscribes to a room without joining as a player.

Here's the marking scheme:
| Score | Outcome |
| --- | --- |
| 5 | Compiled, ran, behaved correctly, defended in walkthrough. |
| 4 | Compiled and ran, minor messiness or one bug honestly acknowledged, defended. |
| 3 | Compiled and ran but partially incorrect or incomplete. Direction was right. |
| 2 | Did not compile at the bell, or compiled but did not behave. Some progress visible. |
| 1 | Floundered. Could not orient in own codebase. |
| 0 | Did not engage, used a forbidden tool, or accepted code that could not be explained. |

### Live Q&A (10%)

{:.note}
This part of the checkoff is an extensive ownership test.

We want to **validates** that the submitted system is actually yours and that you understand it. Each member will be questioned on their declared role plus the integration boundary with the other roles. Other members may attend but **may not answer, prompt, or signal** during your turn.

You are allowed to refer to your own submitted code and lecture notes. You are also allowed to discuss among your team members. No AI tools or online searches are allowed.

{:.important-title}

> Procedure
>
> You will be having a **conversation** with your instructor for 30 minutes. Instructor is free to ask at most 6 questions in the style of the pool below. Follow-up questions and questions about the code e.g: "what does this line do? which functions call this?" don't count either.

#### Question pool

Questions can be about **anything** in your project. Some examples by area are written below.

**Concurrency and lifecycle**

1. Walk me through what happens, line by line, when a client sends `MOVE LEFT` while the room ticker is mid-gravity. Which lock do you hold? In what order? Why?
2. I `kill -SIGTERM` your `tetrisd` while a game is in progress. Walk me through the shutdown path. Where does each thread or process learn it should exit?
3. Show me where you handle a slow client. What happens to the room if one client stops reading?
4. What is your global lock acquisition order? Show me one site that respects it and one that could deadlock if you got the order wrong.
5. Your `tetrislogd` IPC channel fills up. Show me the line that decides to drop. What is the drop counter, and how does it become visible?

**Networking and security**

1. Walk me through the handshake from `accept()` to the first encrypted HTTTP frame.
2. A man-in-the-middle replays a `MOVE` frame from earlier in the session. What stops it? Show me the code.
3. The client sends an HTTTP request with `Content-Length` larger than the actual body. Where do you detect this? What status code do you return?
4. What attack does RSA-PSS prevent that RSA-PKCS1v15 signing does not?
5. Show me where you free the `X509*` and `EVP_PKEY*` from the handshake. What happens if the handshake fails halfway?

**Protocol and parsing**

1. A client sends a request with `\n` line endings instead of `\r\n`. What happens?
2. A header has a value with a colon in it. Show me how your parser handles this.
3. Walk me through the path of a `STATE` broadcast from the room ticker to the client's terminal.
4. Where is `Date` set on a response? Why there?
5. Two clients hit the same room with `START` simultaneously. What happens? Show me the code path.

**IPC and control plane**

1. I run `tetrisctl shutdown` while the public TCP listener is being flooded. Why does the control command still work?
2. Your `tetrislogd` dies. What does `tetrisd` do? Show me the reconnection logic.
3. Walk me through the wire format on your control plane channel. Show me the parser.

**Application and game logic**

1. Show me the line clear function. Walk me through it on a board where rows 18 and 20 are full.
2. Where does lock delay live? Walk me through a piece that is being held in place by repeated rotations.
3. A garbage row arrives from another room while the local ticker is mid-tick. What synchronises this?
4. Your client is rendering at 30 Hz but the server ticks at 60 Hz. What happens visually? Where in the code?

{:.note-title}

> Code query
>
> Instructor is free to ask about ANY line of submitted code, any directive in `.tetrishrc`, any entry in your README, any commit in your git history, any test in your test suite.

#### Multiplier scheme

Each group starts with a multiplier of `1.0` on their Q&A mark. We would then ask a question with a certain topic. Here's what you'll gain:

- **Strong answer**: specific, code-grounded, identifies the structure and the failure mode. No multiplier change.
- **Weak answer**: vague, generic, or detached from your actual code (the handout's "we used a mutex to prevent race conditions" example). Multiplier applied: `× 0.85`.
- **Cannot answer**: silence, a wrong answer you concede, or "I don't know." Multiplier applied: `× 0.7`.

These multipliers would <span class="orange-bold">compound</span>, not add.

| Misses                    | Resulting multiplier (rough) |
| ------------------------- | ---------------------------- |
| 0                         | 1.00                         |
| 2 weak                    | 0.72                         |
| 1 weak + 1 cannot         | 0.60                         |
| 3 cannot                  | 0.34                         |
| 2 cannot in declared role | 0.25                         |
| 4+ cannot, mixed          | below 0.20                   |

Final Q&A mark is computed as `10% × multiplier`.

#### Honesty discount

A simple "I don't know, but here is how I'd find out in my code" is worth more than a confident wrong answer. Our instructor will make judgments and impose penalty accordingly.

Bullshitting compounds harder than admitting a gap. If you bluff and the instructor catches it on a follow-up, that question is treated as a "cannot answer".

## AI-generated code

Using AI tools is allowed. However, the submitted code must be understood and you must own the architecture. A group may use AI tools heavily and still do well, provided they read, test, debug, modify, and understand the generated code.

A group that submits working AI-generated code but cannot explain it may lose <span class="orange-bold">immediate 10% penalty</span> and will not be eligible for the prize.

## Prize eligibility

The prize is <span class="orange-bold">separate</span> from course marks. There will be a live demo session for shortlisted groups in Week 13, stay tuned.

To be eligible for the prize, a group must:

1. complete the baseline MVP,
2. complete Battle Royale mode,
3. pass the live demo,
4. pass the live Q&A validation,
5. demonstrate that all three members understand their declared roles,
6. build successfully from a clean checkout,
7. have no academic integrity issues
8. ship a clean release bundle from a tagged commit (build instructions, sample `.tetrishrc`, certificates, install steps) that <span class="orange-bold">any other group in the cohort</span> can clone and run end to end without your assistance,
9. survive a cohort-scale playtest. We might ask other students to point their `tetrisu` at a hosted instance of your `tetrisd` and play. The system must hold up under real concurrent load from the rest of the batch, not just from your own team's demo machines.
10. must project the server's cli in front of the cohort, and live log the sessions / peek into any rooms, or manage live users when asked.

{:.note-title}

> SUTD Wifi
>
> It is your responsibility to check how clients can connect to the server's private IP via SUTDWifi. Otherwise, you may also host it somewhere and ssh into it. See [this]({{ site.baseurl }}/pa2/intro-c) for more info.

A group may receive project credit but still be ineligible for the prize if:

1. the implementation is too fragile to evaluate beyond the baseline,
2. the live Q&A shows weak understanding,
3. there are serious code ownership concerns,
4. Battle Royale is present but clearly hacked together without proper systems design.
5. the system (client) cannot be run cleanly by anyone other than the original authors.
6. the cohort-scale playtest exposes deadlocks, leaks, or crashes that would not show up on a two-client local demo.

Completing Battle Royale does not guarantee winning the prize. It only makes the group eligible for prize consideration.

### Version control discipline

{:.note-title}

> **Required for prize eligibility (not for passing the project)**
>
> Your git history must read like the record of a real team of three building a real system over 13 weeks. Specifically:
>
> - Meaningful commit messages that describe intent, not "update" or "fix stuff" or pasted AI output.
> - Sensible branching. Feature branches merged via PR (or equivalent) with the other two members visible as reviewers in the history.
> - At least one tagged release for the bundle you submit for the cohort-scale playtest (e.g. `v1.0-mvp`, `v1.0-br`).
> - No squashed "final commit" that dumps the whole project in one go.
> - Co-authorship visible. If 2/3 members pair-programmed a module, the commit reflects that.
> - The three declared roles should be traceable in the history. We should be able to see which member touched which subsystem.
>
> We will read your git log during prize consideration. A clean codebase with a fabricated git history is not a clean codebase.

### Cohort-scale playtest

Each prize-eligible group hosts their own `tetrisd` and `tetrislogd` during a scheduled window in Week 13. The cohort is invited to point `tetrisu` (built from the group's own repository) at the hosted daemon and play.

The group decides where to host: campus WiFi, home network with port forwarding, a VPS, a tunnel, anything that lets the cohort actually reach the daemon. Hosting choice is part of the engineering decision and will be discussed at Q&A.

**Expected concurrent load**: approximately [N] connections. Your system must remain responsive under this load. The system must survive ill-behaved clients (malformed HTTTP, oversized frames, slow readers, half-open handshakes) without the daemon crashing or hanging.

### Selection of Winning group

Among prize-eligible groups, the winning group is selected based on **overall** engineering quality. There's no limit to what you can do, as long as you do not violate the [constraints](#project-constraints) above.

The following criteria are considered:

| Criterion                            | What we look for                                                                                                                               |
| ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Correctness**                      | The required system works reliably during the demo.                                                                                            |
| **Architecture**                     | The process model, library boundaries, IPC choices, and control plane are clean and defensible.                                                |
| **Code quality**                     | The code is readable, modular, maintainable, and handles errors properly.                                                                      |
| **Systems depth**                    | The group demonstrates strong understanding of processes, signals, IPC, concurrency, and shutdown behaviour.                                   |
| **Security depth**                   | The secure session is implemented correctly and the group understands its threat model.                                                        |
| **Protocol robustness**              | HTTTP parsing, status handling, framing, and malformed input behaviour are sensible.                                                           |
| **Application quality**              | The terminal client is usable and the game logic is stable.                                                                                    |
| **Testing and debugging discipline** | The group can show evidence of testing, logging, and debugging.                                                                                |
| **Q&A performance**                  | All three members demonstrate ownership of their declared parts.                                                                               |
| **Extensions beyond baseline**       | Battle Royale, richer admin controls, better logging, observability, tests, or other well-integrated extensions may strengthen the submission. |

The prize is <span class="orange-bold">not</span> awarded purely for the most features. Basically, a smaller but cleaner and better-understood system may beat a larger but fragile system.

If no group meets the prize eligibility standard, the prize may be withheld.

## Summary

This `tetriSH` project exist purely as a challenge to push your potential further. You are to embark on this solely if you are interested.

{:.important}
If you're committed, it would replace PA1 and PA2 on the 50.005 side and carries the same course-mark ceiling as the normal PA1 + PA2 path. There are no bonus marks for this, only a chance at the prize. The 50.003 side runs in parallel under the 50.003 project rubric.

Groups are to indicate interest by the end of Week 3 to both course instructors and the groups should consist of 3 members with declared roles. The live checkoff includes a demo, live patching & coding, and compulsory Q&A.

Note that the prize is separate from course marks.

Depending on the course budget, one or two winning group receives multiple Apple products worth approximately S$3,000.

{:.important-title}

> Important Reminder
>
> **You are NOT allowed to modify `common.h` or `common.c`.** These files contain all the cryptographic wrappers and socket helpers you need. This is the same rule as PA2.
>
> **Memory management in C**: Every `malloc`'d buffer, `X509*`, and `EVP_PKEY*` must be freed by the owner. Run your demo under `valgrind`; visible leaks at checkoff lose marks.
>
> **No HTTPS:** You should implement the secure session yourself in `libtetrissh` over TCP. You are not using TLS, OpenSSL's `SSL_*` API, or any reverse proxy. Cryptographic primitives come from `common.c` only.
