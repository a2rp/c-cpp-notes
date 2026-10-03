[Back to notes index](../README.md)

| [Previous: Build systems, debugging, and testing](17-build-systems-debugging-and-testing.md) | [Notes index](../README.md) | [Next: Complete questions and answers](99-complete-q-and-a.md) |
| --- | --- | --- |

# 98. All code samples

This chapter gathers the code samples from the C and C++ study notes. Each section links back to the notes that explain the example.

## 1. Toolchain, compilation, and the first program

[Open chapter](01-toolchain-compilation-and-first-program.md)

### Sample 1

```c
#include <stdio.h>

int main(void)
{
    puts("Hello from C");
    return 0;
}
```

### Sample 2

```sh
cc -std=c17 -Wall -Wextra -Wpedantic -Wconversion -Wshadow -g main.c -o hello-c
./hello-c
```

### Sample 3

```cpp
#include <iostream>
#include <string_view>

int main()
{
    constexpr std::string_view message{"Hello from C++"};
    std::cout << message << '\n';
}
```

### Sample 4

```sh
c++ -std=c++20 -Wall -Wextra -Wpedantic -Wconversion -Wshadow -g main.cpp -o hello-cpp
./hello-cpp
```

### Sample 5

```sh
cc -std=c17 -Wall -Wextra -Wpedantic -g -fsanitize=address,undefined main.c -o hello-c-check
c++ -std=c++20 -Wall -Wextra -Wpedantic -g -fsanitize=address,undefined main.cpp -o hello-cpp-check
```

### Sample 6

```sh
cc -std=c17 -Wall -Wextra -Wpedantic -c main.c -o main.o
cc main.o -o hello-c
```

## 2. C types, objects, and lifetime

[Open chapter](02-c-types-objects-and-lifetime.md)

### Sample 1

```c
#include <inttypes.h>
#include <stddef.h>
#include <stdint.h>
#include <stdio.h>

int main(void)
{
    int score = 42;
    size_t score_size = sizeof score;
    int32_t record_id = 123;

    printf("score=%d size=%zu id=%" PRId32, score, score_size, record_id);
    putchar('\n');
    return 0;
}
```

## 3. Expressions and control flow in C

[Open chapter](03-expressions-and-control-flow-in-c.md)

### Sample 1

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

### Sample 2

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

## 4. C functions, headers, and linkage

[Open chapter](04-c-functions-headers-and-linkage.md)

### Sample 1

```c
int add(int left, int right);

int add(int left, int right)
{
    return left + right;
}
```

### Sample 2

```c
#ifndef READING_MATH_H
#define READING_MATH_H

int add(int left, int right);

#endif
```

### Sample 3

```c
#include <limits.h>
#include <stddef.h>
#include "reading_math.h"

static int is_nonnegative(int value)
{
    return value >= 0;
}

int add_nonnegative(int left, int right, int *result)
{
    if (result == NULL || !is_nonnegative(left) || !is_nonnegative(right)) {
        return 0;
    }
    if (left > INT_MAX - right) {
        return 0;
    }

    *result = left + right;
    return 1;
}
```

### Sample 4

```sh
cc -std=c17 -Wall -Wextra -Wpedantic -c reading_math.c -o reading_math.o
cc -std=c17 -Wall -Wextra -Wpedantic -c main.c -o main.o
cc main.o reading_math.o -o app
```

## 5. C arrays and strings

[Open chapter](05-c-arrays-and-strings.md)

### Sample 1

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

### Sample 2

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

## 6. C pointers and dynamic memory

[Open chapter](06-c-pointers-and-dynamic-memory.md)

### Sample 1

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

### Sample 2

```c
int *resized = realloc(values, new_count * sizeof *values);
if (resized == NULL) {
    /* values is still valid here */
    free(values);
    return 1;
}
values = resized;
```

## 7. C structures, unions, and enumerations

[Open chapter](07-c-structures-unions-and-enumerations.md)

### Sample 1

```c
#include <stdio.h>

struct Point {
    double x;
    double y;
};

static void print_point(const struct Point *point)
{
    printf("(%.1f, %.1f)", point->x, point->y);
}

int main(void)
{
    struct Point origin = {.x = 0.0, .y = 0.0};
    print_point(&origin);
    return 0;
}
```

### Sample 2

