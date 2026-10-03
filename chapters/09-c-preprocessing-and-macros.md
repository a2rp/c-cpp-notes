[Back to notes index](../README.md)

| [Previous: C files, streams, and errors](08-c-files-streams-and-errors.md) | [Notes index](../README.md) | [Next: C++ types, references, const, and initialization](10-cpp-types-references-const-and-initialization.md) |
| --- | --- | --- |
# 9. C preprocessing and macros

The C preprocessor handles directives before compilation. It includes header contents, expands macros, and selects conditional regions. Preprocessing is textual and token-based; it does not understand C types or function argument evaluation.

## Includes and header guards

A quoted include commonly searches the source file's directory before configured include paths. An angle-bracket include usually searches configured system include paths. Exact lookup order is controlled by the compiler and build configuration.

A header guard prevents a header from being included more than once in one translation unit:

```c
#ifndef APP_CONFIG_H
#define APP_CONFIG_H

#define APP_DEFAULT_PORT 8080

#endif
```

Choose a guard name that is unique to the project and header. Some compilers support pragma once, but header guards are part of the portable preprocessor model.

## Object-like and function-like macros

An object-like macro replaces a token sequence. It is useful for conditional compilation or values that must be available to the preprocessor. Parenthesize replacement expressions when they are intended to be used in larger expressions.

A function-like macro can evaluate an argument more than once and does not provide type checking. A macro such as MIN(a, b) can evaluate a or b multiple times, so calls with side effects may behave unexpectedly. Prefer a function when a value-level operation can be expressed as a function.

When a macro expands to multiple statements, a do-while pattern can make it behave syntactically like one statement, but it still has no type system. Keep macros small and document why a function cannot be used.

## Conditional compilation

Conditional directives include or exclude source based on macro definitions. This can select platform-specific implementations or remove diagnostics from a release build. Keep each branch independently compilable and test every supported branch in the build matrix.

```c
#include <stdio.h>

int main(void)
{
#ifdef APP_DIAGNOSTICS
    fputs("diagnostics enabled", stderr);
    putchar('\n');
#endif
    puts("application started");
    return 0;
}
```

Compile with -DAPP_DIAGNOSTICS to include the diagnostic message. Without that definition, the preprocessor removes the block before the compiler sees it.

Use #error when a required configuration is absent or unsupported. Avoid using conditional compilation as a substitute for a clear platform abstraction when ordinary functions can keep the code simpler.

## Predefined macros

The preprocessor provides macros such as __FILE__ and __LINE__ that expand to source location information. These can help form diagnostic messages. Do not use reserved identifiers beginning with double underscores or an underscore followed by an uppercase letter for user-defined names.

Macro expansion can make errors difficult to read. Inspect preprocessor output with compiler options such as -E when a declaration or conditional branch is surprising. The exact options vary by compiler.

## C and C++ note

C++ has the preprocessor too, but many C++ tasks are better expressed with constexpr values, templates, inline functions, and type-safe library facilities. A macro should not replace language features that can express the intent with type checking and normal scope.

## Key points

- Preprocessing occurs before language compilation and does not understand types.
- Header guards prevent repeated inclusion in one translation unit.
- Macro arguments with side effects can be evaluated more than once.
- Conditional branches need separate build coverage.
- Prefer functions and language features when they express the same operation safely.

## Practice

1. Add a unique header guard to a project header and include it from two source files.
2. Compile the conditional example with and without its diagnostic definition.
3. Explain why a MIN macro can produce surprising behavior when an argument increments a variable.
4. Inspect preprocessor output and locate one included declaration.

## References

- [GCC C preprocessor documentation](https://gcc.gnu.org/onlinedocs/cpp/)
- [GCC preprocessor options](https://gcc.gnu.org/onlinedocs/gcc/Preprocessor-Options.html)
- [C working group](https://www.open-std.org/jtc1/sc22/wg14/)
