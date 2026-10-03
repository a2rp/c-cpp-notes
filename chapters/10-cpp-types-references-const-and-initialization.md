[Back to notes index](../README.md)

| [Previous: C preprocessing and macros](09-c-preprocessing-and-macros.md) | [Notes index](../README.md) | [Next: C++ classes, constructors, and RAII](11-cpp-classes-constructors-and-raii.md) |
| --- | --- | --- |
# 10. C++ types, references, const, and initialization

C++ builds on C-like syntax with its own type and object model. Value semantics, references, const qualification, and initialization rules help make interfaces clear and catch mistakes during compilation.

## Values, objects, and types

A variable has a type that controls which values it can represent and which operations are valid. Fundamental types include integers, floating-point values, and bool. Library types such as std::string manage more behavior than a raw character array.

Use a type that expresses the data. Use std::size_t or a container's size_type for sizes and indexes when appropriate. Do not convert a size to a signed type without checking that it is representable.

## Initialization and narrowing

C++ supports direct initialization, copy initialization, and list initialization. Braces reject many narrowing conversions, which makes them useful when converting a value could lose information.

```cpp
#include <iostream>
#include <string>
#include <string_view>

void print_length(std::string_view text)
{
    std::cout << text.size();
}

int main()
{
    std::string name{"Ashish"};
    const int retry_limit{3};
    auto name_length = name.size();

    print_length(name);
    std::cout << " retries=" << retry_limit
              << " length=" << name_length;
}
```

std::string owns its characters. std::string_view is a non-owning view into character storage. The function above uses the view only during the call, while name remains alive. Do not store a view beyond the lifetime of its source.

## References and pointers

A reference is an alias to an existing object and must be initialized when declared. A reference cannot later be reseated to refer to another object. A pointer can be null and can be changed to refer to another object, so its validity must be checked before dereferencing.

Use a reference for a required object that the function will not rebind. Use a pointer when absence is meaningful or when pointer arithmetic and ownership conventions are part of the interface. Prefer smart pointers for owned dynamic objects, covered in the next chapter.

A const reference can inspect an object without copying it or modifying it through that reference. Passing a small scalar by value is often simpler than a reference. Measure before avoiding copies of small values.

## Const and auto

Const restricts mutation through a particular access path. In const int *ptr, the pointed-to integer cannot be changed through ptr. In int *const ptr, the pointer itself cannot be reassigned after initialization. Const does not make an object deeply immutable if it contains pointers to mutable objects.

The auto keyword asks the compiler to deduce a type from an initializer. Plain auto usually drops top-level references and const qualification. Use auto& or const auto& when the reference or const behavior is part of the intent. Keep explicit types where they make an important conversion or interface easier to see.

Use nullptr for a null pointer value. It has a dedicated type and avoids some overload ambiguities that can occur with the integer literal zero.

## Non-owning views

std::string_view and std::span provide lightweight views over existing storage. They do not own that storage. A view is valid only while the underlying object remains alive and its elements remain at stable addresses.

A function can accept std::span<const T> to inspect a contiguous sequence without copying it. If the function stores the view, document the lifetime requirement or store an owning container instead.

## Key points

- Prefer standard library value types such as std::string over manual buffers when they fit the need.
- Brace initialization catches many narrowing conversions.
- A reference must bind to an object. A pointer can be null and needs a validity rule.
- Const limits mutation through an access path, but does not guarantee deep immutability.
- Views do not own memory. The source object must outlive every use of the view.

## Practice

1. Change the sample to pass a const std::string reference and compare its interface with std::string_view.
2. Try to initialize an int with a fractional value using braces and explain the diagnostic.
3. Give an example where a string_view would become invalid after its source string is destroyed.
4. Explain when a pointer is more appropriate than a reference.

## References

- [C++ working draft](https://eel.is/c++draft/)
- [C++ Core Guidelines: interfaces](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-functions)
- [C++ standard library string view](https://en.cppreference.com/w/cpp/string/basic_string_view)
- [C++ standard library span](https://en.cppreference.com/w/cpp/container/span)