```c
#include <stdio.h>

enum ValueKind {
    VALUE_INTEGER,
    VALUE_DECIMAL
};

struct Value {
    enum ValueKind kind;
    union {
        int integer;
        double decimal;
    } data;
};

static void print_value(const struct Value *value)
{
    switch (value->kind) {
    case VALUE_INTEGER:
        printf("%d", value->data.integer);
        break;
    case VALUE_DECIMAL:
        printf("%.2f", value->data.decimal);
        break;
    default:
        fputs("invalid value kind", stderr);
        break;
    }
}

int main(void)
{
    struct Value value = {
        .kind = VALUE_INTEGER,
        .data.integer = 12
    };
    print_value(&value);
    return 0;
}
```

## 8. C files, streams, and errors

[Open chapter](08-c-files-streams-and-errors.md)

### Sample 1

```c
#include <stdio.h>

static int write_status_file(const char *path)
{
    FILE *file = fopen(path, "w");
    if (file == NULL) {
        perror("fopen");
        return 1;
    }

    int failed = fprintf(file, "status=ready") < 0;
    if (fclose(file) != 0) {
        perror("fclose");
        failed = 1;
    }
    return failed;
}

int main(void)
{
    return write_status_file("status.txt");
}
```

### Sample 2

```c
#include <stdio.h>

static int print_file(const char *path)
{
    FILE *file = fopen(path, "r");
    if (file == NULL) {
        perror("fopen");
        return 1;
    }

    int character;
    while ((character = fgetc(file)) != EOF) {
        if (putchar(character) == EOF) {
            perror("putchar");
            fclose(file);
            return 1;
        }
    }

    int failed = ferror(file) != 0;
    if (fclose(file) != 0) {
        perror("fclose");
        failed = 1;
    }
    return failed;
}
```

## 9. C preprocessing and macros

[Open chapter](09-c-preprocessing-and-macros.md)

### Sample 1

```c
#ifndef APP_CONFIG_H
#define APP_CONFIG_H

#define APP_DEFAULT_PORT 8080

#endif
```

### Sample 2

```c
#include <stdio.h>

int main(void)
{
#ifdef APP_DIAGNOSTICS
    fputs("diagnostics enabled", stderr);
    putchar('\n');
#endif
    puts("application started");
    return 0;
}
```

## 10. C++ types, references, const, and initialization

[Open chapter](10-cpp-types-references-const-and-initialization.md)

### Sample 1

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

## 11. C++ classes, constructors, and RAII

[Open chapter](11-cpp-classes-constructors-and-raii.md)

### Sample 1

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

## 12. C++ value categories and move semantics

[Open chapter](12-cpp-value-categories-and-move-semantics.md)

### Sample 1

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

## 13. C++ templates and generic programming

[Open chapter](13-cpp-templates-and-generic-programming.md)

### Sample 1

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

### Sample 2

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

### Sample 3

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

## 14. C++ containers, iterators, and algorithms

[Open chapter](14-cpp-containers-iterators-and-algorithms.md)

### Sample 1

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

## 15. C++ errors, exceptions, and value results

[Open chapter](15-cpp-errors-exceptions-and-value-results.md)

### Sample 1

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

### Sample 2

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

## 16. Concurrency, threads, and atomics

[Open chapter](16-concurrency-threads-and-atomics.md)

### Sample 1

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

### Sample 2

```cpp
std::unique_lock lock{mutex};
condition.wait(lock, [&] { return ready; });
process_ready_item();
```

## 17. Build systems, debugging, and testing

[Open chapter](17-build-systems-debugging-and-testing.md)

### Sample 1

```cmake
cmake_minimum_required(VERSION 3.20)
project(language_notes LANGUAGES C CXX)

enable_testing()

add_executable(c_example main.c)
set_target_properties(c_example PROPERTIES C_STANDARD 17 C_STANDARD_REQUIRED YES C_EXTENSIONS NO)

add_executable(cpp_example main.cpp)
target_compile_features(cpp_example PRIVATE cxx_std_20)

if(MSVC)
    target_compile_options(c_example PRIVATE /W4)
    target_compile_options(cpp_example PRIVATE /W4)
else()
    target_compile_options(c_example PRIVATE -Wall -Wextra -Wpedantic)
    target_compile_options(cpp_example PRIVATE -Wall -Wextra -Wpedantic)
endif()

add_test(NAME c_example_runs COMMAND c_example)
add_test(NAME cpp_example_runs COMMAND cpp_example)
```

### Sample 2

```sh
cmake -S . -B build
cmake --build build
ctest --test-dir build --output-on-failure
```

### Sample 3

```c
#ifndef READING_MATH_H
#define READING_MATH_H

#ifdef __cplusplus
extern "C" {
#endif

int add_nonnegative(int left, int right, int *result);

#ifdef __cplusplus
}
#endif

#endif
```
