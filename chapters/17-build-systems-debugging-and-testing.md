[Back to notes index](../README.md)

| [Previous: Concurrency, threads, and atomics](16-concurrency-threads-and-atomics.md) | [Notes index](../README.md) | [Next: All code samples](98-all-code-samples.md) |
| --- | --- | --- |
# 17. Build systems, debugging, and testing

A build system records source files, compiler settings, dependencies, and test commands. A repeatable build makes it easier to review changes and reproduce failures on another machine.

## CMake targets

CMake describes targets and their relationships. A target can be an executable, library, or test. Prefer target-scoped settings so language standards, include directories, and warnings apply only where intended.

```cmake
cmake_minimum_required(VERSION 3.20)
project(language_notes LANGUAGES C CXX)

enable_testing()

add_executable(c_example main.c)
set_target_properties(c_example PROPERTIES C_STANDARD 17 C_STANDARD_REQUIRED YES C_EXTENSIONS NO)

add_executable(cpp_example main.cpp)
target_compile_features(cpp_example PRIVATE cxx_std_20)

if(MSVC)
    target_compile_options(c_example PRIVATE /W4)
    target_compile_options(cpp_example PRIVATE /W4)
else()
    target_compile_options(c_example PRIVATE -Wall -Wextra -Wpedantic)
    target_compile_options(cpp_example PRIVATE -Wall -Wextra -Wpedantic)
endif()

add_test(NAME c_example_runs COMMAND c_example)
add_test(NAME cpp_example_runs COMMAND cpp_example)
```

CMake selects a generator for the available build tool. Keep generated build output outside the source files, for example in a build directory, and exclude it from version control.

```sh
cmake -S . -B build
cmake --build build
ctest --test-dir build --output-on-failure
```

The test command runs the executables registered with CTest. Each example should return a nonzero status when its check fails so the test runner can report failure.

## C and C++ interfaces

C++ has different name linkage from C. A C header included from C++ can wrap declarations in an extern "C" block so the linker uses C-compatible names.

```c
#ifndef READING_MATH_H
#define READING_MATH_H

#ifdef __cplusplus
extern "C" {
#endif

int add_nonnegative(int left, int right, int *result);

#ifdef __cplusplus
}
#endif

#endif
```

The preprocessor leaves the extern block out when compiling the header as C. The implementation must use compatible declarations, types, and calling conventions.

## Unit tests and assertions

A unit test checks one small behavior with controlled inputs. Keep tests repeatable and independent of external services where possible. Test normal values, boundary values, invalid input, and failure paths.

The C assert macro is useful for internal invariants during development. Assertions may be disabled when NDEBUG is defined, so do not use assert for required input validation or operations that must run in production.

Test output should make failures actionable. Include the expected and actual result when useful, and return a nonzero exit status when a test fails so build systems can detect it.

## Debugging workflow

Build with debug information and reproduce the failure with the smallest input that shows it. Set a breakpoint near the first incorrect state, inspect variables, step through calls, and examine the stack trace. A debugger shows one execution; it does not prove all branches are correct.

Read compiler warnings and sanitizer reports from the first relevant diagnostic. Address the defect rather than hiding the warning or suppressing the sanitizer. Use a separate build configuration for address and undefined-behavior sanitizers, and another for thread sanitizer when supported.

## Testing and portability

Build with more than one compiler when portability matters. Platform differences can expose assumptions about type width, path syntax, alignment, and library support. Test every configured language standard and conditional compilation branch.

Keep third-party dependencies explicit and versioned. Do not rely on headers or libraries that happen to be installed globally but are not declared in the build configuration.

## Key points

- Build files should state source files, language standards, warnings, and dependencies.
- Target-scoped CMake settings keep configuration clear.
- Tests should cover normal cases, boundaries, invalid inputs, and failure paths.
- Assertions are for internal development checks, not required runtime validation.
- Debuggers and sanitizers are evidence tools. Continue to test branches and platforms.

## Practice

1. Create the sample CMake project and build both the C17 and C++20 targets.
2. Run both registered CTest entries and make one test fail with a nonzero status.
3. Compile with debug symbols and set a breakpoint where a value first becomes incorrect.
4. Build separate address/undefined behavior and thread sanitizer configurations where your compiler supports them.

## References

- [CMake documentation](https://cmake.org/documentation/)
- [CMake build system manual](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html)
- [GCC warning options](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html)
- [GCC instrumentation options](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html)
- [Clang AddressSanitizer](https://clang.llvm.org/docs/AddressSanitizer.html)
- [Clang UndefinedBehaviorSanitizer](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html)
