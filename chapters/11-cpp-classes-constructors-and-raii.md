[Back to notes index](../README.md)

| [Previous: C++ types, references, const, and initialization](10-cpp-types-references-const-and-initialization.md) | [Notes index](../README.md) | [Next: C++ value categories and move semantics](12-cpp-value-categories-and-move-semantics.md) |
| --- | --- | --- |
# 11. C++ classes, constructors, and RAII

A class combines data with operations that preserve its invariants. Resource Acquisition Is Initialization (RAII) ties resource ownership to object lifetime: acquire a resource during construction and release it during destruction. This gives cleanup a clear owner and works with normal returns and exceptions.

## Classes and invariants

A class should make invalid states difficult to represent. Keep implementation details private and expose operations that preserve the object's rules. A constructor establishes the initial invariant. Public methods should keep it valid after every successful call.

Initialize data members in the member initializer list. Members are initialized in the order they are declared in the class, not the order written in the initializer list. Order declarations to make dependencies clear and enable compiler warnings about accidental ordering mistakes.

Use explicit on a single-argument constructor when the conversion should not happen implicitly. Use const member functions for operations that inspect an object without changing its observable state.

```cpp
#include <memory>
#include <string>
#include <utility>

class Reading {
public:
    explicit Reading(std::string title)
        : title_{std::move(title)}
    {
    }

    const std::string& title() const noexcept
    {
        return title_;
    }

private:
    std::string title_;
};

int main()
{
    auto reading = std::make_unique<Reading>("C++ ownership");
    return reading->title().empty() ? 1 : 0;
}
```

std::string owns its character storage, and unique_ptr owns the Reading object. Both release their resources automatically when their objects leave scope.

## RAII for resources

File streams, locks, and smart pointers are standard examples of RAII. Prefer a standard library owner over manual acquire and release calls. If a resource needs a custom owner, encapsulate it in a type whose destructor releases it exactly once.

A destructor should not allow an exception to escape. Resource cleanup must be reliable during stack unwinding. If cleanup can fail in a way the program must report, expose an explicit operation that can return an error before destruction.

## Ownership pointers

Use std::unique_ptr when one object owns a dynamically allocated object. Use std::shared_ptr only when shared ownership is part of the design and several components must extend the same lifetime. Use std::weak_ptr to observe a shared object without extending its lifetime and to break ownership cycles.

Raw pointers and references are commonly non-owning. Their interfaces should make it clear that they do not delete the object and how long the object remains valid. Avoid manual new and delete in application code when make_unique, containers, or value types can express ownership.

## Copy and move behavior

A class that owns a raw resource must define how copying behaves. Copying a pointer value is not a deep copy. It can create double deletion or aliasing bugs. Prefer the rule of zero: compose standard library members that already manage resources, and let their copy and move operations define the class behavior.

When a class directly manages a resource and needs custom copy, move, or destruction, define the operations deliberately. A unique_ptr member makes a type move-only by default. That can be the correct ownership model.

## Exception safety

RAII keeps already-acquired resources valid when an exception leaves a scope. Build objects so their constructors either establish a valid object or fail without leaking resources. Prefer operations that preserve invariants if they fail. Standard containers and smart pointers simplify these guarantees.

## Key points

- Constructors establish class invariants and member initializer lists initialize members.
- RAII attaches resource release to object lifetime.
- Prefer value types and standard owners over manual new and delete.
- Use unique_ptr for one owner. Use shared_ptr only when shared lifetime is required.
- The rule of zero avoids hand-written ownership operations when members already manage resources.

## Practice

1. Add a validation rule to the Reading constructor and decide whether invalid input should throw or use a factory that returns an error.
2. Replace a raw owning pointer with unique_ptr and remove the manual delete.
3. Explain why a class with a raw pointer member cannot safely use a default shallow copy.
4. Identify which members in a class own resources and which are non-owning views.

## References

- [C++ Core Guidelines: resource management](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-resource)
- [C++ working draft](https://eel.is/c++draft/)
- [C++ standard library unique_ptr](https://en.cppreference.com/w/cpp/memory/unique_ptr)
- [C++ standard library shared_ptr](https://en.cppreference.com/w/cpp/memory/shared_ptr)
