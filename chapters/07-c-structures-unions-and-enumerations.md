[Back to notes index](../README.md)

| [Previous: C pointers and dynamic memory](06-c-pointers-and-dynamic-memory.md) | [Notes index](../README.md) | [Next: C files, streams, and errors](08-c-files-streams-and-errors.md) |
| --- | --- | --- |
# 7. C structures, unions, and enumerations

Structures, unions, and enumerations let a program represent related data and named states. They describe values in the running program. Their memory layout should not be treated as a portable file or network format without an explicit encoding.

## Structures

A struct groups named members into one object. The implementation may insert padding between members or at the end to satisfy alignment. Therefore sizeof a struct can be larger than the sum of its member sizes.

Initialize structures explicitly. C99 and later support designated initializers, which name the member being initialized. A struct can be copied by assignment, which copies each member value. If a struct contains pointers, the pointer values are copied, not the objects they point to.

Pass a pointer to a large structure when copying it would be wasteful. Use a pointer to const when the function should inspect but not modify the object through that pointer.

```c
#include <stdio.h>

struct Point {
    double x;
    double y;
};

static void print_point(const struct Point *point)
{
    printf("(%.1f, %.1f)", point->x, point->y);
}

int main(void)
{
    struct Point origin = {.x = 0.0, .y = 0.0};
    print_point(&origin);
    return 0;
}
```

The arrow operator accesses a member through a pointer. The function borrows the Point and does not retain its address.

## Enumerations

An enum assigns names to a set of integer values. It makes control flow and function arguments easier to understand than unexplained numeric constants. C does not require a fixed underlying integer type for an enum, so avoid relying on a particular representation in a file format or external protocol.

Values received from outside the program should be checked before use. A value may not match any named enumerator. A default case can report or reject unknown values.

## Unions and tagged alternatives

A union stores its members in overlapping storage. The program should track which member currently contains the meaningful value. A common pattern pairs a union with an enum tag and updates the tag whenever the active member changes.

```c
#include <stdio.h>

enum ValueKind {
    VALUE_INTEGER,
    VALUE_DECIMAL
};

struct Value {
    enum ValueKind kind;
    union {
        int integer;
        double decimal;
    } data;
};

static void print_value(const struct Value *value)
{
    switch (value->kind) {
    case VALUE_INTEGER:
        printf("%d", value->data.integer);
        break;
    case VALUE_DECIMAL:
        printf("%.2f", value->data.decimal);
        break;
    default:
        fputs("invalid value kind", stderr);
        break;
    }
}

int main(void)
{
    struct Value value = {
        .kind = VALUE_INTEGER,
        .data.integer = 12
    };
    print_value(&value);
    return 0;
}
```

This example must keep the tag and active union member consistent. A union is not a general permission to reinterpret arbitrary bytes as another type.

## Padding, alignment, and external data

The layout of a structure can depend on the compiler, target ABI, and member order. Writing sizeof(struct) bytes directly to a file can include padding and host-specific byte order. Reading that representation on a different platform can fail or expose uninitialized padding bytes.

For portable files and network protocols, encode each field explicitly with a documented width, byte order, and validation rule. Do not assume that a C struct layout is a stable wire format.

Bit-fields can pack flags, but allocation order, storage unit, and alignment are implementation-defined details. Use bit masks on fixed-width unsigned integers when a specified bit representation is required, and validate shift counts to stay below the type width.

## Key points

- Structures group fields, but implementations may add padding for alignment.
- Struct assignment copies member values, including pointer values, not pointed-to storage.
- Enums improve readability but should not be assumed to have a portable external representation.
- A union needs a reliable tag or another explicit rule for its active member.
- Encode persistent and network data field by field instead of dumping a struct's bytes.

## Practice

1. Define a struct for a point and pass it to a read-only function using a pointer to const.
2. Add a new state to an enum and find every switch that needs to handle it.
3. Create a tagged union for an integer or text value and maintain the tag during updates.
4. Explain why writing sizeof(struct) bytes to a file is not a portable serialization format.

## References

- [C working group](https://www.open-std.org/jtc1/sc22/wg14/)
- [GCC structure layout documentation](https://gcc.gnu.org/onlinedocs/gcc/Structures-unions-enumerations-implementation.html)
- [GCC alignment options](https://gcc.gnu.org/onlinedocs/gcc/Common-Type-Attributes.html)
