[Back to notes index](../README.md)

| [Previous: C++ containers, iterators, and algorithms](14-cpp-containers-iterators-and-algorithms.md) | [Notes index](../README.md) | [Next: Concurrency, threads, and atomics](16-concurrency-threads-and-atomics.md) |
| --- | --- | --- |
# 15. C++ errors, exceptions, and value results

Error handling should make failure visible and leave objects in a valid state. C++ supports exceptions and value-based results. Choose a convention that matches the layer and use it consistently across the interface.

## Exceptions

An exception reports that an operation could not complete normally. Throw an exception when a function cannot meet its contract and the caller can handle or propagate the failure. Catch exceptions at a boundary where the program can add context, recover, or report a useful error.

RAII releases resources while stack unwinding leaves scopes. This is one reason resource-owning objects should manage cleanup in their destructors. Destructors should not throw. Avoid catching every exception and continuing as if the operation succeeded.

Use standard exception types where they match the condition. For example, invalid_argument can report an invalid caller value, while runtime_error can describe a runtime failure. Do not expose secrets or internal implementation details in messages sent to users.

```cpp
#include <stdexcept>
#include <string>

int positive_length(const std::string& text)
{
    if (text.empty()) {
        throw std::invalid_argument("text must not be empty");
    }
    return static_cast<int>(text.size());
}

int main()
{
    try {
        return positive_length("notes") > 0 ? 0 : 1;
    } catch (const std::invalid_argument& error) {
        (void)error;
        return 1;
    }
}
```

For a general string, converting size_t to int may narrow. The sample uses a small constant. Production code should return size_t or check that the value fits before converting.

## Optional values

std::optional<T> represents either a value or no value. It is useful when absence is an expected result, such as a lookup that may not find an item. It does not explain why the value is absent. Use a richer result type when callers need to distinguish failure reasons.

```cpp
#include <charconv>
#include <optional>
#include <string_view>
#include <system_error>

std::optional<int> parse_integer(std::string_view text)
{
    if (text.empty()) {
        return std::nullopt;
    }

    int value{};
    const char* first = text.data();
    const char* last = first + text.size();
    const auto result = std::from_chars(first, last, value);

    if (result.ec != std::errc{} || result.ptr != last) {
        return std::nullopt;
    }
    return value;
}
```

The parser rejects empty text, invalid characters, and values outside the range of int. from_chars does not throw for parse failure. A caller can check whether the optional contains a value before using it.

## Value-based error choices

An enum status plus an output parameter can be useful in low-level APIs or code that avoids exceptions. std::variant can represent one of several known result alternatives. Keep the success and failure cases explicit and ensure callers handle every alternative.

C++23 adds std::expected for a value or an error. The notes use C++20 as the baseline, so check compiler and standard library support before using that facility.

## Exception safety

The no-throw guarantee means an operation does not fail by exception. The basic guarantee means invariants remain valid and resources do not leak after failure. The strong guarantee means failure has no externally visible effect. Choose a guarantee that callers can rely on and avoid promising more than the implementation can provide.

A function should validate inputs before changing important state where practical. For a multi-step update, build new state separately and commit it only after the operation succeeds. Standard containers often provide documented exception guarantees for their operations.

## Key points

- Exceptions communicate failures that callers can handle or propagate.
- RAII releases resources during stack unwinding. Destructors should not throw.
- std::optional expresses presence or absence, not a detailed error reason.
- Use a richer result representation when callers need to distinguish failures.
- Document the exception guarantee and preserve invariants when an operation fails.

## Practice

1. Choose between an exception, optional value, and status result for a configuration lookup and explain the caller behavior.
2. Modify the parser to return a distinct error for invalid syntax and out-of-range values.
3. Explain why catching every exception and returning a default value can hide a broken operation.
4. Identify how RAII changes cleanup when an exception leaves a function.

## References

- [C++ working draft: exceptions](https://eel.is/c++draft/except)
- [C++ working draft: optional](https://eel.is/c++draft/optional)
- [C++ Core Guidelines: error handling](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-errors)
- [C++ standard library from_chars](https://en.cppreference.com/w/cpp/utility/from_chars)
