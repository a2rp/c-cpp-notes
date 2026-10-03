[Back to notes index](../README.md)

| [Previous: All code samples](98-all-code-samples.md) | [Notes index](../README.md) | Next: End |
| --- | --- | --- |

# 99. Complete questions and answers

These questions review the core C and C++ ideas in the notes. Use them to explain what a program guarantees, where a lifetime ends, and why an interface is designed a particular way.

## 1. Toolchain, compilation, and the first program

**1. What are the main stages between source code and an executable?**

The preprocessor handles directives, the compiler translates source, the assembler creates object code, and the linker combines objects and libraries into an executable.

**2. How does a compile error differ from a link error?**

A compile error means a source file could not be translated. A link error commonly means a referenced definition or library was not included in the final link.

**3. Why select the language standard explicitly?**

It makes the language features and compiler mode repeatable across builds and helps reveal accidental reliance on compiler extensions.

**4. What do compiler warnings help find?**

They identify suspicious or nonportable code before runtime. Each warning should be understood and fixed or deliberately justified.

**5. What do sanitizers prove?**

They can detect certain defects on executed paths. Passing a sanitizer run does not prove that all paths are correct or safe.

## 2. C types, objects, and lifetime

**6. Why should code avoid assuming that int is always 32 bits?**

The language specifies minimum ranges, while exact widths can vary by implementation. Use a fixed-width type when an exact width is required and available.

**7. What can happen when an uninitialized automatic variable is read?**

It has an indeterminate value, and reading it can produce undefined behavior. Initialize it before use.

**8. How does scope differ from lifetime?**

Scope describes where a name can be used in source code. Lifetime describes how long the object's storage remains valid.

**9. What is the difference between signed overflow and unsigned wrap?**

Unsigned arithmetic is computed modulo its range. Signed overflow is undefined behavior and must not be used as a wraparound mechanism.

**10. What does sizeof return?**

It returns a size_t count of C bytes occupied by the operand. A C byte is not required to be exactly eight bits on every implementation.

## 3. Expressions and control flow in C

**11. Does operator precedence determine evaluation order?**

No. Precedence determines grouping. Operand evaluation order is governed by separate language sequencing rules.

**12. When do logical AND and OR short-circuit?**

The right side of AND is evaluated only if the left side is true. The right side of OR is evaluated only if the left side is false.

**13. Why avoid modifying one scalar multiple times in an expression?**

If the modifications are unsequenced, the behavior can be undefined or depend on an invalid assumption about evaluation order.

**14. What makes a loop boundary safe for an array?**

For a count n, valid indexes are from zero through n minus one. The loop should stop when the index equals the count.

**15. Why include a default branch in a switch over external data?**

Input may contain a value that is not one of the named enumerators. The default path can reject or report that state.

## 4. C functions, headers, and linkage

**16. Why should a C function with no parameters declare void?**

It gives the compiler a complete prototype that says the function accepts no arguments.

**17. What belongs in a header and what usually belongs in a source file?**

A header publishes declarations and shared type definitions. A source file usually owns the corresponding function definitions.

**18. What does file-scope static do to a function name?**

It gives the function internal linkage, so the name is visible only within that translation unit.

**19. Why is copying a pointer not the same as copying the pointed-to object?**

The pointer value is copied, so both pointers can refer to the same storage. Ownership and lifetime remain separate concerns.

**20. Why must a returned pointer not refer to an automatic local object?**

The local object's lifetime ends when the function returns, leaving the returned pointer dangling.

## 5. C arrays and strings

**21. How is an array element count computed while the object is still an array?**

Divide sizeof the array by sizeof one element. This does not work after the array has decayed to a pointer parameter.

**22. Why should a function receive an array length with its pointer?**

A pointer does not carry the original array bound. The explicit count lets the function validate indexes and iteration limits.

**23. What makes a character array a C string?**

It contains a null terminator within its bounds after the final character.

**24. What should code check after snprintf?**

A negative result indicates an error. A result at least as large as the buffer capacity means the output was truncated.

