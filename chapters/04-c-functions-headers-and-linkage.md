[Back to notes index](../README.md)

| [Previous: Expressions and control flow in C](03-expressions-and-control-flow-in-c.md) | [Notes index](../README.md) | [Next: C arrays and strings](05-c-arrays-and-strings.md) |
| --- | --- | --- |
# 4. C functions, headers, and linkage

Functions give a program named operations with explicit inputs and results. Headers publish declarations so other source files can call those functions. Linkage determines whether names in different translation units refer to the same program entity.

## Function declarations and definitions

A function declaration states its name, parameter types, and return type. A definition provides the body. In C, a function with no parameters should declare void in its parameter list. This gives the compiler a complete prototype for calls.

C passes argument values to parameters. To let a function modify a caller's object, pass a pointer and document whether the pointer may be null and how long it must remain valid. Returning a value is often simpler when the function computes one result.

```c
int add(int left, int right);

int add(int left, int right)
{
    return left + right;
}
```

The declaration and definition must agree. If values can approach the limits of int, check for overflow before adding them because signed overflow is undefined behavior.

## Headers and include boundaries

A header usually contains declarations, type definitions, and macros needed by multiple source files. A source file contains the corresponding function definitions. Include the headers that declare the functions and types a file uses instead of relying on another header to include them indirectly.

A header guard prevents the same header from being processed multiple times in one translation unit:

```c
#ifndef READING_MATH_H
#define READING_MATH_H

int add(int left, int right);

#endif
```

Keep implementation details out of public headers when users do not need them. Avoid defining a non-inline function in a header included by multiple source files, because that can create multiple external definitions during linking.

## Linkage and storage duration

A function declared at file scope has external linkage by default. A file-scope function declared static has internal linkage and is visible only within that translation unit. File-scope objects follow related linkage rules.

The static keyword also has a different use for a block-scope object: it gives that object static storage duration, so it remains alive for the entire program. The name is still limited to its block scope. Do not confuse this with file-scope static linkage.

An extern declaration refers to an entity defined elsewhere. A program should have one compatible external definition for an ordinary function or object. Headers normally provide declarations, while one source file owns the definition.

## Separate source files

A small module can expose a public header and keep a helper private to its source file:

```c
#include "reading_math.h"

static int clamp_nonnegative(int value)
{
    return value < 0 ? 0 : value;
}

int add_nonnegative(int left, int right)
{
    return clamp_nonnegative(left) + clamp_nonnegative(right);
}
```

The header for this file should declare add_nonnegative. The helper has internal linkage because it is file-scope static. A caller should not depend on it.

Compile and link the implementation together with the caller:

```sh
cc -std=c17 -Wall -Wextra -Wpedantic -c reading_math.c -o reading_math.o
cc -std=c17 -Wall -Wextra -Wpedantic -c main.c -o main.o
cc main.o reading_math.o -o app
```

A linker error about an undefined reference often means a required object file or library was not included, or the declaration does not match the actual definition.

## Function design

Keep a function focused on one operation and make failure behavior explicit. A function that can fail can return a status code and write its result through a pointer, or use another documented convention. Do not return a pointer to an automatic local object because its lifetime ends when the function returns.

Document ownership for pointers. State whether the caller owns memory, whether the callee borrows it, and whether a returned pointer remains valid after later calls. C does not enforce these conventions, so clear interfaces are important.

## Key points

- A prototype declares a complete function interface before a call.
- C passes arguments by value. Use pointers only when an interface needs access to caller-owned objects.
- Header guards prevent repeated inclusion. Headers generally declare, while source files define.
- File-scope static gives internal linkage. Block-scope static gives static storage duration.
- A returned pointer must refer to storage that remains alive after the function returns.

## Practice

1. Split a small program into a public header, an implementation file, and a caller file.
2. Add a private helper with file-scope static and verify that another file cannot call it.
3. Explain the difference between a function prototype, definition, and call.
4. Design an error return convention for a function that parses an integer into an output parameter.

## References

- [C working group](https://www.open-std.org/jtc1/sc22/wg14/)
- [GCC warning options](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html)
- [GNU linker documentation](https://sourceware.org/binutils/docs/ld/)
