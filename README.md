<!---
{
  "id": "ba108c71-19c6-4063-b813-fdbbe0b5d775",
  "depends_on": ["a23aa456-d465-4860-b955-676181a511ba"],
  "author": "Stephan Bökelmann",
  "first_used": "2025-12-04",
  "keywords": ["C++", "CMake", "Subdirectories", "Project structure", "External library directory"]
}
--->

# Using a Local C++ Library with Its Own `CMakeLists.txt` via `add_subdirectory` — And Using a Library Located Outside the Project

> In this exercise you will learn how to organize a project with a self-written library in its own folder using its own `CMakeLists.txt`. Furthermore we will extend this by integrating a library that is **not located in the project directory**, and explore how CMake can include external paths using `add_subdirectory()` and directory variables.

## Introduction

Modularization is essential for maintainable and scalable C++ projects. In the previous exercises, you learned to link header-only libraries and compiled libraries within a single project directory. Real-world projects, however, also depend on libraries located outside the main source tree—sometimes shared between multiple projects, sometimes stored in separate repositories.

This exercise therefore includes two tasks:

**Task 1:** Create and use a library located *inside* the project folder, each with its own `CMakeLists.txt` and linked using `add_subdirectory()`.

**Task 2:** Repeat the workflow, but place the library *outside* the project directory, and configure CMake so the main project can still build and link it cleanly.

These two scenarios mirror how professional C++ repositories are structured and demonstrate how flexible CMake is for managing multi-directory builds.

### Further Readings and Other Sources

* Modern CMake: [https://cliutils.gitlab.io/modern-cmake/chapters/projects/subdir.html](https://cliutils.gitlab.io/modern-cmake/chapters/projects/subdir.html)
* CMake `add_subdirectory()`: [https://cmake.org/cmake/help/latest/command/add_subdirectory.html](https://cmake.org/cmake/help/latest/command/add_subdirectory.html)
* ISO C++ Guidelines: [https://isocpp.github.io/CppCoreGuidelines](https://isocpp.github.io/CppCoreGuidelines)
* YouTube: *Modern CMake Tutorial* (CppCon)

---

## Tasks

---

# **Task 1 — Library *inside* the project directory**

### 1. Create the project directory structure

```bash
mkdir cpp_modular
cd cpp_modular
mkdir -p src mathlib
```

### 2. Create the library source files (`mathlib/`)

**`mathlib.h`**

```cpp
#pragma once

namespace mathlib {
    int triple(int x);
}
```

**`mathlib.cpp`**

```cpp
#include "mathlib.h"

namespace mathlib {
    int triple(int x) {
        return 3 * x;
    }
}
```

### 3. Library `CMakeLists.txt`

Inside `mathlib/`:

```cmake
add_library(mathlib
    mathlib.cpp
)
target_include_directories(mathlib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
```

### 4. Main program (`src/main.cpp`)

```cpp
#include <iostream>
#include "../mathlib/mathlib.h"

int main() {
    std::cout << mathlib::triple(5) << std::endl;
    return 0;
}
```

### 5. Root `CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.10)
project(ModularProject LANGUAGES CXX)

add_subdirectory(mathlib)

add_executable(modularapp src/main.cpp)
target_link_libraries(modularapp PRIVATE mathlib)
```

### 6. Build & run

```bash
cmake -B build
cmake --build build
./build/modularapp
```

Expected output:

```
15
```

---

# **Task 2 — Library *outside* the project directory**

In real projects, a library often lives in a shared location (e.g., `~/libs/`, a monorepo root, or a sibling directory).
In this task, you move the library outside the project structure and still integrate it cleanly.

### 1. Create a folder *outside* the project

```bash
mkdir -p ~/sharedlibs/mathlib
```

Copy your library files there:

```bash
cp cpp_modular/mathlib/*.cpp ~/sharedlibs/mathlib/
cp cpp_modular/mathlib/*.h ~/sharedlibs/mathlib/
```

### 2. Create a `CMakeLists.txt` **inside the external library**

In `~/sharedlibs/mathlib/CMakeLists.txt`:

```cmake
add_library(mathlib
    mathlib.cpp
)
target_include_directories(mathlib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
```

### 3. Modify the *project’s* root `CMakeLists.txt`

Edit `cpp_modular/CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.10)
project(ModularProjectExternal LANGUAGES CXX)

# Path to the external library
set(MATHLIB_PATH "$ENV{HOME}/sharedlibs/mathlib")

add_subdirectory(${MATHLIB_PATH} ${CMAKE_BINARY_DIR}/mathlib_build)

add_executable(modularapp src/main.cpp)
target_link_libraries(modularapp PRIVATE mathlib)
```

Explanation:

* The first argument to `add_subdirectory()` is the source directory of the library
* The second argument specifies where its build artifacts should go
* This works **even if the library is not inside the project directory**

### 4. Build & run

```bash
cmake -B build
cmake --build build
./build/modularapp
```

You should again see:

```
15
```

---

## Questions

**1. Why does `add_subdirectory()` require both a source and a binary directory when pointing to an external path?**

<details>
<summary>Click to reveal answer</summary>

Because when using external folders, CMake must know where to store the build artifacts for that library. For internal folders, this is implicit; for external ones, it must be specified manually.

</details>

**2. What would happen if you forget to set `PUBLIC` visibility for the library’s include directory?**

<details>
<summary>Click to reveal answer</summary>

The main program would not be able to include `mathlib.h` because the include path is not propagated to dependent targets.

</details>

**3. Why might a real project place libraries outside the main source tree?**

<details>
<summary>Click to reveal answer</summary>

This allows sharing code across multiple projects, separating concerns, versioning libraries independently, or structuring a monorepo.

</details>

---

## Advice

Learning to integrate libraries both internally and externally is a key step toward mastering real-world CMake workflows. External library folders occur frequently—whether in monorepositories, shared utility libraries, or when separating teams maintain different components. Understanding how to use `add_subdirectory()` with absolute paths and custom binary directories prepares you for more advanced topics like exported CMake packages, dependency managers, and large-scale modular C++ architectures.
