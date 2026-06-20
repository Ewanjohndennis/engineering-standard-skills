---
name: c-standards
description: Apply C-specific engineering standards when writing, reviewing, refactoring, or debugging C code. Use alongside code-generation-standards and code-review-standards — this skill adds C idioms, memory safety discipline, and language-specific pitfalls that the generic skills can't cover. Triggers on any C file, function, struct definition, or snippet. Applies to C99/C11 unless the project targets a specific standard.
---

# C Standards

> Language-specific standards for C. Apply on top of the generic code-generation-standards and code-review-standards skills — this narrows them to C idioms, memory safety, undefined behavior, and systems-level pitfalls. Defaults to C11 unless the project specifies otherwise.

---

## Memory management

Every `malloc`/`calloc`/`realloc` must have a corresponding `free` on every exit path — including error paths. When a function allocates and then hits an error midway, it must free everything allocated so far before returning. Draw the ownership boundary clearly: who allocates, who frees, and what happens on error.

Always check the return value of `malloc`/`calloc`/`realloc`. They return `NULL` on failure. Dereferencing a `NULL` pointer is undefined behavior and will crash or silently corrupt state depending on platform:

```c
// Bad
int *buf = malloc(n * sizeof(int));
buf[0] = 0; // UB if malloc returned NULL

// Good
int *buf = malloc(n * sizeof(int));
if (buf == NULL) {
    return ERROR_OOM;
}
```

Use `calloc` instead of `malloc` when zero-initialization is needed — it's clearer in intent and avoids a separate `memset`. Use `malloc` + explicit initialization only when zeroing is provably unnecessary.

After freeing a pointer, set it to `NULL` if there's any chance it could be used again. Use-after-free is undefined behavior and often silent.

---

## Buffer and string safety

Never use `strcpy`, `strcat`, `sprintf`, or `gets` with untrusted or variable-length input. Use the `n`-bounded variants: `strncpy`, `strncat`, `snprintf`. Even then, check that the result was not truncated when truncation would be incorrect:

```c
// Bad — truncation is silent
strncpy(dst, src, sizeof(dst));

// Good — truncation is checked
if (snprintf(dst, sizeof(dst), "%s", src) >= (int)sizeof(dst)) {
    return ERROR_TRUNCATED;
}
```

`strncpy` does not guarantee NUL-termination if the source is longer than `n`. Either ensure the last byte is always written (`dst[sizeof(dst)-1] = '\0'`) or prefer `snprintf`/`strlcpy` (where available).

When computing buffer sizes, check for integer overflow before the allocation. `n * sizeof(T)` overflows if `n` is large:

```c
// Bad — silent overflow if n is large
int *buf = malloc(n * sizeof(int));

// Good
if (n > SIZE_MAX / sizeof(int)) return ERROR_OVERFLOW;
int *buf = malloc(n * sizeof(int));
```

---

## Undefined behavior

C has extensive undefined behavior (UB) that compilers are allowed to exploit for optimization in ways that break intuitions. Avoid:

- **Signed integer overflow**: use `unsigned` arithmetic or check before the operation. `INT_MAX + 1` is UB, not wrap-around.
- **Out-of-bounds array access**: always validate indices before use. Off-by-one errors are UB, not "probably fine."
- **Accessing freed memory**: free then NULL, and never pass freed pointers to any function.
- **Uninitialized reads**: initialize all variables. Local variables in C have indeterminate value, not zero.
- **Strict aliasing violations**: don't cast a `float *` to `int *` to inspect the bits; use `memcpy` into the target type instead.
- **Pointer arithmetic outside arrays**: pointer arithmetic is only defined within a single array object (and one past the end). Don't subtract unrelated pointers.

Enable and fix all warnings at `-Wall -Wextra`. Treat `-Werror` as the target state. Use sanitizers during development: `-fsanitize=address,undefined` catches memory errors and UB at runtime that static analysis misses.

---

## Error handling

C has no exceptions. Every function that can fail must communicate failure through its return value or an output parameter. Be consistent within a module — don't mix conventions.

Common patterns (pick one per codebase and stick to it):
- Return `0` for success, negative error code for failure (common in POSIX/kernel style)
- Return a pointer, `NULL` on failure (with `errno` or an output error code for details)
- Return an explicit result enum or struct

