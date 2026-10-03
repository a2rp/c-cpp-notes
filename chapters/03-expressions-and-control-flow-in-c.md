[Back to notes index](../README.md)

| [Previous: C types, objects, and lifetime](02-c-types-objects-and-lifetime.md) | [Notes index](../README.md) | [Next: C functions, headers, and linkage](04-c-functions-headers-and-linkage.md) |
| --- | --- | --- |
# 3. Expressions and control flow in C

Expressions compute values or perform operations. Control-flow statements decide which expressions run and how execution repeats. Clear sequencing and explicit conditions make C code easier to review and less dependent on assumptions about the compiler.

## Operators and conversions

C operators include arithmetic, comparison, logical, bitwise, assignment, and pointer operations. Precedence determines how an expression is grouped, but it does not always determine the order in which operands are evaluated. Use parentheses to show grouping and separate expressions when execution order matters.

The logical operators && and || evaluate left to right and short-circuit. The right operand of && runs only if the left operand is true. The right operand of || runs only if the left operand is false. This is useful for checking a pointer before dereferencing it, provided the expression is otherwise valid.

The conditional operator chooses one of two expressions. The comma operator evaluates its left operand before the right, but it is uncommon in ordinary application code and can make expressions harder to read.

Avoid modifying the same scalar object multiple times in one expression when the evaluations are not sequenced. An expression whose behavior depends on operand evaluation order is difficult to review and can invoke undefined behavior.

## Conditions and loops

An if condition is true when its value compares unequal to zero. Write the intended comparison explicitly when working with numeric values. A switch is useful for a small set of integral or enumeration cases. Use break unless fall-through is intentional and documented.

A for loop fits a known initialization, condition, and update pattern. A while loop fits repetition controlled by a condition. A do-while loop executes its body at least once. Each loop should make its exit condition and progress step easy to identify.

Use continue to skip the remaining body for the current iteration. Use break to leave the nearest loop or switch. Avoid deeply nested control flow by extracting a function or returning early when that improves clarity.

## A small control-flow example

This program totals nonnegative values below a limit. It keeps the loop variable and total in a well-defined range for the small example.

```c
#include <stdio.h>

int main(void)
{
    int limit = 8;
    int total = 0;

    for (int value = 0; value < limit; ++value) {
        if (value == 3) {
            continue;
        }
        total += value;
    }

    if (total > 0) {
        printf("total=%d", total);
        putchar('\n');
    } else {
        fputs("no positive total", stderr);
        putchar('\n');
    }

    return 0;
}
```

The continue skips only the addition for value 3. It still reaches the loop update. The example uses a small fixed limit; user-controlled limits need input validation and overflow checks.

## Switch and state handling

A switch groups behavior for values of an integral type or an enumeration. Each case label must be a constant expression. If one case should continue into another, document the fall-through because it is easy to mistake for a missing break.

```c
switch (state) {
case STATE_READY:
    start_work();
    break;
case STATE_STOPPED:
    stop_work();
    break;
default:
    report_invalid_state(state);
    break;
}
```

The example assumes that state and the functions are declared elsewhere. A default branch is useful when a value can come from a file, network packet, or cast rather than a validated enum value.

## Integer conditions and edge cases

A comparison can trigger integer conversions before it produces a result. Mixing signed and unsigned values can cause a negative signed value to convert to a large unsigned value. Validate values and use compatible types before comparisons.

Check loop boundaries carefully. For an array with length n, valid indexes are from zero through n minus one. An empty array has no valid index. Do not use a sentinel value unless it is impossible for that value to be valid data.

## Key points

- Precedence controls grouping. It does not always specify operand evaluation order.
- Logical operators short-circuit and can guard a later operation.
- Avoid multiple unsequenced modifications to the same scalar in one expression.
- Make loop progress and exit conditions visible.
- Validate input and check integer ranges before using them as indexes or sizes.

## Practice

1. Rewrite a long condition with parentheses and named intermediate variables.
2. Create a switch over an enum and add a default path for an unexpected value.
3. Trace the loop example by hand and explain which value is skipped.
4. Find a mixed signed and unsigned comparison and explain how conversions affect it.

## References

- [C working group](https://www.open-std.org/jtc1/sc22/wg14/)
- [GCC warning options](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html)
- [Clang diagnostics reference](https://clang.llvm.org/docs/DiagnosticsReference.html)
