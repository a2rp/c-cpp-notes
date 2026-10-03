[Back to notes index](../README.md)

| [Previous: C++ templates and generic programming](13-cpp-templates-and-generic-programming.md) | [Notes index](../README.md) | [Next: C++ errors, exceptions, and value results](15-cpp-errors-exceptions-and-value-results.md) |
| --- | --- | --- |
# 14. C++ containers, iterators, and algorithms

The standard library provides containers for common data structures and algorithms that operate on iterator ranges. Prefer these facilities over custom loops and memory management when they express the requirement clearly.

## Choose a container from its operations

- std::array stores a fixed number of elements in the object and knows its size.
- std::vector is a contiguous dynamic sequence and is a strong default for ordered collections.
- std::string owns a sequence of characters.
- std::map stores ordered key-value pairs.
- std::unordered_map stores key-value pairs with hash-based average lookup.
- std::set and std::unordered_set store unique values with ordered or hash-based lookup.

Consider how the program inserts, searches, iterates, and removes values. std::list is not automatically faster for insertion; pointer-heavy structures can have poor memory locality. Measure with representative data before choosing an unusual container.

## Iterators and invalidation

Iterators describe positions in a container or range. Invalidation rules vary by container and operation. Reallocation of a vector invalidates pointers, references, and iterators to its elements. Erasing an element invalidates iterators at and after the erased position in a vector.

Do not keep an iterator across an operation that may invalidate it. Read the container's invalidation guarantees when storing iterators, references, or pointers. Reacquire them after a mutation when needed.

## Algorithms

Standard algorithms name common operations such as sorting, searching, counting, and transforming. An algorithm takes an iterator range or range object and often accepts a predicate. This separates what the operation does from how the data is stored.

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> scores{18, 7, 12, 7, 20};
    std::ranges::sort(scores);

    const auto first_passing = std::ranges::lower_bound(scores, 12);
    if (first_passing != scores.end()) {
        std::cout << *first_passing;
    }
}
```

The vector is sorted in ascending order. lower_bound requires a range sorted according to the same ordering and returns the first position that is not less than the requested value.

## Ranges and views

C++20 ranges let algorithms accept whole ranges and provide composable views. A view can represent a transformed or filtered sequence without immediately copying every element. Views generally refer to underlying data, so that data must remain alive while the view is used.

Use a view when it makes a transformation clear. Avoid retaining a view past the lifetime of its source. A range pipeline still needs tests that confirm the order of filtering, transformation, and iteration.

## Complexity and measurement

Container choice affects time complexity and memory use. A vector provides constant-time indexed access and amortized constant-time append, but insertion near the front shifts elements. A map provides ordered lookup with logarithmic complexity. An unordered map provides average constant-time lookup, with different worst-case behavior and extra storage.

Big O describes growth, not actual latency. Cache behavior, allocation, data distribution, and workload size also matter. Choose the simplest container that meets the real operations and measure before adding complexity.

## Key points

- Use std::vector as a default sequence unless the required operations suggest another container.
- Algorithms work with ranges and iterators, keeping data structure and operation separate.
- Iterator, pointer, and reference invalidation rules depend on the container and mutation.
- Sorted-range algorithms require the input to be sorted in the expected order.
- Views do not own their source. Keep the source alive for every use of the view.

## Practice

1. Sort a vector and use lower_bound to find an insertion position.
2. Explain which references become invalid after a vector grows beyond its capacity.
3. Choose a container for ordered unique keys and another for average fast key lookup.
4. Compare a handwritten loop with a standard algorithm and explain which communicates the intent better.

## References

- [C++ working draft: containers](https://eel.is/c++draft/containers)
- [C++ working draft: algorithms](https://eel.is/c++draft/algorithms)
- [C++ standard library containers](https://en.cppreference.com/w/cpp/container)
- [C++ standard library algorithms](https://en.cppreference.com/w/cpp/algorithm)