Always check return values from standard library functions that can fail: `fopen`, `fclose`, `fread`, `fwrite`, `malloc`, `realloc`, `pthread_*`, `close`, `write`. Ignoring them silently is a common source of data loss and resource leaks.

When an error propagates up, preserve the original error code rather than replacing it with a generic one. The caller needs enough information to log or handle it correctly.

---

## Structs and types

Use `typedef struct` to avoid the `struct Foo` repetition, but give the struct a tag too — it allows forward declarations:

```c
// Allows forward declaration in headers
typedef struct Foo {
    int x;
    int y;
} Foo;
```

Use opaque pointers (forward-declared structs with implementation hidden in `.c` files) for public APIs where callers should not depend on the internal layout. This is the C equivalent of encapsulation.

Use `const` aggressively on pointer parameters that the function should not modify through. `const char *str` communicates that the function is a reader, not a writer. Don't cast away `const`.

Use `size_t` for sizes and counts. Use `ptrdiff_t` for pointer differences. Use `uint8_t`, `int32_t`, and friends from `<stdint.h>` when the exact width matters (protocol buffers, file formats, hardware registers). Don't assume `int` is 32 bits.

---

## Header discipline

Every header must be idempotent — wrap the entire contents in an include guard or `#pragma once`. Prefer include guards for portability:

```c
#ifndef MY_MODULE_H
#define MY_MODULE_H
// ...
#endif /* MY_MODULE_H */
```

Headers should declare only what callers need: function prototypes, types, and constants that form the public API. Implementation details, `static` helper functions, and internal types belong in the `.c` file. Don't include headers in headers unless the type is actually used in the public API — use forward declarations where possible to reduce coupling and compilation time.

Never define variables with external linkage in a header. Declare with `extern` in the header, define once in a `.c` file.

---

## Resource and fd safety

Track file descriptors with the same discipline as heap memory — every `open`/`socket`/`pipe` needs a matching `close` on every exit path. File descriptor leaks in long-running processes exhaust the fd table.

In code that returns early on error, use `goto cleanup` patterns to consolidate cleanup:

```c
int result = ERROR_NONE;
FILE *f = fopen(path, "r");
if (!f) { return ERROR_IO; }

char *buf = malloc(SIZE);
if (!buf) { result = ERROR_OOM; goto cleanup; }

// ... use f and buf ...

cleanup:
    free(buf);
    fclose(f);
    return result;
```

This is idiomatic C for multi-resource cleanup and is clearer than deeply nested if-else or duplicated cleanup code.

---

## Concurrency (pthreads / platform threads)

Always pair `pthread_mutex_lock` with `pthread_mutex_unlock` on every exit path — including error paths and early returns. Document which mutex protects which data.

Use `pthread_mutex_t` initialized with `PTHREAD_MUTEX_INITIALIZER` for static mutexes; use `pthread_mutex_init` for dynamic ones, and `pthread_mutex_destroy` when done.

Avoid global mutable state wherever possible. When it's unavoidable, protect all access with a mutex and document it. Signal handlers and threads sharing data need explicit synchronization — compiler optimizations can reorder reads/writes in ways that break lockless assumptions; use `volatile` only as a signal to the compiler, not as a synchronization primitive (`volatile` is not a memory barrier).

---

## Common pitfalls

- **`sizeof(ptr)` vs `sizeof(*ptr)`**: `sizeof(arr)` where `arr` has decayed to a pointer gives the pointer size, not the array size. Pass the size explicitly.
- **`char` signedness**: `char` may be signed or unsigned depending on the platform. Use `unsigned char` for byte buffers; use `int` for `getchar()`/`fgetc()` return values (to detect `EOF` = -1 correctly).
- **`strtol`/`strtod` vs `atoi`**: `atoi` has no error detection. Use `strtol` with `errno` checking and end-pointer validation for all numeric parsing.
- **Format string mismatches**: `%d` for a `long` is UB; use `%ld`. `%zu` for `size_t`. Enable `-Wformat` (included in `-Wall`) and treat warnings as errors.
- **Sequence point violations**: `a[i] = i++` is UB. Don't modify and use a variable in the same expression.
- **`realloc` failure**: `ptr = realloc(ptr, new_size)` leaks the original block if `realloc` returns `NULL`. Always use a temporary: `tmp = realloc(ptr, new_size); if (tmp) ptr = tmp; else /* handle error */`.
