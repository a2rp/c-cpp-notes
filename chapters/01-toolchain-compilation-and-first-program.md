[Back to notes index](../README.md)

| [Previous: Notes index](../README.md) | [Notes index](../README.md) | [Next: C types, objects, and lifetime](02-c-types-objects-and-lifetime.md) |
| --- | --- | --- |
# 1. Toolchain, compilation, and the first program

C and C++ are compiled languages. A compiler translates source into object code, and a linker combines object files and libraries into an executable. Understanding these stages makes build errors easier to locate.

## The build pipeline

A typical native build has these stages:

1. The preprocessor expands includes and conditional compilation directives.
2. The compiler checks language rules and translates each source file.
3. The assembler produces object files for the target machine.
4. The linker resolves references between object files and libraries.
5. The operating system loads the executable when it runs.

A syntax error usually appears during compilation. An undefined reference often appears during linking because a declaration was found but no matching definition was linked. A runtime error appears after the executable starts.

C and C++ use different language modes and standard libraries. A C source file should be compiled as C, and a C++ source file should be compiled as C++. Do not rename a file to another extension to try to fix a language error.

## First C program

Save this as main.c. The function signature with void states that main takes no arguments.

```c
#include <stdio.h>

int main(void)
{
    puts("Hello from C");
    return 0;
}
```

Compile with a C17 mode and useful warnings:

```sh
cc -std=c17 -Wall -Wextra -Wpedantic -Wconversion -Wshadow -g main.c -o hello-c
./hello-c
```

The compiler name may be gcc or clang. Check which implementation the build uses and record it for reproducible results.

## First C++ program

Save this as main.cpp. C++ uses its own standard library and type rules.

```cpp
#include <iostream>
#include <string_view>

int main()
{
    constexpr std::string_view message{"Hello from C++"};
    std::cout << message << '\\n';
}
```

Compile using a C++20 mode:

```sh
c++ -std=c++20 -Wall -Wextra -Wpedantic -Wconversion -Wshadow -g main.cpp -o hello-cpp
./hello-cpp
```

C++ permits main to reach the closing brace and return zero. Writing the return explicitly is also valid and can be useful while learning control flow.

## Warnings and debug information

Warnings identify suspicious constructs before they become runtime failures. Enable a consistent warning set in local builds and continuous integration. Do not silence a warning until its cause is understood. Different compilers can produce different diagnostics, so build with the compiler used by the project.

The g option adds debug information so a debugger can map machine instructions back to source lines. Sanitizers can detect some memory and undefined-behavior errors during test runs. They do not prove a program is correct and may not be available in every environment.

```sh
cc -std=c17 -Wall -Wextra -Wpedantic -g -fsanitize=address,undefined main.c -o hello-c-check
c++ -std=c++20 -Wall -Wextra -Wpedantic -g -fsanitize=address,undefined main.cpp -o hello-cpp-check
```

Use a supported GCC or Clang version and check the sanitizer documentation for platform limits.

## Separate compilation

Large projects compile several source files independently and link them together. A header provides declarations to other source files. The matching definition is compiled once and linked into the final program.

```sh
cc -std=c17 -Wall -Wextra -Wpedantic -c main.c -o main.o
cc main.o -o hello-c
```

The first command creates an object file. The second links it into an executable. A build system such as CMake records this relationship and can rebuild only the affected targets.

## Key points

- Compilation and linking are separate stages with different error messages.
- Select the language standard explicitly for repeatable builds.
- Enable warnings and debug symbols during development.
- C and C++ have different compilation modes and library APIs.
- Sanitizers and tests find classes of defects but do not prove correctness.

## Practice

1. Compile the C and C++ examples with GCC or Clang and record the compiler version.
2. Introduce a misspelled function name and compare a compile error with a link error.
3. Add a second source file and compile it separately before linking.
4. Enable warnings and explain one warning produced by a deliberately suspicious example.

## References

- [GCC C dialect options](https://gcc.gnu.org/onlinedocs/gcc/C-Dialect-Options.html)
- [GCC warning options](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html)
- [Clang command line reference](https://clang.llvm.org/docs/ClangCommandLineReference.html)
- [CMake documentation](https://cmake.org/documentation/)
