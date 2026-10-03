[Back to notes index](../README.md)

| [Previous: C++ value categories and move semantics](12-cpp-value-categories-and-move-semantics.md) | [Notes index](../README.md) | [Next: C++ containers, iterators, and algorithms](14-cpp-containers-iterators-and-algorithms.md) |
| --- | --- | --- |
# 13. C++ templates and generic programming

Templates let a function or type work with several types while preserving compile-time checking. Generic code is most useful when it expresses one clear operation for a family of types. It becomes difficult when constraints and ownership expectations are hidden.

## Function templates

A function template describes a family of functions. The compiler deduces template arguments from a call when possible and instantiates a concrete function for the selected types.

```cpp
#include <string>

template <typename T>
T larger_value(const T& left, const T& right)
{
    return left < right ? right : left;
}

int main()
{
    int larger_number = larger_value(4, 9);
    std::string later_word = larger_value(std::string{"alpha"}, std::string{"beta"});
    return larger_number == 9 && later_word == "beta" ? 0 : 1;
}
```

The example expects both arguments to have the same type and that the type supports comparison and copying. A general interface should make those requirements explicit.

## Class templates

A class template describes a type family, such as a box that holds a value of a chosen type. Standard containers are class templates. Template definitions are generally visible where they are instantiated, which is why they are commonly defined in headers.

```cpp
template <typename T>
class Box {
public:
    explicit Box(T value) : value_{std::move(value)} {}

    const T& value() const noexcept { return value_; }

private:
    T value_;
};
```

This example needs the utility header for std::move. It stores a value and returns a non-owning reference valid only while the Box object remains alive.

## Concepts and constraints

C++20 concepts name requirements on template arguments. Constraints make generic interfaces easier to understand and can produce more focused diagnostics than a failure deep inside a function body.

```cpp
#include <concepts>
#include <iostream>

template <typename T>
concept StreamWritable = requires(std::ostream& output, const T& value) {
    { output << value } -> std::same_as<std::ostream&>;
};

template <StreamWritable T>
void print_value(const T& value)
{
    std::cout << value;
}

int main()
{
    print_value(42);
    print_value("ready");
}
```

The constraint says the expression must be valid and return the required stream reference type. A concept describes syntax and types, but it cannot by itself prove the operation has the desired runtime meaning.

## Requires expressions and overload selection

A requires expression checks whether an expression or type requirement is valid. A constrained overload can be selected only when its requirements are satisfied. Keep constraints close to the template declaration and avoid constraints that depend on obscure implementation details.

Template specialization can customize behavior, but overuse makes code harder to navigate. Prefer overloads, small helper functions, and standard library algorithms when they make the behavior clearer.

## Template errors and build boundaries

Template code is compiled when a concrete type is instantiated. A function template declaration alone may not be enough for another translation unit to instantiate it; the definition normally needs to be visible in the header. This differs from an ordinary function whose definition can be compiled once into an object file.

Compiler diagnostics can be long because they include nested instantiations. Read the first error in the instantiation chain and identify which requirement failed. Concepts can shorten this path by stating the required operations directly.

## Key points

- Templates define families of functions and types that are checked for concrete arguments.
- Type deduction works from call expressions, but it does not remove the need for clear constraints.
- C++20 concepts name requirements and improve generic interfaces.
- Template definitions usually need to be visible where they are instantiated.
- Use generic code when several types share one meaningful operation, not only to avoid writing a small function.

## Practice

1. Add a type requirement to a template and make the diagnostic explain the operation it needs.
2. Write a class template that stores a value and returns it by const reference.
3. Explain why template definitions commonly live in headers.
4. Compare a generic helper with two simple overloads and choose the easier interface to review.

## References

- [C++ working draft: templates](https://eel.is/c++draft/temp)
- [C++ working draft: concepts](https://eel.is/c++draft/concepts)
- [C++ Core Guidelines: templates](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-templates)
- [C++ standard library concepts](https://en.cppreference.com/w/cpp/concepts)
