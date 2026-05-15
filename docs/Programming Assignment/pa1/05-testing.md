---
layout: default
permalink: /pa1/part-test
title: Testing
description: Implement unit test and integration test
parent: Programming Assignment 1
grand_parent: Programming Assignment
nav_order:  6
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

Writing a working shell is just one aspect of programming. It is also equally important that you thoroughly test it, just like what you've learned from your other subject, 50.003 
There are two kinds of tests in this assignment:

```text
Unit tests:
Test one small function directly.

Integration tests:
Run the whole program and check whether it behaves correctly.
```

Example:

```text
Unit test:
Call parse_command("ls -l", &cmd) directly.

Integration test:
Run: printf "ls\nexit\n" | ./myshell
```

You are to **design your code** so that important logic can be tested <span class="orange-bold">without</span> always running the full shell.

### Project Structure

Your end project should follow this general structure:

```text
shell-project/
  include/
    parser.h
    builtins.h
    shell.h
    logger.h

  src/
    parser.c
    builtins.c
    shell.c
    logger.c
    main.c

  tests/
    unit/
      test_parser.c
      test_builtins.c
      test_logger.c

    integration/
      test_basic_shell.sh
      test_logging.sh

    unity/
      unity.c
      unity.h
      unity_internals.h

  scripts/
    run_integration_tests.sh
    gen_unit_tests.sh

  prompts/
    generate-unit-tests.md

  AGENTS.md
  Makefile
  README.md
```

Here's the main 3 locations that you should expand:

```text
src/ contains your program logic.
include/ contains function declarations.
tests/ contains code that checks whether your logic works.
```


## Code Placement Philosophy

You should not create any gigantic function that does everything:

```c
int main(void) {
    while (1) {
        // read input
        // parse command
        // execute command
        // handle logging
        // handle builtins
    }
}
```

This is a better structure:

```c
#include "shell.h"

int main(void) {
    return shell_loop();
}
```

You can put the real shell behavior in testable functions:

```c
// include/shell.h
#ifndef SHELL_H
#define SHELL_H

int shell_loop(void);
int shell_run_line(const char *line);

#endif
```

```c
// src/shell.c
#include "shell.h"

int shell_run_line(const char *line) {
    // parse and execute one line
    return 0;
}

int shell_loop(void) {
    // interactive loop
    return 0;
}
```

Now tests can call these functions without running the entire interactive shell manually:

```c
shell_run_line("echo hello");
```


## Unit Tests

{:.note}
Unit tests should focus on small, predictable functions.

These are good unit test targets, assuming you have files like these that implement specific functions for your shell. We gave you one sample in the starter code `test_perms.c`, `test_rc_parser.c`.

```text
parser.c
  tokenize()
  parse_command()
  parse_redirection()
  parse_pipeline()

builtins.c
  is_builtin()
  builtin_echo()
  builtin_pwd()

logger.c
  format_log_message()
  write_log_message()
  open_log_file()

path.c
  resolve_path()
  search_path()

process_info.c
  parse_proc_status()
  parse_ps_line()
```

These are good because they usually take clear input, produce clear output, and do not depend too heavily on operating-system behavior.


### Example Unit Test

Suppose you have this function declaration:

```c
// include/parser.h
#ifndef PARSER_H
#define PARSER_H

#define MAX_ARGS 32

typedef struct {
    char *argv[MAX_ARGS];
    int argc;
} command_t;

int parse_command(const char *input, command_t *cmd);
void free_command(command_t *cmd);

#endif
```

A unit test might look like this:

```c
// tests/unit/test_parser.c
#include "unity.h"
#include "parser.h"

void setUp(void) {}

void tearDown(void) {}

void test_parse_simple_command(void) {
    command_t cmd;

    int result = parse_command("ls -l", &cmd);

    TEST_ASSERT_EQUAL_INT(0, result);
    TEST_ASSERT_EQUAL_INT(2, cmd.argc);
    TEST_ASSERT_EQUAL_STRING("ls", cmd.argv[0]);
    TEST_ASSERT_EQUAL_STRING("-l", cmd.argv[1]);

    free_command(&cmd);
}

void test_parse_empty_command(void) {
    command_t cmd;

    int result = parse_command("", &cmd);

    TEST_ASSERT_NOT_EQUAL(0, result);
}
```



## Integration Tests

Some behavior is difficult to unit test in C because it depends on the operating system.

These are usually better for integration tests, commonly written as bash scripts (see starter code).

```text
fork()
execvp()
pipes
signals
daemon creation
terminal behavior
process groups
background jobs
actual filesystem side effects
```

You can still test these behaviors, but usually by running the whole program or by creating a special test-friendly mode.


### Example integration test

An integration test runs your compiled shell.

```bash
#!/usr/bin/env bash
set -e

OUTPUT=$(printf "echo hello\nexit\n" | timeout 3s ./myshell)

echo "$OUTPUT" | grep "hello" > /dev/null
```

Save this as:

```text
tests/integration/test_basic_shell.sh
```

The `timeout` prevents the test from hanging forever if your shell gets stuck.



## Testing logging or daemon behavior

Programs involving logging, daemonizing a process, or a `ps`-like program should be broken down into smaller functions and should not be unit tested directly.

### Logging example

Write functions like these that can be unit tested:

