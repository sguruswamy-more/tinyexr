# TinyEXR CMake Integration Guide

This document explains how to use the modern CMake build system for TinyEXR.

## Building and Installing

### Build Options

- `TINYEXR_BUILD_SAMPLE` (default: ON) - Build the test_tinyexr sample executable
- `TINYEXR_USE_MINIZ` (default: ON) - Use bundled miniz for compression
- `TINYEXR_INSTALL` (default: ON) - Generate installation targets

### Build Commands

```bash
# Configure
cmake -B build -DCMAKE_BUILD_TYPE=Release

# Build
cmake --build build

# Install (optional, requires TINYEXR_INSTALL=ON)
cmake --install build --prefix /path/to/install
```

## Using TinyEXR in Your Project

### Method 1: Using find_package (Installed)

After installing TinyEXR, use it in your CMake project:

```cmake
cmake_minimum_required(VERSION 3.14)
project(MyProject)

# Find the installed tinyexr package
find_package(tinyexr REQUIRED)

# Link against tinyexr
add_executable(myapp main.cpp)
target_link_libraries(myapp PRIVATE tinyexr::tinyexr)
```

Configure your project with:
```bash
cmake -B build -DCMAKE_PREFIX_PATH=/path/to/tinyexr/install
```

### Method 2: Using add_subdirectory (Embedded)

Add TinyEXR as a subdirectory in your project:

```cmake
cmake_minimum_required(VERSION 3.14)
project(MyProject)

# Add tinyexr subdirectory (disable sample builds)
set(TINYEXR_BUILD_SAMPLE OFF CACHE BOOL "" FORCE)
add_subdirectory(external/tinyexr)

# Link against tinyexr
add_executable(myapp main.cpp)
target_link_libraries(myapp PRIVATE tinyexr::tinyexr)
```

### Method 3: Using FetchContent

Download TinyEXR automatically during configure:

```cmake
cmake_minimum_required(VERSION 3.14)
project(MyProject)

include(FetchContent)

FetchContent_Declare(
  tinyexr
  GIT_REPOSITORY https://github.com/syoyo/tinyexr.git
  GIT_TAG        master  # or a specific tag/commit
)

set(TINYEXR_BUILD_SAMPLE OFF CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(tinyexr)

# Link against tinyexr
add_executable(myapp main.cpp)
target_link_libraries(myapp PRIVATE tinyexr::tinyexr)
```

## Usage in Code

```cpp
#include <tinyexr.h>

int main() {
    EXRVersion exr_version;
    int ret = ParseEXRVersionFromFile(&exr_version, "image.exr");
    
    if (ret == TINYEXR_SUCCESS) {
        // Process EXR file...
    }
    
    return 0;
}
```

## Key Features of Modern CMake Build

- ✅ Proper namespaced targets (`tinyexr::tinyexr`, `tinyexr::miniz`)
- ✅ Full `find_package()` support with version checking
- ✅ Generator expressions for include directories
- ✅ Correct transitive dependency handling
- ✅ Installation targets with proper export
- ✅ Config-file package support
- ✅ Compatible with CMake 3.14+
- ✅ Position-independent code (PIC) enabled
- ✅ Modern target-based approach (no global variables)
