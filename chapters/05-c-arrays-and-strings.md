[Back to notes index](../README.md)

| [Previous: C functions, headers, and linkage](04-c-functions-headers-and-linkage.md) | [Notes index](../README.md) | [Next: C pointers and dynamic memory](06-c-pointers-and-dynamic-memory.md) |
| --- | --- | --- |
# 5. C arrays and strings

An array stores a fixed number of elements of one type in contiguous storage. C does not check array bounds at runtime. The programmer must preserve the array length and ensure every index is within the valid range.

## Array length and parameter decay

For an array object in the scope where it is declared, sizeof array divided by sizeof array[0] gives the element count. This works only while the expression is an actual array. When an array is passed to a function, the parameter is adjusted to a pointer, so sizeof inside the function reports the pointer size.

Pass the element count explicitly alongside a pointer. For an empty range, define whether a null pointer is permitted and ensure the function does not dereference it when the count is zero.

```c
#include <stddef.h>
#include <stdio.h>

int main(void)
{
    int values[] = {4, 6, 8};
    size_t count = sizeof values / sizeof values[0];
    int total = 0;

    for (size_t index = 0; index < count; ++index) {
        total += values[index];
    }

    printf("count=%zu total=%d", count, total);
    return 0;
}
```

The loop condition uses index less than count. An index equal to count is one past the last element and must not be dereferenced.

## Strings are character arrays with a terminator

A C string is a sequence of char values ending with a null character. The storage capacity must include space for that terminator. A character array is not automatically a string; code using string functions must ensure the array contains a terminator within its bounds.

The string length and the buffer capacity are different values. A buffer of 16 bytes can hold at most 15 ordinary characters plus the terminating null character. strlen reads until it finds a terminator, so calling it on a non-terminated array can read beyond the object.

Prefer functions that receive a destination capacity. snprintf writes at most the provided size and reports how many characters would have been written, excluding the terminator. Check its result to detect formatting errors or truncation.

```c
#include <stdio.h>

int main(void)
{
    char label[32];
    int user_id = 42;
    int written = snprintf(label, sizeof label, "user-%d", user_id);

    if (written < 0 || (size_t)written >= sizeof label) {
        fputs("label could not be formatted", stderr);
        return 1;
    }

    puts(label);
    return 0;
}
```

The cast in the comparison converts the nonnegative successful length to size_t. A production function should include the correct headers and preserve the same checked pattern.

## Reading text

fgets reads at most one less than the supplied buffer size and appends a terminator when it succeeds. It may leave the rest of a long input line in the stream. Check whether the newline was read when the full line is required; reject or drain an incomplete line instead of treating a truncated prefix as complete input.

Do not use gets. It has no capacity argument and cannot safely prevent a buffer overflow. Also avoid assuming that strncpy always produces a terminated string. Choose an operation with an explicit size and make the termination behavior clear.

## Copying and overlapping ranges

memcpy copies a specified number of bytes but does not support overlapping source and destination ranges. memmove supports overlap. Both functions require valid pointers for the requested number of bytes. Check multiplication when calculating a byte count such as element_count times sizeof element, because the multiplication itself can overflow size_t.

For text, use string functions only when the input is a valid null-terminated string. For arbitrary binary data, carry an explicit byte length and use memory operations instead.

## Key points

- C arrays have fixed bounds, and ordinary indexing is not checked at runtime.
- An array parameter is adjusted to a pointer. Pass the length separately.
- A C string requires a null terminator within the array capacity.
- Use bounded formatting and check whether output was truncated.
- Choose memcpy only when ranges do not overlap; use memmove when overlap is possible.

## Practice

1. Write a function that receives a pointer and element count, then processes an array without indexing past its end.
2. Explain why sizeof on an array parameter does not provide the caller's element count.
3. Format a name into a fixed buffer and handle both formatting failure and truncation.
4. Read a line with fgets and design behavior for input longer than the buffer.

## References

- [C working group](https://www.open-std.org/jtc1/sc22/wg14/)
- [GNU C Library string and array functions](https://www.gnu.org/software/libc/manual/html_node/String-and-Array-Utilities.html)
- [GCC warning options](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html)