```c
int format_log_message(const char *event, char *buffer, size_t size);
int write_log_message(const char *path, const char *message);
```

Example unit tests:

```text
format_log_message() produces the expected text.
write_log_message() creates a file.
write_log_message() appends a second line.
write_log_message() returns an error for an invalid path.
```

Then use integration tests for the full program:

```bash
#!/usr/bin/env bash
set -e

rm -f test.log

printf "log hello\nexit\n" | timeout 3s ./myshell --log test.log

grep "hello" test.log > /dev/null
```

### Daemon example

Daemon behavior is <span class="orange-bold">harder</span> because a daemon may fork, detach from the terminal, close file descriptors, and keep running in the background.

To make this testable, your program should support a test-friendly mode that doesn't go to the background or just fork once to demonstrate the functionality:

```bash
./mydaemon --foreground --log test.log
```

or:

```bash
./mydaemon --once --log test.log
```

Recommended design:

```text
Unit test:
  Test log formatting.
  Test config parsing.
  Test PID file path generation.
  Test command-line argument parsing.

Integration test:
  Start the program.
  Wait briefly.
  Check that the log file was written.
  Stop the process.
```

Example integration test:

```bash
#!/usr/bin/env bash
set -e

rm -f daemon.log

timeout 3s ./mydaemon --foreground --log daemon.log &
PID=$!

sleep 1

kill "$PID" || true

grep "started" daemon.log > /dev/null
```

## Testing system-related reports like `ps` 

If you write a simple `ps` program, avoid hardcoding tests against the real system process list. The real process list changes constantly.

A better design is to again split it into smaller functions:

```c
int parse_proc_status_file(const char *path, process_info_t *info);
int read_process_info(const char *proc_root, int pid, process_info_t *info);
```

This is so that the unit test can use fake files to test its working:

```text
tests/fixtures/proc/123/status
```

Example:

```text
tests/
  fixtures/
    proc/
      123/
        status
```

Then your test calls:

```c
read_process_info("tests/fixtures/proc", 123, &info);
```


Integration tests can still run the real program, but they should only check *general* behavior:

```bash
OUTPUT=$(./myps)

echo "$OUTPUT" | grep "PID" > /dev/null
```

{:.note}
Do not require exact process IDs unless you control the process being tested.


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

{:.note}
If you're confused, see `/scripts/gen_unit_tests.sh` for details. It's self explanatory.

### `git add && git commit` first then `git diff`

This is very **important**. You should add and commit your changes first before asking the agents to modify your test files.

### AI workflow

When you add a new module or function:

```text
1. Write the function in src/.
2. Declare it in include/.
3. Run:

   make ai-unit-tests MODULE=[module-name]

4. Review the generated file in tests/unit/.
5. Run:

   make unit

6. Fix your code or tests.
7. Commit only tests you understand.
```

Useful commands:

```bash
make ai-unit-tests MODULE=parser
make unit
git diff
```

The agent may draft tests, but you are responsible for them. To survive live checkoffs, do <span class="orange-bold">not</span> write any test you can't explain.


### Suggested AI prompt

If your tool asks for a prompt, use something like this:

```text
Generate Unity unit tests for the new or modified functions in this C project.

Rules:
1. Only create or edit files under tests/unit/.
2. Do not modify src/ or include/.
3. Use the public function declarations in include/*.h.
4. Write tests for normal cases, edge cases, and invalid input.
5. Do not directly unit test fork(), execvp(), signals, terminal behavior, or daemon detachment.
6. Mark generated tests with a comment saying they are AI-generated drafts.
7. Keep the tests simple and readable.
```

For a specific file:

```text
Generate Unity unit tests for the functions in src/parser.c using include/parser.h.
Create or update tests/unit/test_parser.c.
Cover empty input, whitespace input, simple commands, multiple arguments, and invalid syntax.
Do not modify production code.
```



## Testing commands

The following commands are self-explanatory:

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

Runs the optional AI-assisted unit-test generator for one module, remember: one module at a time.

## Submission

You should submit all of these as part of PA1:

```text
1. Source code
2. Header files
3. Unit tests
4. Integration tests, if required by the assignment
5. A short testing note in README.md
```

Example testing note (we will read this during checkoffs):

```text
Testing note:

Unit tests:
- Tested parse_command with simple input, empty input, and multiple arguments.
- Tested is_builtin with cd, exit, pwd, and unknown commands.
- Tested log formatting with normal and empty messages.

Integration tests:
- Tested echo through the shell.
- Tested exit behavior.
- Tested logging output to a file.

AI use:
- AI was used to draft some unit tests.
- I reviewed and modified the tests before submission.
```


## Summary

We want you to gain some experience in testing your work thoroughly. The rule of thumb is that if a function is hard to test, that is often a sign that it is doing **too much**.

Split large function into smaller functions, but do not over-split either. It's kinda like an art, and you need to code and gain experience to get a feel of what is "small enough".

Also, keep `main()` small, and keep operating-system behavior separate from parsing, formatting, and validation logic.

Then:
- Use unit tests for small logic.
- Use integration tests for full-program behavior.

AI is very helpful to draft tests and catch edge-cases bugs once you guide them properly. However,AI may help draft tests, but you must understand, review, and run every test you submit.


