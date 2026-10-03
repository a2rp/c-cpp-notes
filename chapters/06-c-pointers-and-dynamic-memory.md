[Back to notes index](../README.md)

| [Previous: C arrays and strings](05-c-arrays-and-strings.md) | [Notes index](../README.md) | [Next: C structures, unions, and enumerations](07-c-structures-unions-and-enumerations.md) |
| --- | --- | --- |
# 6. C pointers and dynamic memory

A pointer object stores an address or a null pointer value. Pointers let functions refer to other objects and let programs work with allocated storage. Every pointer use depends on the lifetime, type, and bounds of the object it designates.

## Pointer basics

The address-of operator produces a pointer to an object. The indirection operator accesses the pointed-to object. A null pointer does not designate an object and must not be dereferenced.

Pointer arithmetic is defined only within an array object and to the one-past position. A one-past pointer can be used for comparison or iteration boundaries, but it cannot be dereferenced. Do not subtract or compare unrelated pointers as if they belonged to one array.

A pointer can become dangling when the referenced object reaches the end of its lifetime. Returning the address of an automatic local variable creates a dangling pointer. After releasing allocated memory, clear or stop using every pointer that referred to it.

## Const-qualified pointers

The location of const changes what may be modified:

- const int *ptr means the pointed-to integer cannot be modified through ptr.
- int *const ptr means ptr cannot be changed to point elsewhere after initialization.
- const int *const ptr means neither the pointer nor the pointed-to value can be changed through ptr.

Const qualification is an interface promise for access through that expression. It does not guarantee that no other alias can modify the same object.

## Allocate and release memory

malloc requests a number of bytes and returns a pointer to suitably aligned storage, or a null pointer if allocation fails. The returned storage is not initialized. calloc allocates an array and initializes its bytes to zero. free releases allocated storage. Reading an uninitialized allocated value is invalid, and using it after free is a use-after-free defect.

Calculate allocation sizes with overflow checks. This example allocates a small array and releases it through one cleanup path:

```c
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    size_t count = 4;
    if (count > SIZE_MAX / sizeof(int)) {
        return 1;
    }

    int *values = malloc(count * sizeof *values);
    if (values == NULL) {
        fputs("allocation failed", stderr);
        putchar('\n');
        return 1;
    }

    for (size_t index = 0; index < count; ++index) {
        values[index] = (int)index;
    }

    printf("last=%d", values[count - 1]);
    free(values);
    return 0;
}
```

This fixed count is small enough to fit in int. A general function must validate the count before converting each index to a narrower signed type.

## Realloc without losing the original allocation

realloc can move an allocation. Store its result in a temporary pointer so a failure does not lose the original pointer. For a nonzero new size, a null result indicates failure and the original allocation remains valid.

```c
int *resized = realloc(values, new_count * sizeof *values);
if (resized == NULL) {
    /* values is still valid here */
    free(values);
    return 1;
}
values = resized;
```

The example assumes new_count has already been checked for multiplication overflow and is not zero. Handle a request for zero elements explicitly rather than relying on implementation-specific realloc behavior.

## Ownership and cleanup

C does not track ownership. An interface should state who allocates memory, who releases it, and whether a pointer is borrowed or transferred. A borrowed pointer must not be freed by the borrower and must not be used beyond the owner's lifetime.

When a function acquires several resources, route failures through cleanup code that releases every acquired resource exactly once. A single cleanup section can make this visible. Avoid both leaks and double frees. free(NULL) is safe, but it does not make an already freed non-null pointer safe to reuse.

## Common pointer defects

- Dereferencing a null or uninitialized pointer.
- Accessing beyond the bounds of the pointed-to object.
- Returning a pointer to a local automatic object.
- Using a pointer after its object has been freed.
- Freeing memory twice or freeing memory not returned by an allocation function.
- Computing a byte count that overflows before allocation.
- Keeping aliases after ownership has been released.

Use compiler warnings, address sanitizers, and tests to help find defects. These tools cannot prove that every pointer path is valid.

## Key points

- A pointer is valid only while its target object is alive and the access is within bounds.
- Pointer arithmetic is limited to one array object and its one-past boundary.
- Check allocation results and byte-count multiplication.
- Use a temporary for realloc and define how zero-size requests behave.
- Document ownership, borrowing, and cleanup in C interfaces.

## Practice

1. Draw the lifetime of a local object and an allocated object, then mark which pointers remain valid after a function returns.
2. Add an allocation failure path and prove that every successful allocation is released once.
3. Explain why realloc should write into a temporary pointer.
4. Build a small example with address and undefined-behavior sanitizers and investigate any reported invalid access.

## References

- [C working group](https://www.open-std.org/jtc1/sc22/wg14/)
- [GNU C Library memory allocation](https://www.gnu.org/software/libc/manual/html_node/Memory-Allocation.html)
- [GCC instrumentation options](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html)
