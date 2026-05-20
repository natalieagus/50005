---
layout: default
permalink: /labs/01-c-primer
title: C Primer
description: Introduction to C 
parent: Labs
nav_order:  2
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

# C Primer
{: .no_toc}

This notes will be used in Week 2 and Week 3 Labs when we introduce C and finish up lecture theory in class. You should refer to this notes whenever possible to not make beginner mistakes when writing in C.

During the lab, we will use [this](https://github.com/natalieagus/50005-C-exercises) exercises. Special thanks to [Ryan Pek](https://github.com/rpeky/50005-C-exercises), a 2027 CSD graduate who created this exercise.

## Header `.h` files vs `.c` files in C

In C, a header file should usually describe **what other files are allowed to use**. A `.c` file should usually contain the actual **implementation**.

{:.note-title}
> General Rule 
> 
> Put declarations in `.h` files. Put definitions in `.c` files.

This matters because `#include` literally copies the header file into the `.c` file before compilation. If many `.c` files include the same header, they all receive the same header content.

So if you define a global variable inside a header, every `.c` file gets its own copy. That commonly causes linker errors such as:

```text
duplicate symbol '_something'
```

Below we list things that belong in the header file.

### Function Declaration 
A function declaration tells C that this function exists somewhere.

Example header:

```c
// math_utils.h
#ifndef MATH_UTILS_H
#define MATH_UTILS_H

int add(int a, int b);
int multiply(int a, int b);

#endif
```

The function bodies go in the `.c` file:

```c
// math_utils.c
#include "math_utils.h"

int add(int a, int b) {
    return a + b;
}

int multiply(int a, int b) {
    return a * b;
}
```

Another file can use these functions:

```c
// main.c
#include <stdio.h>
#include "math_utils.h"

int main(void) {
    printf("%d\n", add(3, 4));
    return 0;
}
```



The header says what functions exist and the `.c` file (that includes it) contains the actual implementation.


### Global variables

Suppose you want to share a global counter between multiple `.c` files. Do **not** write this in the header:

```c
// counter.h
int total_requests = 0;   // This is bad
```

This is a <span class="orange-bold">definition</span>. It creates real **storage**.

{:.important-title}
> Why definition in header file is bad
> 
> If both `main.c` and `server.c` include this header, then both files define their own `total_requests`.

That causes a <span class="orange-bold">duplicate</span> symbol error. Some people however don't care about this because they know that this header file will only be included by exactly 1 `.c` file so in this case that's okay.

This is the good version:

```c
// counter.h
#ifndef COUNTER_H
#define COUNTER_H

extern int total_requests;

void increment_requests(void);

#endif
```

Then define the variable in exactly one `.c` file:

```c
// counter.c
#include "counter.h"

int total_requests = 0;

void increment_requests(void) {
    total_requests++;
}
```

Now other files can use the same shared variable:

```c
// main.c
#include <stdio.h>
#include "counter.h"

int main(void) {
    increment_requests();
    increment_requests();

    printf("Total requests: %d\n", total_requests);

    return 0;
}
```

The key difference is that this means this variable exists somewhere else:

```c
extern int total_requests;
```

But the following means: create the actual variable here.


```c
int total_requests = 0;
```
### Arrays

The same rule applies to arrays. This is a <span class="orange-bold">bad</span> header:

```c
// colors.h
const char *colors[] = {
    "red",
    "green",
    "blue"
};
```

This creates the actual array every time the header is included.

This is the correct header:

```c
// colors.h
#ifndef COLORS_H
#define COLORS_H

extern const char * const colors[];
extern const int NUM_COLORS;

#endif
```

And the correct `.c` file:

```c
// colors.c
#include "colors.h"

const char * const colors[] = {
    "red",
    "green",
    "blue"
};

const int NUM_COLORS =
    sizeof(colors) / sizeof(colors[0]);
```

You can use it elsewhere freely too:

```c
// main.c
#include <stdio.h>
#include "colors.h"

int main(void) {
    for (int i = 0; i < NUM_COLORS; i++) {
        printf("%s\n", colors[i]);
    }

    return 0;
}
```

This is safe because only `colors.c` creates the real array.


### Private variables should stay in the `.c` file

Not everything needs to be exposed through a header. Suppose `logger.c` has an internal log level.

```c
// logger.c
#include <stdio.h>

static int log_level = 1;

void log_message(const char *message) {
    if (log_level > 0) {
        printf("[LOG] %s\n", message);
    }
}
```

The `static` keyword at file scope means: *this variable is private to this `.c` file.*

Other files <span class="orange-bold">cannot</span> directly access `log_level`.

The header only exposes the public function:

```c
// logger.h
#ifndef LOGGER_H
#define LOGGER_H

void log_message(const char *message);

#endif
```

Other files do not need to know how the logger stores its internal state. They only need to call:

```c
log_message("Program started");
```


This is usually better design, but you probably won't need to care about it in this course since we don't go really deep into C.

### Structs and typedefs

Headers are good places to put <span class="orange-bold">shared</span> types.

Example:

```c
// point.h
#ifndef POINT_H
#define POINT_H

typedef struct {
    int x;
    int y;
} Point;

int point_distance_squared(Point p);

#endif
```

Implementation:

```c
// point.c
#include "point.h"

int point_distance_squared(Point p) {
    return p.x * p.x + p.y * p.y;
}
```

Usage:

```c
// main.c
#include <stdio.h>
#include "point.h"

int main(void) {
    Point p = {3, 4};

    printf("%d\n", point_distance_squared(p));

    return 0;
}
```

This is fine because the header defines a type, not a global variable with storage. 

#### About `typedef`

{:.note}
`typedef` gives an existing type a new name.

Example:

```c
typedef unsigned int uint;
```

Now instead of writing:

```c
unsigned int age;
```

you can write:

```c
uint age;
```

Most common use is with `struct`. Without `typedef`, you need the keyword `struct` to declare it:

```c
struct Point {
    int x;
    int y;
};

struct Point p;
```

With `typedef`, it gets a little neater:

```c
typedef struct {
    int x;
    int y;
} Point;

Point p;
```

So this means: *create a struct type with `x` and `y`, and let me call that type `Point`*.

```c
typedef struct {
    int x;
    int y;
} Point;
```

It does **not** create a variable, only a type name.

### Constants

For simple integer constants, this is okay to put in a header:

```c
// config.h
#ifndef CONFIG_H
#define CONFIG_H

#define MAX_USERS 100
#define BUFFER_SIZE 1024

#endif
```

Another common style is to use `enum`:

```c
// config.h
#ifndef CONFIG_H
#define CONFIG_H

enum {
    MAX_USERS = 100,
    BUFFER_SIZE = 1024
};

#endif
```

Both are fine for simple compile-time constants, but <span class="orange-bold">avoid</span> putting actual global variables in the header:

```c
// config.h
int max_users = 100;   // BAD
```

Instead use this correct version:

```c
// config.h
#ifndef CONFIG_H
#define CONFIG_H

extern int max_users;

#endif
```

```c
// config.c
#include "config.h"

int max_users = 100;
```



### Include guards

Every header <span class="orange-bold">should</span> usually have an include guard:

```c
#ifndef FILE_NAME_H
#define FILE_NAME_H

// declarations here

#endif
```

Example:

```c
// point.h
#ifndef POINT_H
#define POINT_H

typedef struct {
    int x;
    int y;
} Point;

int point_distance_squared(Point p);

#endif
```

The guard prevents the *same* header from being processed multiple times in one compilation unit.

For example:

```c
// main.c
#include "point.h"
#include "geometry.h"
```

If `geometry.h` also includes `point.h`, then `point.h` may be included twice.

The include guard prevents duplicate type definitions and repeated declarations inside the same `.c` file.

Meaning:

```c
#ifndef POINT_H
```

means:

```text
If POINT_H has not been defined yet, continue.
```

Then the following marks *this* header as already included so that the second time the compiler sees the same header, it skips the content.

```c
#define POINT_H
```


### Summary about Header files

If you don't want to read anything else, at least read these:

```c
// Good things for .h files

int add(int a, int b);                 // function declaration

extern int total_requests;             // global variable declaration

typedef struct {
    int x;
    int y;
} Point;                               // shared type definition

#define BUFFER_SIZE 1024               // macro constant
```

And put these in `.c` files:

```c
// Good things for .c files

int add(int a, int b) {
    return a + b;
}

int total_requests = 0;

static int internal_state = 123;
```

<span class="orange-bold">Avoid</span> this in headers because these create actual storage, and every `.c` file that includes the header will define another copy:

```c
// Bad in .h files

int total_requests = 0;

const char *colors[] = {
    "red",
    "green",
    "blue"
};
```

So basically, a header file should usually contain:

```text
Declarations
Shared types
Constants
Function prototypes
extern variable declarations
```

A `.c` file should usually contain:

```text
Function bodies
Actual global variable definitions
Private helper functions
Private static variables
Implementation logic
```


The header tells other files what they can use and the `.c` file contains the actual code and storage.


## Pointers and Double Pointers in C

This is a quick note about `T *` vs `T **`, and what’s actually happening underneath, complete with snippets and sample code just enough to make you understand the basics. You still need to do more practice on your own to actually feel natural about it.

### The one rule about C

**C passes everything by value.** When you pass a pointer to a function, the function gets a *copy* of the pointer. Assigning to that copy changes only the local parameter, not the caller’s variable.

If you want a function to modify something the caller owns, you must give the function the **address** of that something. Then it can write through the address.

This single rule explains every use of double pointers.

### What each indirection level means

|Parameter |What it points to               |What the function can modify in the caller|
|----------|--------------------------------|------------------------------------------|
|`int x`   |a copy of x                     |nothing in the caller                     |
|`int *p`  |an int                          |the int                                   |
|`int **p` |a pointer to an int             |the pointer, and through it, the int      |
|`int ***p`|a pointer to a pointer to an int|the pointer to a pointer, and so on       |

Rule of thumb: **one more `*` in the parameter than the thing you want to mutate in the caller.**

### When `int *p` is the right choice

The function only needs to read or write the int(s). The caller’s pointer variable does not need to change.

**Modify a single value the caller owns:**

```c
void increment(int *p) { (*p)++; }

int x = 5;
increment(&x);   // x is now 6
```

**Iterate over an array (array decays to `int *`):**

```c
int sum(int *a, size_t n) {
    int s = 0;
    for (size_t i = 0; i < n; i++) s += a[i];
    return s;
}
```

**Return multiple values via out-parameters:**

```c
void divmod(int a, int b, int *q, int *r) {
    *q = a / b;
    *r = a % b;
}
```

**Fill a buffer the caller already allocated:**

```c
void read_line(char *buf, size_t cap);
```

### When `int **p` is the right choice

The function needs to change **the caller’s pointer itself**, not just the data behind it.

**Allocate on behalf of the caller:**

```c
int alloc_int(int **out) {
    *out = malloc(sizeof(int));
    if (!*out) return -1;
    **out = 42;
    return 0;
}

int *p;
alloc_int(&p);   // p is now a heap pointer
```

{:.important}
Why not `alloc_int(int *out)` and `out = malloc(...)` inside? Because `out` is a local copy. The malloc address is written into the local slot and lost when the function returns. The caller’s `p` never sees it. Result: leak.

**Grow a buffer that may move (`realloc`):**

```c
int grow(int **buf, size_t *cap) {
    int *tmp = realloc(*buf, (*cap * 2) * sizeof(int));
    if (!tmp) return -1;
    *buf = tmp;          // update caller's pointer if realloc moved the block
    *cap *= 2;
    return 0;
}
```

**Free and null in one helper:**

```c
void int_free(int **pp) {
    free(*pp);
    *pp = NULL;          // caller's pointer is nulled
}
```

**Linked list operations without head special-casing:**

```c
void list_remove(Node **cur, int key) {
    while (*cur) {
        if ((*cur)->key == key) {
            Node *dead = *cur;
            *cur = dead->next;   // patches previous node's next, or head
            free(dead);
            return;
        }
        cur = &(*cur)->next;
    }
}
```

**Array of pointers (not the same use case, just looks similar):**

```c
int main(int argc, char **argv);
```

Here the second `*` means “array of”; `argv[i]` is a `char *` (one C string). This is a different reason for the second indirection from the “modify caller’s pointer” cases above, but the syntax is the same.

### Memory Leak Example

```c
void alloc_int_BROKEN(int *out) {
    out = malloc(sizeof(int));   // writes to LOCAL copy
    *out = 42;
}

int *p = NULL;
alloc_int_BROKEN(p);             // p is still NULL, malloc'd block leaked
```

Here's what's happening:
1. Caller’s `p` sits at some address, say `0x7ff0`, holding `NULL`.
2. The call copies `NULL` into the function’s parameter slot `out`, at some other address, say `0x7fa0`.
3. `out = malloc(...)` writes the new heap address into `0x7fa0`. The caller’s `p` at `0x7ff0` is untouched.
4. Function returns. Parameter slot is destroyed. No surviving pointer to the heap block. Leak.

The fix is to pass `&p` and take `int **out`, so `*out = ...` writes back through the address of `p`.

### Storage class is independent of pointer level

The pointer indirection level says nothing about where the memory lives. Where the memory lives depends on how it was acquired.

|Source                |Storage          |Free?|
|----------------------|-----------------|-----|
|`int x;` local        |stack            |no   |
|`int a[10];` local    |stack            |no   |
|`int *p = malloc(...)`|heap             |yes  |
|`static` or global    |static           |no   |
|`"literal string"`    |static, read-only|no   |
|`strdup("...")`       |heap             |yes  |

**Common mistake**: returning a pointer to a stack local.

```c
int *bad(void) {
    int x = 42;
    return &x;          // x dies when bad() returns; pointer is dangling
}
```

### C has no garbage collector

Every `malloc` / `calloc` / `realloc` / `strdup` (and friends that return heap memory) must be matched by exactly one `free`.

Here's a simple guideline:

- Make ownership <span class="orange-bold">explicit</span>. For every allocation, one piece of code is responsible for **freeing** it.
- `realloc(p, n)`: on success the old `p` is invalid (even if not moved). On failure (`NULL` returned) the old `p` is still valid and still must be freed eventually.
- After `free(p)`, `p` is dangling. Set to `NULL` if it might be reused.
- Use AddressSanitizer (`-fsanitize=address`) or valgrind to catch leaks and use-after-free.



### One subtle point on `const`

`int *p` as a parameter does <span class="orange-bold">not</span> promise the function won’t write through it. If you want that promise:

```c
size_t my_strlen(const char *s);
```

`const` placement matters. `const int *` (pointer to const int), `int *const` (const pointer to int), and `const int *const` (const pointer to const int) all mean different things. For double pointers it gets thornier: `const int **`, `int *const *`, `int **const` are three different types. Worth its own note.

### Quick Summary


Here's one simple decision table to live by:

|The function needs to…          |Take                      |
|--------------------------------|--------------------------|
|Read or write one int           |`int *p`                  |
|Iterate over N ints             |`int *p, size_t n`        |
|Return several ints to caller   |several `int *` out params|
|Promise it will not write       |`const int *p`            |
|Allocate and hand back a new int|`int **p`                 |
|Reallocate (buffer may move)    |`int **buf`               |
|Null out the caller’s pointer   |`int **p`                 |
|Walk an array of `int *`        |`int **p`                 |


And if you don't read the above and just want to know what's important, here it is:
- Pass by value applies to pointers too. The function gets a copy.
- Use one more `*` than the thing you want to mutate in the caller.
- `int *p`: change the data. `int **p`: change the caller’s pointer.
- Storage class is orthogonal to pointer level. Stack/heap is decided by how the memory was acquired, not by how many stars the parameter has.
- No GC. Every heap allocation needs exactly one `free`. Ownership must be explicit.


## More C Behaviors
This section details a handful of C behaviors that explain most of the bugs students write and most of the *wtf* moments in production code. We do not include any niche stuffs, just stuffs that you might encounter during this term.

### Recap: pass by value

As mentioned avoce, everything in C is passed by value. When you pass a pointer, the function gets a **copy** of the pointer. To let a function modify the caller’s variable of type `T`, pass a `T *` (the address of that variable). Inside the function, write through it with `*`.

This is the rule behind double pointers, output parameters, and most of what looked weird in the previous note.

### Array decay

{:.note-title}
> Array decay 
> 
> In almost every expression, an array’s name is automatically converted to a pointer to its first element. The array’s “array-ness” (its size, the fact that it’s an array type) is dropped on the floor.

The standards literature calls this “conversion of array to pointer,” and informally people call it the array **decaying** into a pointer. The information loss is the key idea.


```c
int a[10];
int *p = a;          // a "decays" to &a[0], an int *
                     // identical to: int *p = &a[0];
```

```c
int a[10];
foo(a);              // foo receives an int *, not an array
```

```c
int a[10];
a[3] = 7;            // a decays to int *; a[3] is *(a + 3)
```

```c
void f(int x[]) { ... }      // looks like an array parameter
void f(int x[10]) { ... }    // looks more specific
void f(int *x)    { ... }    // these three are all the SAME function
```

The compiler *ignores* the `[]` or `[10]` in a parameter declaration. The 10 is documentation.

#### Where decay does NOT happen

These three exceptions:

**`sizeof`:**

```c
int a[10];
sizeof(a);           // 10 * sizeof(int), e.g. 40 bytes. NO decay.

void f(int a[10]) {
    sizeof(a);       // sizeof(int *), e.g. 8. Because the parameter
                     // is actually int *, and the 10 was a lie.
}
```

This is why you cannot find an array’s length inside a function from the parameter alone. You MUST pass the length separately.

**Address-of (`&`):**

```c
int a[10];
&a;                  // type: int (*)[10], pointer to array of 10 ints
                     // NOT int **, and NOT int *
a;                   // type after decay: int *
```

`&a` and `a` have the same numeric address, but different types. Pointer arithmetic on them advances by different amounts:

```c
a + 1;               // advances by sizeof(int)
&a + 1;              // advances by sizeof(int) * 10
```

**String literal initializing a char array:**

```c
char s[] = "hello";  // copies 6 bytes into s. No decay of the literal here.
char *p = "hello";   // p points to static read-only memory.
```

#### Consequences

These are a short summary of the consequences of array decay:
1. **You cannot pass an array by value.** Passing `a` passes a pointer. The function cannot see the size.
1. **You cannot return an array.** Same reason; functions return pointers, and returning a pointer to a local array is UB.
1. **`sizeof` only gives the array size where the compiler can still see the array type.** Usually that means the same scope as the declaration.
1. **`int **` is NOT how you pass a 2D array.** `int m[3][4]` decays to `int (*)[4]`, a pointer to an array of 4 ints. The right parameter is `int m[][4]` or `int (*m)[4]`.

```c
void f(int (*m)[4], int rows);    // correct
void f(int **m);                  // WRONG for int m[3][4]
```

### Undefined behavior (Undefined Behavior)

UB does not mean “might crash.” *It means the standard imposes no requirements*, and modern compilers ASSUME UB never happens, then optimize on that assumption. Code that worked at `-O0` can vanish, behave randomly, or appear to violate cause and effect at `-O2`.

This is a classic example:

```c
int foo(int x) {
    if (x + 1 < x) return 1;   // signed overflow is UB
    return 0;                   // compiler: "x + 1 < x cannot happen, return 0"
}
```

The compiler may eliminate the comparison entirely because it would only be true on signed overflow, which the standard says cannot happen.

Common sources to memorize:

- Signed integer overflow
- Dereferencing NULL, dangling, or uninitialized pointers
- Out-of-bounds array access
- Use-after-free, double-free
- Modifying a string literal
- Strict aliasing violations (see below)
- Data races

To tackle this: compiling with `-fsanitize=address,undefined` catches most of these at **runtime**.

### Integer promotions and signed/unsigned mixing

{:.note}
**Anything narrower than `int` gets promoted to `int` before arithmetic.** Mixed signed/unsigned operations convert the signed operand to unsigned.

```c
unsigned int u = 1;
int s = -1;
if (s < u) puts("yes");
else       puts("no");
// Prints: no
// s is converted to unsigned, becomes UINT_MAX, which is > 1.
```

```c
uint8_t a = 200, b = 200;
int c = a + b;         // a and b promoted to int. c = 400. No overflow.
```

```c
size_t n = some_value();
for (int i = 0; i < n; i++) { ... }
// i is signed, n is unsigned. The comparison converts i to size_t.
// If i goes negative for some reason, the loop runs nearly forever.
```

{:.important}
Never compare signed and unsigned without thinking about it. Either match types, or cast explicitly.

### Evaluation order is mostly unspecified

The order in which function arguments are evaluated, and most operands of most operators, is <span class="orange-bold">unspecified</span>. If two side effects on the same object happen without a sequence point between them, you get UB, which is a nightmare.

```c
int i = 0;
printf("%d %d\n", i++, i++);    // unspecified order; pre-C11 also UB
a[i] = i++;                     // UB
f() + g();                      // order of f and g calls unspecified
```

Here are the list of sequenced operators (safe between their operands):

- `&&` and `||` (short-circuit; also create a sequence point)
- `?:` (condition is fully evaluated before the chosen branch)
- `,` (the comma operator, not the comma between function arguments)

### Strict aliasing

You may access an object only through a pointer of a compatible type, or through `char *` / `unsigned char *`. Casting a pointer to an incompatible type and dereferencing it is UB, and the optimizer <span class="orange-bold">will reorder</span> loads and stores around it as if the aliasing never happens.

```c
float f = 1.0f;
int i = *(int *)&f;          // UB. Strict aliasing violation.
```

The correct way to reinterpret bits is using `memcpy`:

```c
int i;
memcpy(&i, &f, sizeof i);    // legal. Compilers turn this into a register move.
```

Unions are also allowed for type punning in C (this is one of the C-vs-C++ differences). This confuses people most often when reading binary file formats or doing low-level networking. Use `memcpy` instead and stop fighting the optimizer.

### Strings and string literals

```c
char *p = "hello";    // p points to static, read-only memory.
p[0] = 'H';           // UB. Probably segfault.

char s[] = "hello";   // s is a stack array, 6 bytes, mutable.
s[0] = 'H';           // fine.
```

Other things to take note of:

- `sizeof("abc")` is **4**. Includes the null terminator.
- `strlen("abc")` is **3**. Does not.
- `strncpy` does NOT guarantee null termination. If the source is longer than `n`, no `\0` is written. It also pads short sources with extra zeros. It is not a safe `strcpy`.
- `strncat`’s `n` is “max bytes to append,” not buffer size.
- `snprintf` returns the number of bytes it WOULD have written, which can exceed the buffer. Useful for sizing.

### Struct padding

The compiler **inserts** padding between members to satisfy alignment, and adds trailing padding so arrays of the struct stay aligned.

```c
struct S {
    char c;          // 1 byte
    int  i;          // 4 bytes, but needs 4-byte alignment
};
sizeof(struct S);    // typically 8, not 5
```

Here's the layout:

```
offset 0: c
offset 1..3: padding
offset 4..7: i
```

What consequences do this bring?

- `memcmp` on two structs may report inequality even when all members match, because padding bytes are indeterminate. Zero-init with `= {0}` or `memset` first if you must compare with `memcmp`.
- Reorder members from largest to smallest alignment to save space.
- Use `offsetof` from `<stddef.h>` to find member offsets portably. Never hand-compute.

### Reading declarations right-to-left from the name

C declaration syntax mirrors usage. Read <span class="orange-bold">outward</span> from the variable name following the precedence of `()`, `[]`, and `*`.

```c
int *a[10];          // a is an array of 10 pointers to int
int (*a)[10];        // a is a pointer to an array of 10 ints
int *f(void);        // f is a function returning int *
int (*f)(void);      // f is a pointer to a function returning int
int (*fp[5])(int);   // fp is an array of 5 pointers to functions (int) -> int
```

When in doubt, use `typedef`:

```c
typedef int (*handler_t)(int);
handler_t fp[5];     // much clearer
```

### Linkage: `static`, `extern`, file scope

You probably see some `c` files containing the word `static` or `extern`. Here's the basic rule:
- `static` at **file scope**: internal linkage. The symbol is private to this translation unit. Use this for helpers in a `.c` file.
- `static` at **block scope**: same name visibility, but the variable’s lifetime is the entire program and it’s zero-initialized once.
- `extern`: this is a declaration only, no storage. Used in headers to announce a variable defined in some `.c` file.
- File scope variable with NO `static`: external linkage by default. **Visible to all translation units**.

{:.note}
The most common bug you might make as beginner is to define a variable in a header. It is kinda acceptable for class projects, but NOT anywhere near production.

```c
// header.h
int global_counter;   // every .c that includes this defines its own,
                      // OR you get a multiple-definition link error.
```

Correct pattern:

```c
// header.h
extern int global_counter;   // declaration only

// one .c file
int global_counter;          // the single definition
```

### Preprocessor macros 

Some macros definition might not work the way you want them to:

```c
#define SQ(x) x*x
SQ(1 + 2);           // expands to: 1 + 2*1 + 2 = 5
```

Always parenthesize macro arguments AND the whole expansion:

```c
#define SQ(x) ((x)*(x))
SQ(1 + 2);           // ((1+2)*(1+2)) = 9
```

Beware side effects in macro arguments:

```c
#define MAX(a,b) ((a) > (b) ? (a) : (b))
MAX(i++, j++);       // the larger one gets incremented TWICE
```

When you can, do `static inline` functions. They’re type-safe, debuggable, and don’t evaluate arguments multiple times.

## `char` may be signed or unsigned

Whether plain `char` is signed or unsigned is <span class="orange-bold">implementation-defined</span>. On most ARM platforms it’s unsigned; on x86 it’s typically signed.

```c
char c = 0xFF;
if (c == 0xFF) { ... }   // false on platforms with signed char.
                          // c sign-extends to -1, compared to 255.
```

Here's some generic rules to live by peacefully:

- For text, use `char`.
- For bytes (binary data), use `unsigned char` or `uint8_t`.
- For indexing tables by character value, cast to `unsigned char` first:

```c
isalpha((unsigned char)c);    // the ctype functions require this
```

Passing a negative `char` to `isalpha` or similar is UB on systems with signed `char`.

## How to make life easier when writing in C?

Always build with this flags at all times:

```
-std=c11 -Wall -Wextra -Wpedantic -Wshadow -Wconversion
-Wstrict-prototypes -Wmissing-prototypes
```

For test and debug builds, add sanitizers:

```
-fsanitize=address,undefined -fno-omit-frame-pointer -g
```

Most C bugs that ship to production were warnings the developer ignored, so don't ignore the warnings and start fixing them.


## Quick Summary

If you do not read anything else, hopefully this table is useful for you.

|Topic            |Content                                                 |
|------------------|---------------------------------------------------------|
|Pass by value     |Function modifies a copy, caller sees nothing            |
|Array decay       |`sizeof` lies in functions; `int **` is not 2D           |
|Undefined behavior|Compiler assumes UB cannot happen, deletes your code     |
|Integer promotions|Signed vs unsigned comparisons go wrong silently         |
|Evaluation order  |`f(i++, i++)` is unspecified or UB                       |
|Strict aliasing   |Type-punning casts break under optimization; use `memcpy`|
|String literals   |Read-only; modifying is UB                               |
|Struct padding    |`sizeof` is not the sum of member sizes; `memcmp` lies   |
|Declarator syntax |`int *a[10]` vs `int (*a)[10]` are different             |
|Linkage           |Variables in headers cause multi-def or duplication      |
|Macros            |Always parenthesize; never put side effects in args      |
|`char` signedness |Implementation-defined; cast to `unsigned char` for bytes|