**25. When should memmove be used instead of memcpy?**

Use memmove when source and destination byte ranges may overlap. memcpy requires non-overlapping ranges.

## 6. C pointers and dynamic memory

**26. Where is pointer arithmetic defined?**

Within one array object and up to its one-past position. The one-past pointer cannot be dereferenced.

**27. What should code check before dereferencing a pointer?**

It should establish that the pointer is non-null, refers to a live object, and designates an element within the object's bounds.

**28. Why check multiplication before allocating an array?**

The byte-count multiplication can overflow size_t, causing an allocation smaller than the requested element count requires.

**29. Why store realloc output in a temporary pointer?**

If a nonzero resize fails, the original allocation remains valid. Assigning directly could lose the only pointer to it.

**30. What should an interface say about a pointer parameter?**

It should explain whether the pointer is borrowed or owned, whether null is allowed, which bounds apply, and how long the target remains valid.

## 7. C structures, unions, and enumerations

**31. Why can sizeof a structure exceed the sum of its member sizes?**

The implementation can insert padding between members or at the end to satisfy alignment requirements.

**32. What does struct assignment copy?**

It copies each member value. If a member is a pointer, the address is copied, not the pointed-to object.

**33. Why should an enum not be used as a portable file encoding without conversion?**

The enum's underlying representation is not fixed by the language for every implementation.

**34. How should code track a union's active value?**

Keep an explicit tag or another documented rule that records which member currently contains the meaningful value.

**35. Why not write an entire struct directly to a portable file?**

Padding, byte order, member representation, and layout can vary. Encode each field with a defined width and byte order instead.

## 8. C files, streams, and errors

**36. Why does fgetc return int instead of char?**

The int result can represent every unsigned-char value and the distinct EOF sentinel.

**37. How does code distinguish stream error from normal end of file?**

After the read loop, inspect ferror. EOF alone does not distinguish the two conditions.

**38. Why check fclose after writing?**

Buffered output may be flushed during close, and that operation can fail. The caller may need to report unsuccessful persistence.

**39. Why is writing raw struct bytes not a portable binary format?**

The struct may contain padding and implementation-specific representations. A portable format needs explicit field encoding.

**40. What should happen after a successful fopen on every later path?**

The stream should be closed exactly once, including paths where a later read or write fails.

## 9. C preprocessing and macros

**41. When does the C preprocessor run?**

It processes directives and macro expansions before the compiler analyzes the resulting C translation unit.

**42. What is a header guard for?**

It prevents the same header contents from being included multiple times in one translation unit.

**43. Why can a function-like macro evaluate an input more than once?**

The preprocessor substitutes tokens without creating a typed function call. If the parameter appears repeatedly, its expression can appear repeatedly after expansion.

**44. What does conditional compilation do?**

It includes or excludes source regions based on preprocessor definitions and conditions before compilation.

**45. Why prefer a function over a macro for a normal operation?**

A function provides type checking, ordinary scope, and predictable argument evaluation.

## 10. C++ types, references, const, and initialization

**46. What does brace initialization help prevent?**

It rejects many narrowing conversions where a value could lose information.

**47. How does a reference differ from a pointer?**

A reference must bind to an object and cannot be reseated. A pointer can be null or changed to refer to another object.

**48. What does const guarantee?**

It prevents mutation through a particular qualified access path. It does not necessarily make an entire object graph immutable.

**49. What is a non-owning view?**

It provides access to existing storage without owning that storage. The source must outlive all uses of the view.

**50. Why use nullptr instead of zero for a null pointer?**

nullptr has a dedicated null pointer type and avoids some overload ambiguities caused by using an integer value.

## 11. C++ classes, constructors, and RAII

**51. What is a class invariant?**

It is a property that should remain true for every valid object before and after its public operations.

**52. What does RAII connect?**

It connects resource ownership to object lifetime so a destructor releases the resource when the owner leaves scope.

**53. When is unique_ptr appropriate?**

When one object owns a dynamically allocated resource and should release it automatically.

**54. When is shared_ptr appropriate?**

