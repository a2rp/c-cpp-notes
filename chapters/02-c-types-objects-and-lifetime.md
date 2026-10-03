[Back to notes index](../README.md)

| [Previous: Toolchain, compilation, and the first program](01-toolchain-compilation-and-first-program.md) | [Notes index](../README.md) | [Next: Expressions and control flow in C](03-expressions-and-control-flow-in-c.md) |
| --- | --- | --- |
# 2. C types, objects, and lifetime

C is a typed language. A type describes the values an object can represent and the operations that are valid for it. A reliable program makes object lifetime and initialization explicit instead of relying on whatever bits happen to be in memory.

## Integer and floating types

C provides integer types such as char, short, int, long, and long long, with signed and unsigned forms. The standard specifies minimum ranges, but their exact sizes can differ across platforms. Use a type for its semantic role rather than assuming that int is always 32 bits.

The header stdint.h provides optional fixed-width types such as int32_t when the implementation supports an exact width. Types such as size_t and ptrdiff_t are intended for sizes and pointer differences. The limits.h header exposes ranges such as INT_MAX and UINT_MAX.

Floating-point types include float, double, and long double. Their precision and representation are implementation details. Decimal fractions such as 0.1 may not have an exact binary representation, so avoid comparing floating-point calculations as if they were exact decimal arithmetic.

## Objects and initialization

An object is a region of storage with a type and a lifetime. A declared variable is an object. An expression can designate an object's value, and an object can be modified only through rules permitted by its type and storage.

An automatic local variable without an initializer has an indeterminate value. Reading it before assigning a valid value can produce undefined behavior. Objects with static storage duration are initialized to zero when no explicit initializer is provided. Initialize local objects at their declaration when practical.

Storage duration describes how long storage exists:

- Automatic storage duration usually lasts until the block exits.
- Static storage duration lasts for the entire program execution.
- Allocated storage duration lasts from successful allocation until it is released.
- Thread storage duration lasts for a thread's execution when thread-local storage is used.

Lifetime, scope, and linkage are related but distinct. Scope describes where a name can be used in source code. Linkage describes whether declarations in different scopes or translation units refer to the same entity. Lifetime describes how long an object's storage remains valid.

## Signed and unsigned arithmetic

Unsigned integer arithmetic is computed modulo one more than the maximum representable value. Signed overflow is undefined behavior. It is not a reliable way to test whether a value wrapped.

Converting an out-of-range value to an unsigned type produces a value modulo the type's range. Converting an out-of-range value to a signed type is not a portable way to clamp or wrap data. Validate ranges before converting values received from users, files, or network input.

Integer promotions can convert smaller integer types to int or unsigned int before an operation. Usual arithmetic conversions then determine the common type. This can make mixed signed and unsigned expressions surprising. Keep operands consistent and check compiler warnings.

## Inspect a value's type size

sizeof returns a size_t value measured in C bytes. A C byte is the size of a char and is not required to contain exactly eight bits on every implementation. Use CHAR_BIT from limits.h when the number of bits matters.

```c
#include <inttypes.h>
#include <stddef.h>
#include <stdint.h>
#include <stdio.h>

int main(void)
{
    int score = 42;
    size_t score_size = sizeof score;
    int32_t record_id = 123;

    printf("score=%d size=%zu id=%" PRId32, score, score_size, record_id);
    putchar('\n');
    return 0;
}
```

The exact-width typedef int32_t exists only when the implementation provides an integer type with exactly 32 bits. The portable format macro PRId32 comes from inttypes.h.

## Boolean and enumerated values

C's _Bool stores zero or one. The stdbool.h header provides bool, true, and false spellings. Use an enum to name a fixed set of states instead of scattering unexplained numeric values through a program.

A value from an external input should still be checked before being treated as one of the expected enum cases. A variable can hold a value that does not match a named enumerator, so a switch should include a default path where invalid input is possible.

## Constants and mutability

A const-qualified object cannot be modified through that particular qualified lvalue. This does not make every object reachable from a pointer deeply immutable. C's const also has different rules from C++ compile-time constant expressions, so examples should not assume the languages behave identically.

Use named constants for values that have meaning. For an array length, use sizeof on the actual array while it is still an array, or pass the length explicitly to a function. An array parameter is adjusted to a pointer parameter, so sizeof inside that function measures the pointer, not the original array.

## Key points

- Do not assume integer widths from a type name. Use limits and standard width types when required.
- Initialize automatic objects before reading them.
- Scope, linkage, and lifetime describe different properties.
- Unsigned wrap is defined modulo its range. Signed overflow is undefined behavior.
- Use size_t for object sizes and the correct format specifier when printing it.

## Practice

1. Print sizeof for char, short, int, long, long long, float, and double on your compiler. Compare the result without assuming it is universal.
2. Explain why reading an uninitialized local integer is not a portable way to get a random number.
3. Write a function that receives an int and rejects values outside a valid range before converting them to an unsigned type.
4. Find a mixed signed and unsigned expression and explain which type the comparison uses.

## References

- [C working group](https://www.open-std.org/jtc1/sc22/wg14/)
- [GCC integer implementation details](https://gcc.gnu.org/onlinedocs/gcc/Integers-implementation.html)
- [GCC warning options](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html)
