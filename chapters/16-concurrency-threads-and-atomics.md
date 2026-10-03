[Back to notes index](../README.md)

| [Previous: C++ errors, exceptions, and value results](15-cpp-errors-exceptions-and-value-results.md) | [Notes index](../README.md) | [Next: Build systems, debugging, and testing](17-build-systems-debugging-and-testing.md) |
| --- | --- | --- |
# 16. Concurrency, threads, and atomics

Concurrency lets work progress on multiple threads, but shared mutable data needs synchronization. A data race occurs when conflicting accesses happen without the required synchronization, and at least one access modifies the object. A data race is undefined behavior in C++.

## Thread lifetime

A std::thread represents an executing thread. The program must join or detach it before the thread object is destroyed. Detaching makes lifetime and error handling harder to reason about. C++20 std::jthread requests cooperative stop when appropriate and joins automatically when destroyed.

Automatic joining helps manage lifetime, but it does not make shared data safe. Every shared object still needs an ownership and synchronization plan. Prefer tasks or higher-level abstractions when they make the work and lifetime easier to understand.

## Protect shared state with a mutex

A mutex provides exclusive access to protected data. Use an RAII lock such as std::lock_guard so the lock is released when the scope exits, including when an exception occurs. Every read and write to the shared state must follow the same synchronization rule.

```cpp
#include <iostream>
#include <mutex>
#include <thread>

int main()
{
    std::mutex count_mutex;
    int count = 0;

    auto increment = [&] {
        for (int index = 0; index < 1000; ++index) {
            std::lock_guard lock{count_mutex};
            ++count;
        }
    };

    {
        std::jthread first{increment};
        std::jthread second{increment};
    }

    std::cout << count;
    return count == 2000 ? 0 : 1;
}
```

The inner scope ends only after both jthread destructors have joined. The shared counter is read after both threads finish. The mutex protects each increment.

## Atomics

std::atomic supports synchronized access to particular scalar values. An atomic counter can be simpler than a mutex when the state is only a counter or flag. Atomics do not make a group of related fields update as one transaction.

The default atomic operations use sequentially consistent ordering. This is often a good starting point. Weaker memory orders can improve performance in specific designs, but require a precise proof of which writes become visible to which reads. Do not replace a mutex with an atomic based only on intuition.

## Condition variables

A condition variable lets a thread wait until shared state satisfies a condition. Protect the state with a mutex and wait with a predicate. The predicate form handles spurious wakeups and checks the condition again after the thread resumes.

```cpp
std::unique_lock lock{mutex};
condition.wait(lock, [&] { return ready; });
process_ready_item();
```

This fragment assumes mutex, condition, ready, and process_ready_item are declared in a design where ready is protected by mutex. Keep the wait predicate and updates to ready under the same lock.

## Deadlocks and lock scope

Deadlock can occur when threads acquire locks in inconsistent orders or wait on each other while holding resources. Use a consistent lock order. std::scoped_lock can acquire multiple mutexes without the common lock-order deadlock pattern.

Keep critical sections short. Do not call unknown callbacks or perform slow network and file operations while holding a mutex unless the design specifically requires it. Document which lock protects each shared object.

## Testing concurrent code

Stress tests can expose scheduling-dependent defects, but passing tests do not prove race freedom. Use ThreadSanitizer where supported, review shared-state access, and keep synchronization close to the data it protects. Avoid detached background work when the application has no clear shutdown protocol.

## Key points

- A data race is undefined behavior. Shared state requires synchronization.
- Join threads or use a scoped thread owner such as std::jthread.
- Use RAII locks and ensure every access follows the same protection rule.
- Atomics protect individual operations but do not make multiple fields transactional.
- Condition-variable waits should use a predicate while holding a unique_lock.

## Practice

1. Remove the mutex from the counter and explain why the resulting program has a data race.
2. Add a condition variable to a producer and consumer example and protect its queue with one mutex.
3. Draw a lock ordering for two mutexes and show how a different order could deadlock.
4. Run a concurrent test with ThreadSanitizer and investigate any report before changing the code.

## References

- [C++ working draft: threads](https://eel.is/c++draft/thread)
- [C++ Core Guidelines: concurrency](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#S-concurrency)
- [GCC instrumentation options](https://gcc.gnu.org/onlinedocs/gcc/Instrumentation-Options.html)
- [Clang ThreadSanitizer](https://clang.llvm.org/docs/ThreadSanitizer.html)