When shared ownership is a real part of the design and multiple owners must extend the same object's lifetime.

**55. What is the rule of zero?**

Prefer composing members that already manage resources so the class does not need hand-written copy, move, and destruction operations.

## 12. C++ value categories and move semantics

**56. What does std::move do?**

It casts an expression to enable move-aware overload selection. The selected constructor or assignment performs the transfer.

**57. What can code assume about a moved-from object?**

It remains valid, but its value is generally unspecified unless its type documents more. Reassign it before relying on its contents.

**58. Why can a move constructor be more efficient than a copy?**

It can transfer owned resources instead of allocating and duplicating their contents.

**59. Why avoid std::move on a local return value without evidence?**

Returning the local by name can enable copy elision. Forcing a move may prevent that optimization.

**60. When is std::forward useful?**

In generic code that needs to preserve whether the caller supplied an lvalue or an rvalue.

## 13. C++ templates and generic programming

**61. What does a function template describe?**

It describes a family of functions that the compiler instantiates for concrete types.

**62. What are concepts used for?**

They name requirements on template arguments and can improve interface clarity and compile diagnostics.

**63. Why are template definitions commonly in headers?**

The definition usually needs to be visible at the point where the compiler instantiates the template for a concrete type.

**64. Do concepts prove a type has the intended runtime meaning?**

No. They can check required syntax and types, but the author must define and document semantic expectations.

**65. When is generic code useful?**

When several types share one meaningful operation and a single checked interface is clearer than repeated implementations.

## 14. C++ containers, iterators, and algorithms

**66. Why is vector a common default sequence?**

It stores elements contiguously, supports indexed access, and provides efficient append in the amortized case.

**67. What can invalidate vector iterators and references?**

Reallocation invalidates all element references and iterators. Erasing invalidates positions at and after the erased element.

**68. What does lower_bound require?**

The range must be sorted according to the same ordering used by the search.

**69. What does a C++ range view own?**

Usually it does not own the underlying elements. The source must stay alive while the view is used.

**70. Why consider complexity when choosing a container?**

Lookup, insertion, and removal can have different growth costs. The chosen structure should match the operations the program performs.

## 15. C++ errors, exceptions, and value results

**71. When are exceptions useful?**

When an operation cannot meet its contract and the caller can handle or propagate the failure at an appropriate boundary.

**72. What does optional represent?**

It represents a value that may be absent. It does not explain a detailed reason for absence.

**73. Why should destructors not throw?**

A second exception during stack unwinding can terminate the program and prevent reliable cleanup.

**74. What does the strong exception guarantee mean?**

If an operation fails, it has no externally visible effect on the program state.

**75. What is the benefit of from_chars for parsing?**

It reports parse status without throwing and can parse a bounded character range.

## 16. Concurrency, threads, and atomics

**76. What is a data race?**

It is conflicting unsynchronized access to the same memory, with at least one write. In C++, a data race is undefined behavior.

**77. Why use an RAII lock guard?**

It releases the mutex automatically when its scope exits, including when an exception occurs.

**78. Does an atomic counter make several related fields transactional?**

No. An atomic protects its own operations. Related state may need a mutex or another synchronization design.

**79. Why use a predicate with a condition-variable wait?**

The predicate is checked again after wakeup and handles spurious wakeups while the condition is protected by the lock.

**80. Why avoid detached threads in a small application?**

Their lifetime and shutdown behavior become difficult to coordinate with the objects and resources they use.

## 17. Build systems, debugging, and testing

**81. What does a build system record?**

It records source files, compiler settings, dependencies, targets, and test commands needed to build the project repeatably.

**82. What does CTest run?**

It runs tests registered with the CMake project and reports their success or failure.

**83. Why should assert not be used for required input validation?**

Assertions can be disabled in builds that define NDEBUG, so they cannot enforce required runtime behavior.

**84. How does a debugger help find a defect?**

Breakpoints and stepping show where values change and provide call-stack context for one execution path.

**85. Why build with more than one compiler?**

Different implementations can expose nonportable assumptions and differences in compiler or library support.
