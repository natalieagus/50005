---
layout: default
permalink: /pa1/part7
title: A Senior's Sample
description: Sample PA1 implementation
parent: Programming Assignment 1
grand_parent: Programming Assignment
nav_order:  8
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

# Senior's Sample: C Shell Custom
{: .no_toc}

{:.note}
This is the work of [Ryan Pek](https://github.com/rpeky), a class of 2027 CSD senior. This code is given for you as a reference, for you to learn. Any attempt to plagiarise will be seriously dealt with and may result in suspension.

The full implementation can be viewed from [here](https://github.com/natalieagus/C-Shell-custom).

### Purpose
These notes are for juniors who are about to start PA1. The codebase here is a senior's submission that extended the base PA with raw-mode line editing, an rc file, aliases, history, a daemon family (`dplant`, `dspawn`, `dkill`, `dcheck`), and a few quality of life helpers. It is worth reading because it is **structured cleanly**, the build system is solid, and the rough patches are honest learning material. Read it to understand decisions, not to copy code.

Open the project alongside this page. Each section points you at a specific file or block. Treat the "*Things to read critically*" section as required reading: every senior sample has rough spots and pretending otherwise is bad for your own work.



## Project layout at a glance

```
C-Shell-custom/
  makefile
  source/
    shell.c        builtin.c        cmdline.c
    shell.h        builtin.h        cmdline.h
    system_programs/
      dplant.c  dspawn.c  dkill.c  dcheck.c
      ld.c      ldr.c     find.c   sys.c    backup.c
      system_program.h
  cseshell                 (built binary, normally in .gitignore)
  bin/                     (built util binaries, created by make)
  files/                   (test fixtures)
  debug_scripts/           (valgrind, gdb wrappers)
  daemonslayer.sh, fixbroken.sh, qformat.sh, reset.sh
  README.md
```

Three things to notice before reading any code:

1. **The shell is split into three translation units** (`shell.c`, `builtin.c`, `cmdline.c`), each with its own header. Each file has one clear responsibility: process control, builtin commands, line editing. Compare this with starter code that puts everything in `main.c`. Keeping concerns separated is the single biggest readability win in this sample.
2. **System programs are standalone executables**, not functions inside the shell. The shell finds them via `$PATH` after prepending `./bin`. This is how a real shell works: `ls` is `/bin/ls`, not a function the shell calls directly. The `execvp` family does the handover. You will appreciate why later when you implement piping and signal forwarding.
3. **There is a `bin/` directory that the Makefile creates**, separate from `source/`. Built artefacts never live next to source. This is not just tidiness, it is what lets `make clean` be a single `rm -rf`.


## The Makefile

The Makefile is short (about 60 lines) but it does a lot of work. Ryan spent some time customising it and tailoring it to his needs when completing the assignment.

Each section below is numbered for easy reference.

### 2.1 The compiler and flags lines

```make
CC = gcc
CFLAGS = -Werror -Wall -Wextra -fsanitize=address -g -Og \
         -D_POSIX_C_SOURCE=200809L -D_GNU_SOURCE
```

`CC` is a conventional Make variable that implicit rules also use. By setting it explicitly you make the choice of compiler obvious and overridable from the command line (`make CC=clang` would work).

`CFLAGS` is where the interesting decisions live. Every flag is deliberate. The next section breaks them down individually.

{:.note}
Ryan left a fallback `CFLAGS` line commented out immediately below the main one. That is the "if `-Werror` is too strict to even start, use this to get unblocked" line. Keep that pattern in your own `Makefile` while you are *still writing code*, but uncomment the strict version before you submit.

### 2.2 Path variables

```make
SRC_ROOT   = ./source
UTIL_DIR   = ./source/system_programs
BIN_DIR    = ./bin
MAIN_SRC   = $(SRC_ROOT)/shell.c $(SRC_ROOT)/builtin.c $(SRC_ROOT)/cmdline.c
MAIN_EXEC  = ./cseshell
```

Three things to learn from this block:

- **Hardcoded paths are <span class="orange-bold">isolated</span> at the top.** If the directory layout changes, you edit one line, not 20.
- **`MAIN_SRC` is a list, not a wildcard.** This is a deliberate choice: the shell has a small, known set of TUs, and listing them by hand means `make` will not silently pick up a stray `test.c` you forgot in `source/`. The opposite choice (wildcard) is used for the system programs below; see 2.3 for why that trade-off makes sense in that case.
- The `MAIN_EXEC` is `./cseshell` at the project root, not in `bin/`. That is a UX choice: students expect to type `./cseshell` to run.

### 2.3 The *wildcard* pattern for system programs

```make
UTIL_SRC = $(wildcard $(UTIL_DIR)/*.c)
UTIL_BIN = $(patsubst $(UTIL_DIR)/%.c,$(BIN_DIR)/%,$(UTIL_SRC))
```

This is the most useful `Make` idiom in the file.

- `$(wildcard $(UTIL_DIR)/*.c)` expands to every `.c` file in `source/system_programs/` at the moment `make` is invoked.
- `$(patsubst PATTERN,REPLACEMENT,TEXT)` does pattern substitution. Here it rewrites every `source/system_programs/foo.c` to `./bin/foo`. The `%` is the wildcard inside `patsubst`.

The author left an alternative hand-maintained list commented out:

```make
#UTILS = dcheck backup find ld ldr sys
#UTIL_SRC = $(addprefix $(UTIL_DIR)/, $(addsuffix .c, $(UTILS)))
#UTIL_BIN = $(addprefix $(BIN_DIR)/, $(UTILS))
```

This shows you both options. 
- The wildcard version means "drop a new `.c` in `system_programs/` and it builds automatically".
-  The hand list version means "I control exactly what gets built". For this PA the wildcard wins because the assignment encourages you to add your own programs. 

{:.highlight}  
For production-grade build systems, the hand list is often preferred so a stray file does not silently change the output.

### 2.4 The `all` target

```make
.PHONY: all
all: $(MAIN_EXEC) $(UTIL_BIN)
	@echo "shell build complete"
```


`.PHONY` declares `all` is not a file. Without it, if a file named `all` ever appeared in the directory, `make all` would think it was already up to date and do nothing. 

{:.important}
Always mark `all`, `clean`, `install`, `test`, and similar as `.PHONY`.

The `@` in front of `echo` suppresses Make's normal habit of printing each command *before* running it. Use it for diagnostic messages so the output is not "echo shell build complete" followed by "shell build complete".

### 2.5 The main executable rule

```make
$(MAIN_EXEC): $(MAIN_SRC) $(SRC_ROOT)/shell.h $(SRC_ROOT)/builtin.h \
              $(SRC_ROOT)/cmdline.h
	@mkdir -p $(BIN_DIR)
	$(CC) $(CFLAGS) -I$(SRC_ROOT) $^ -o $@
	@echo "Run ./$@ to start shell"
```

Two automatic variables to learn cold:

- `$@` is the target (here, `./cseshell`).
- `$^` is the full list of prerequisites (the `.c` and `.h` files).

The senior helpfully wrote a comment about `$<` (first prerequisite only) and `$^` (all of them) right in the Makefile. Memorise these. You will use them in almost every Makefile you ever write.

`-I$(SRC_ROOT)` adds `./source` to the include path so the headers can be found from `#include "shell.h"` without writing relative paths.

Note this rule compiles all the `.c` files together in one `gcc` invocation, producing one executable. That works for a small project. For larger ones, you would compile each `.c` to an `.o` separately and link the `.o` files. This sample's approach is fine here and keeps the Makefile readable.

### 2.6 The pattern rule for system programs

```make
$(BIN_DIR)/%: $(UTIL_DIR)/%.c $(SYSTEM_PROG_HDR)
	@mkdir -p $(BIN_DIR)
	$(CC) $(CFLAGS) -I$(SRC_ROOT) $^ -o $@
```

This is one rule that generates many binaries. The `%` is a pattern match: any target matching `bin/<something>` depends on `source/system_programs/<something>.c`. So `make bin/find` will compile `source/system_programs/find.c` and link to `bin/find`.

Combined with the `$(UTIL_BIN)` expansion from 2.3, this means the `all` target builds every system program without you naming them individually. Add `source/system_programs/myprog.c` and `make` builds `bin/myprog`. This is the payoff of writing a small amount of Make machinery once.

### 2.7 Clean

```make
.PHONY: clean
clean:
	rm -rf $(BIN_DIR)
	@echo "cleaned"
```

Notice what it does **not** remove: the `cseshell` binary at the project root, and any generated `tmp/`, `archive/`, or log files the shell creates at runtime. 

**This is preference.**

If you want a fuller reset, that is what `reset.sh` is for in this sample. Worth being aware of when you are *debugging* and `make clean && make` is not actually starting from a fresh state.


## The `CFLAGS`, every flag explained

This is the part Ryan said helped him most. Each flag has meaning and purpose. 

### `-Werror`

{:.highlight}
Treat every warning as an error. 

The compiler refuses to produce an executable if anything warns so that warning does not get ignored. This is <span class="orange-bold">harsh</span> and that is the point: a warning you ignore once will be ignored forever, and uninitialised variables, format string mismatches, and similar bugs slip through exactly this way. Use it from day one and you will write **cleaner** C.

**When to back off temporarily**: when you are stubbing out a function and have unused parameters everywhere. Either annotate them (`(void)argc;`) or comment out `-Werror` for the moment, then put it back.

Source: [GCC Warning Options](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html).

### `-Wall`

Despite the name, this is **not** "all warnings". 

{:.highlight}
It is a curated set that catches commonly useful problems: unused variables, missing return statements, format string issues, comparing signed and unsigned values in some cases, and so on. 

The actual "all warnings" flag is `-Wpedantic` plus dozens of individually named `-W` flags.

Source: [GCC Warning Options](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html).

### `-Wextra`

{:.highlight}
Adds **further** warnings on top of `-Wall`. 

Things like comparison of different signedness, missing field initialisers in struct literals, unused parameters. Some are noisy in real code, which is one reason they are separate from `-Wall`. Combined with `-Werror`, this triple forces you to think about every warning the compiler raises.

Source: [GCC Warning Options](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html).

### `-fsanitize=address` (AddressSanitizer, "ASan")

<span class="orange-bold">This is the single most useful flag in the list for this assignment</span>. 

ASan instruments memory accesses at compile time so that at runtime the program **detects** heap and stack buffer overflows, use-after-free, use-after-return, and double-frees. On gcc and clang for Linux, LeakSanitizer (memory leak detection) is bundled in by default and runs at exit.

**Cost**: roughly 2x *slowdown* and *increased* memory use. For a course PA this is irrelevant. **The benefit** outweighs the cost: instead of "the shell crashed", you get a stack trace pointing at the exact line where you read 1 byte past the end of a buffer. This will save you hours.

**How it interacts with the rest of the toolchain**: ASan needs `-g` to give you useful filenames and line numbers in the report. Do not run ASan-built binaries under `valgrind`; the two instrumentation systems fight each other and you should pick just one.

Ryan's `debug_scripts/` show both, but in practice ASan covered most of what they needed and Valgrind was rarely used. His `logvalgrind.log` confirms this.

Source: [Clang AddressSanitizer documentation](https://clang.llvm.org/docs/AddressSanitizer.html) (the gcc implementation is compatible and shares most of the same docs).

### `-g`

{:.highlight}
Emit debug symbols in DWARF format. 


Without `-g`, when you load the binary in gdb or when ASan prints a stack trace, you see only addresses. With `-g`, you see source file names, line numbers, and local variable names. There is no runtime cost: `-g` adds to the on-disk size of the binary but the debug info is not loaded unless a debugger asks for it.

Source: [GCC Debugging Options](https://gcc.gnu.org/onlinedocs/gcc/Debugging-Options.html).

### `-Og`


{:.highlight}
Optimise for debugging experience. 

**Per the GCC manual**: "Optimize debugging experience. `-Og` should be the optimization level of choice for the standard edit-compile-debug cycle, offering a reasonable blend of optimization, fast compilation and debugging experience especially for code with a high abstraction penalty."

In practice this means MOST of your variables are still in scope and inspectable in gdb, line numbers map cleanly to source, and you get some easy optimisations (dead code elimination, basic register allocation) that make the binary not painfully slow. It is the right choice while you are developing.

Note `-Og` does **not** imply `-g`; you still need to pass `-g` yourself if you want debug symbols. The senior did.

Source: [GCC Optimize Options](https://gcc.gnu.org/onlinedocs/gcc/Optimize-Options.html).

### `-D_POSIX_C_SOURCE=200809L` and `-D_GNU_SOURCE`


{:.highlight}
These are **feature test macros**, not GCC options. They tell the glibc headers which API surface to expose. 


Without them, calling `getline`, `strdup`, `setenv`, `kill`, `sigaction`, or various other functions may produce "implicit declaration" warnings (which become errors under `-Werror`).

- `_POSIX_C_SOURCE=200809L` exposes POSIX.1-2008 declarations.
- `_GNU_SOURCE` exposes everything POSIX, plus X/Open, plus glibc-specific extensions like `asprintf`, `program_invocation_name`, and the GNU version of `basename`.

**One subtlety point**: per the glibc `feature_test_macros(7)` man page, defining `_GNU_SOURCE` "implicitly defines... `_POSIX_C_SOURCE` with the value 200809L" on modern glibc. So passing both is actually redundant but harmless. If you are aiming for portability beyond glibc, prefer `_POSIX_C_SOURCE=200809L` alone and only add `_GNU_SOURCE` if you genuinely use GNU extensions.

Ryan linked the relevant glibc documentation right in the Makefile comment. That is exactly the RIGHT MOVE when you make a choice you might not remember the reason for in three weeks.

Source: [feature_test_macros(7), man7.org](https://man7.org/linux/man-pages/man7/feature_test_macros.7.html); [GNU C Library: Feature Test Macros](https://sourceware.org/glibc/manual/latest/html_node/Feature-Test-Macros.html).

### What these flags actually catch

This section lays out concrete bugs that students writing a shell typically hit, and which **flag** flags them. Each example is small enough to paste into a scratch file and try yourself.

#### Format string mismatch (`-Wall`, `-Wformat`)

```c
pid_t pid = fork();
printf("child pid: %s\n", pid);  // %s expects char *, got int
```

```
warning: format '%s' expects argument of type 'char *',
         but argument 2 has type 'pid_t' {aka 'int'}
```

Under `-Werror`, this stops the build. Without `-Wformat` (which `-Wall` includes), this compiles cleanly and crashes at runtime when `printf` tries to dereference a pid as a string pointer.

#### Implicit function declaration (without `_POSIX_C_SOURCE` or `_GNU_SOURCE`)

```c
#include <stdio.h>
char *line = NULL;
size_t n = 0;
ssize_t r = getline(&line, &n, stdin);
```

Without the feature test macros:

```
warning: implicit declaration of function 'getline'
warning: initialization of 'ssize_t' from 'int' makes integer from
         pointer without a cast
```

The second warning is the dangerous one. Without a declaration, the compiler assumes `getline` returns `int`, which on 64-bit systems silently truncates the real return value. Adding `-D_POSIX_C_SOURCE=200809L` makes the declaration visible and both warnings disappear. Same story for `strdup`, `setenv`, `kill`, `sigaction`, `nanosleep`, and most of the POSIX surface this shell uses.

#### Unused parameter (`-Wextra`)

```c
static int bi_exit(int argc, char **argv) {
    exit(0);
}
```

```
warning: unused parameter 'argc'
warning: unused parameter 'argv'
```

Fix it with `(void)argc; (void)argv;` at the top of the function, or with `__attribute__((unused))` on the parameter. Do not silence with a cast or by removing the parameters; the signature is fixed because `bi_call` dispatches through a function pointer table.

#### Sign comparison (`-Wextra`, `-Wsign-compare`)

```c
int n = read(fd, buf, sizeof(buf));
if (n < sizeof(buf)) {           // n is int, sizeof yields size_t
    handle_short_read();
}
```

```
warning: comparison of integer expressions of different signedness:
         'int' and 'size_t'
```

This is not pedantry. If `read` returns `-1` on error, that `-1` promotes to `SIZE_MAX` when compared against a `size_t`, so the "short read" branch runs with `n == -1` and the rest of the code treats a failed read as a normal short read. The fix is to check `n < 0` first, then compare the positive case.

#### Missing return value (`-Wall`)

```c
int parse_line(char *line) {
    if (line == NULL) return -1;
    tokenise(line);
    // forgot to return 0 on the success path
}
```

```
warning: control reaches end of non-void function
```

The function returns whatever happens to be in the return register, so the caller sees garbage. Under `-Werror`, this is caught at compile time instead of after an hour of "why does my shell think every line errored".

#### Maybe uninitialized (`-Wall`, but only with optimisation on)

```c
int status;
if (waited)
    waitpid(pid, &status, 0);
printf("status was %d\n", status);
```

```
warning: 'status' may be used uninitialized
```

This warning only fires when optimisation is on, because the dataflow analysis that detects it runs as part of the optimiser. With `-O0` the warning is silently missed. This is one of the reasons `-Og` matters even when you do not care about performance: it turns on enough of the optimiser to surface these bugs.

#### Stack buffer overflow (ASan, runtime)

```c
char prompt[8];
strcpy(prompt, "speedrun> ");  // 10 bytes copied into 8-byte buffer
```

ASan output (abbreviated):

```
==12345==ERROR: AddressSanitizer: stack-buffer-overflow
WRITE of size 11 at 0x7ffe... thread T0
    #0 strcpy
    #1 type_prompt at shell.c:97
    #2 main at shell.c:451

Address 0x7ffe... is located in stack of thread T0 at offset 16
in frame
  This frame has 1 object(s):
    [0, 8) 'prompt' (line 95) <== Memory access overflows
```

Without ASan, this either appears to work (because the stack has slack) or silently corrupts an adjacent local variable, and you spend an afternoon chasing a "random" segfault three function calls later.

#### Heap buffer overflow (ASan, runtime)

```c
char *line = malloc(IN_BUFFER);
line[IN_BUFFER] = '\0';   // last valid index is IN_BUFFER - 1
```

```
==12345==ERROR: AddressSanitizer: heap-buffer-overflow
WRITE of size 1 at 0x60200000... thread T0
0x60200000... is located 0 bytes after 512-byte region
[0x602000000040, 0x602000000240)
allocated by thread T0 here:
    #0 malloc
    #1 read_line at shell.c:200
```

The "0 bytes after" is the classic off-by-one signature.

#### Use after free (ASan, runtime)

```c
free(hist[i]);
size_t len = hist[i]->len;   // hist[i] is dangling
```

```
==12345==ERROR: AddressSanitizer: heap-use-after-free
READ of size 8 at 0x60200000... thread T0
    #0 hist_dump at cmdline.c:120

freed by thread T0 here:
    #0 free
    #1 hist_free at cmdline.c:74

previously allocated by thread T0 here:
    #0 malloc
    #1 history at cmdline.c:50
```

You get three stack traces in one report: where the bad access happened, where the memory was freed, and where it was originally allocated. That is usually enough to fix the bug in one read.

#### Double free (ASan, runtime)

Easy to hit when two code paths each think they own a string. The `argv_allocated` flag in `shell.c` exists precisely to avoid this in the alias-expansion path.

```c
free(argv[i]);
// ... later, in a cleanup path that also iterates argv:
free(argv[i]);
```

```
==12345==ERROR: AddressSanitizer: attempting double-free on 0x602...
```

#### Memory leak (LeakSanitizer at process exit, comes with ASan on Linux gcc)

```c
char *line = strdup(user_input);
return 0;   // never freed
```

At normal program exit:

```
==12345==ERROR: LeakSanitizer: detected memory leaks
Direct leak of 16 byte(s) in 1 object(s) allocated from:
    #0 strdup
    #1 expand_alias at shell.c:312
SUMMARY: AddressSanitizer: 16 byte(s) leaked in 1 allocation(s).
```

LSan runs at clean shutdown via `atexit`. If your shell exits via uncaught signal (Ctrl-`\` for `SIGQUIT` without a handler, for example), LSan does not run and you miss the report. This is one practical reason the senior wired up `catch_and_restore` and the `atexit` cleanup chain: clean shutdown is what makes the instrumentation useful.

### Why `-Werror -Wall -Wextra -fsanitize=address` is the right combination

In 50.005, *you are learning systems programming*. The bugs you write are mostly <span class="orange-bold">not</span> algorithm bugs; they are pointer bugs, off-by-one bugs, uninitialised memory bugs, and incorrect format strings. 

The compiler catches A LOT of these statically with the three `-W` flags. ASan catches the rest at runtime. Together they turn three hours of "why is my shell segfaulting" (and endless AI tokens) into 10 minutes of "ooo, I forgot to null-terminate after `strncpy`".

The cost is that early in the project you will hit A LOT of warnings and that would not make you feel good instantly. Please treat each one as a free code review and fix them. *Do not silence them with casts*.

## Interesting bits in the source worth studying

These are concrete techniques that Ryan used that go beyond what the base PA requires. Read them once for general understanding, then decide which ones, if any, suit your own design.

The sections are numbered for easier reference.

### 4.1 `resolve_project_root` via `/proc/self/exe`

In `shell.c`:

```c
ssize_t n = readlink("/proc/self/exe", path, sizeof(path) - 1);
```

`/proc/self/exe` is a **symlink** on Linux that points to the running executable's absolute path. Reading it gives you the location of the binary regardless of where the user ran it from. The senior uses this to find `./bin`, `./tmp`, and the rc file relative to the binary rather than relative to the current working directory. This means the shell works the same way whether you launch it from the project root or from `/`.

**Caveat**: this is Linux specific. On macOS you would use `_NSGetExecutablePath`. On BSDs, `sysctl`. If portability matters, abstract this.

### 4.2 Raw terminal mode in `cmdline.c`

```c
struct termios raw = orig;
raw.c_iflag &= ~(IXON);
raw.c_lflag &= ~(ECHO | ICANON);
raw.c_cc[VMIN] = 1;
raw.c_cc[VTIME] = 0;
tcsetattr(STDIN_FILENO, TCSAFLUSH, &raw);
```

This puts the terminal into raw mode so the shell receives each keypress immediately *rather than only after Enter*. Clearing `ICANON` disables line buffering. Clearing `ECHO` stops the terminal from echoing keypresses so the shell can echo them in its own way. `VMIN=1, VTIME=0` makes `read()` return as soon as one byte is available.

The pair `line_raw_init()` / `line_raw_cleanup()` save and restore the original termios. The matching `catch_and_restore` signal handler ensures that if the shell is killed by a signal, the terminal is restored to cooked mode before exit. Without that, killing the shell mid-raw-mode leaves your real terminal stuck and you have to type `stty sane` blind. The senior's `fixbroken.sh` exists precisely for the case where that escape hatch is needed.

Read this whole flow carefully. The base PA1 handout does not indicate that you require raw mode, but if you **choose** to add arrow-key history later, you <span class="orange-bold">will need</span> exactly this pattern.

### 4.3 Signal discipline around `fork`

In `shell.c`:

```c
void fork_child(char **argv) {
    pid_t pid = fork();
    if (pid == 0) { // child
        restore_cooked();
        tty_restore_ctrl_c();
        execvp(argv[0], argv);
        perror(argv[0]);
        _exit(errno == ENOENT ? 127 : 126);
    }
    int status;
    waitpid(pid, &status, WUNTRACED);
    ...
}
```

The shell itself <span class="orange-bold">ignores</span> `SIGINT`, `SIGQUIT`, and `SIGTSTP` so that Ctrl-C does not kill the shell. But child processes need normal signal behaviour: `cat` should die on Ctrl-C. The child restores default handlers right after `fork` and before `execvp`. The parent keeps its raw mode and signal-ignoring posture. This is the standard shell pattern; learn it.

The exit codes `126` and `127` are the conventional bash ones for "command found but not executable" and "command not found" respectively. Ryan linked the Bash manual section in a comment.

### 4.4 The builtin dispatch table

In `builtin.c`:

```c
static struct builtin lookup_bifn[] = {
    {"exit", "Exit the shell", bi_exit},
    {"cd",   "Change the current directory", bi_cd},
    ...
    {NULL, NULL, NULL}};
```

A simple array of name-description-function triples, terminated by a sentinel `NULL` row. The `bi_call` function walks it linearly. This is <span class="orange-bold">the right level of complexity</span> for ten or so builtins. Do not reach for a hash table here; it would be slower in practice because the linear scan over ten string pointers fits in cache.

Use this same pattern when you add your own builtins. It keeps `bi_call` short and adding a new builtin is one line in the table plus one function definition.

### 4.5 `atexit` cleanup chain

```c
void cleanup_handlers(void) {
    atexit(alias_free);
    atexit(hist_free);
    atexit(line_raw_cleanup);
    atexit(intentional_terminate_log);
    signal(SIGTERM, catch_and_restore);
    signal(SIGQUIT, catch_and_restore);
}
```

`atexit` registers functions to run on normal `exit()` or return from `main`. They run in reverse order of registration (LIFO). This is a clean way to ensure terminal restoration and memory cleanup without plumbing cleanup calls through every exit path. Note that `atexit` does **not** run if the process is killed by a signal, which is why the signal handlers do their own cleanup before `_exit`.



## Things to read critically

No senior sample is perfect, so the next time you find some PA1 implementation online, please read them carefully. The point of including this section is that you SHOULD read code with the same critical eye you would apply to your own. Spotting these is part of the exercise.

The section below is numbered for clarity.

### 5.1 The fallback `strncpy` size bug in `shell.c`

```c
strncpy(project_bin, project_bin_fallback,
        sizeof(*project_bin_fallback));
```

`project_bin_fallback` has type `const char *`. So `*project_bin_fallback` is a single `char`, and `sizeof(char)` is `1` by definition. This copies exactly one byte. The intent was almost certainly `sizeof(project_bin)` or `strlen(project_bin_fallback) + 1`. 


<span class="orange-bold">This is a real bug!</span> It happens to be in a fallback path that is rarely exercised, which is precisely why bugs hide there.

{:.note}
`sizeof` on a pointer gives you the pointer's size (usually 8 on x86-64), and `sizeof` on a **dereferenced** pointer gives you the size of the pointed-to type, which for `char *` is always 1. To get the buffer length, use `sizeof` on the **array**, <span class="orange-bold">not</span> the pointer or its target.

### 5.2 Dead Windows branch in `clear_screen`

The `#ifdef _WIN32` branch in `clear_screen()` contains:

- `::GetLastError()` (C++ scope resolution, not legal in C)
- `return ::GetLastError()` from a `void` function
- `"]" SetConsoleMode(...)` (a stray string literal next to a function call)

This code is <span class="orange-bold">never</span> compiled on Linux because `_WIN32` is not defined. Ryan might have probably pasted it from a Microsoft docs page and never tested it. It is a useful reminder: <span class="orange-bold">untested code is broken code.</span> If you keep a portability stub, at least put `#error "Windows path not implemented"` inside it so future you cannot accidentally rely on it.

### 5.3 `source_rc` is called twice in `startup`

```c
source_rc();
check_tempdir();
check_archivedir();
clear_screen();
tty_flags_sancheck(1);
source_rc();           // called a second time
line_raw_init();
```

And there is an honest comment a few lines below `line_raw_init`:

```c
line_raw_init(); // magically need to do it twice wtf,
                 // something inbetween breaks raw mode
```

**Do know that this is a workaround, not a fix**. Somewhere between the first `line_raw_init` and where the prompt is drawn, raw mode is being disabled by something else (probably one of the `check_*` or `source_rc` calls that fork children, since the child restores cooked mode and on glibc the parent's state can be affected by shared file descriptors and tty controlling terminal interactions).

If you find yourself writing "magically need to do X twice" in a comment, you have a bug, not a fix. Find the root cause. This is a good exercise for understanding how termios state is shared between processes that share a controlling terminal.

### 5.4 README has duplicate command

The README documents `reset.sh` but the example code block runs `./daemonslayer.sh` (copy-paste error from the previous bullet). And `dpspawn` vs `dspawn` is inconsistent between README and filename. Small things, but they erode trust. Proofread your own README, ideally by running every command in it on a <span class="orange-bold">freshly</span> cloned copy.

### 5.5 Tight coupling via global `project_root`

`shell.c`, `dplant.c`, and other system programs each declare their own `char project_root[PATH_MAX]` global and each implement their own `resolve_project_root()`. 

The logic is similar but NOT identical across files (the `dplant.c` version walks one directory up if it is in `bin/`, the `shell.c` version does not). When you have multiple near-copies of the same function, <span class="orange-bold">expect</span> some kind of divergence. 

A cleaner design would put `resolve_project_root` in a small shared utility translation unit linked into each program. The senior commented out exactly that approach (`#SYSTEM_UTIL_SRC = ...`) and went with the duplication route, presumably because each system program is otherwise self-contained and they did not want to introduce a shared link dependency. 

Trade-off is fair, but you should be aware of the cost. Don't just implement things cos its cool.

## How to study this sample without copying it

This is the most important section. The **Honour Code** rules and the live checkoff structure are designed around the expectation that you understand your own code line by line. You will not be able to bluff that if you have only read someone else's and not code the entire assignment on your own.

Suggested approach:

1. **Read the Makefile first, then the headers.** Headers tell you the public shape of each module without committing you to implementation choices. By the time you read `shell.c`, you should already have an idea of what its public functions do.
2. **Pick ONE feature you find interesting.** It can be the raw mode, the rc file loader, the builtin dispatch, the daemon family. **Just one**, we understand you don't have time for more. When you do, read it in full, then close the file, and re-derive the design from scratch on paper before you write any code.
3. **Implement the base PA your own way first.** Do NOT import any files from this sample. Type your own line reader, your own tokeniser, your own fork-exec loop. They will be uglier than this sample and that is fine.
4. **Then, once your own version works**, compare. Ask *why* Ryan made each different choice. Some of their choices are better than yours. Some of yours are better than theirs. The point is that you now have informed taste rather than a copy.

Specific things from this sample that are worth adopting in spirit (without copying line by line):

- The `CFLAGS` line. Use it from day one.
- The three-file split: process control, builtins, line editing.
- The builtin dispatch table pattern.
- The `atexit` cleanup chain.
- Linking the man page or the glibc docs in a comment when you make a non-obvious choice.

Things to definitely do differently:

- Test your portability stubs or do not write them.
- Do not paper over bugs with "magically need to do X twice".
- Centralise shared utilities. Duplicated `resolve_project_root` is a maintenance trap.
- Get the README right; juniors and graders both read it.
- Implement test cases (new in 2026)



## Quick reference: useful Make idioms from this file

To take away even if you read nothing else:

```make
# Build a list of source files automatically
SRCS = $(wildcard src/*.c)

# Derive object file names from source file names
OBJS = $(patsubst src/%.c,build/%.o,$(SRCS))

# Auto-variables you will use constantly
# $@  the target
# $<  the first prerequisite
# $^  all prerequisites (deduplicated)

# A pattern rule applied to many targets
build/%.o: src/%.c
	$(CC) $(CFLAGS) -c $< -o $@

# Always mark non-file targets phony
.PHONY: all clean
```

That alone covers most Makefiles you will write this term.
