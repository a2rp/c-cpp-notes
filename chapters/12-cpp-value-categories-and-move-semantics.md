[Back to notes index](../README.md)

| [Previous: C++ classes, constructors, and RAII](11-cpp-classes-constructors-and-raii.md) | [Notes index](../README.md) | [Next: C++ templates and generic programming](13-cpp-templates-and-generic-programming.md) |
| --- | --- | --- |
# 12. C++ value categories and move semantics

Move semantics let a type transfer resources from an object that is no longer needed, avoiding some expensive copies. The feature is part of the type and library model. It should express ownership transfer, not be applied mechanically to every variable.

## Value categories

A simplified starting point is:

- An lvalue identifies an object with a persistent identity, such as a named variable.
- A prvalue computes a value that can initialize a destination, such as a temporary result.
- An xvalue identifies an object whose resources may be reused, such as an object cast with std::move.

These categories affect which overloads can be selected. The full C++ value-category model is more detailed, but recognizing named objects and temporary values is enough for many everyday decisions.

## What std::move does

std::move does not move anything by itself. It casts an expression so that a move-aware overload can be selected. The selected constructor or assignment operator performs the transfer.

After an object is moved from, it remains valid but its value is generally unspecified unless the type documents more. Assign a new value before relying on its contents. Do not assume a moved-from string is empty just because a particular implementation often makes it so.

```cpp
#include <string>
#include <utility>
#include <vector>

int main()
{
    std::string title{"Resource ownership"};
    std::vector<std::string> titles;
    titles.push_back(std::move(title));

    title = "A new value";
    return titles.empty() ? 1 : 0;
}
```

The vector can take the string's resources through its move constructor. The source string is then assigned a known new value before it is used again.

## Move constructors and assignment

A move constructor initializes a new object from an object that can give up resources. A move assignment operator replaces an existing object's state. Standard library members such as std::string, std::vector, and std::unique_ptr already support appropriate move operations.

A class that follows the rule of zero usually gets correct move behavior through its members. A custom resource-owning class must define copy and move behavior carefully. The moved-from object must remain destructible and assignable, and the destination must become the clear owner of the transferred resource.

## Parameter design

Passing a small value type by value is usually clear. A read-only large object can be passed by const reference. A function that stores a value may accept by value and move into its member, allowing the caller to copy an lvalue or move an rvalue.

Do not add std::move to a local return value without understanding the return rules. Returning a local object by name allows copy elision and can be more efficient than forcing a move. Do not move from a const object expecting a normal move, because moving commonly requires modifying the source.

## Forwarding references

A T&& parameter is a forwarding reference when T is deduced in the appropriate template context. std::forward preserves whether the caller supplied an lvalue or an rvalue. This is useful in generic wrappers and constructors. It is not the same as an ordinary rvalue reference to a fixed type.

Use forwarding only when an interface must preserve the caller's value category. For ordinary application code, clear overloads and value parameters are often easier to read.

## Key points

- std::move is a cast that enables move overload selection; it is not the operation itself.
- A moved-from object remains valid but usually has an unspecified value.
- Standard library containers and strings already implement move operations.
- Return local objects by name and let copy elision work unless a measured need says otherwise.
- Use std::forward only in generic code that must preserve the caller's value category.

## Practice

1. Move a string into a vector and then assign a new value to the original string before using it.
2. Explain why std::move on a const object may still select a copy operation.
3. Compare passing a large object by const reference with passing by value and moving into a member.
4. Find an unnecessary std::move on a local return and remove it.

## References

- [C++ working draft: value categories](https://eel.is/c++draft/basic.lval)
- [C++ Core Guidelines: move operations](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-move)
- [C++ standard library utility](https://en.cppreference.com/w/cpp/utility/move)
