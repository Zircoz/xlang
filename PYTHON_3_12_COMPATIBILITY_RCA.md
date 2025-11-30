# WinRT Python 3.12 Compatibility: Root Cause Analysis and Proposal

## 1. Root Cause Analysis

The `pywinrt.exe` tool is incompatible with Python versions greater than 3.9 due to its reliance on deprecated and removed features of the Python C API. The primary issues are:

*   **Use of Deprecated Unicode APIs:** The codebase uses `PyUnicode_AsWideCharString`, which was deprecated in Python 3.10 and completely removed in Python 3.12. This function was used to convert Python Unicode objects to wide character strings, and its removal is a hard blocker for compatibility.

*   **Use of Deprecated Integer APIs:** The codebase uses `PyLong_FromLong` and `PyLong_AsLong` with `int32_t` types. While these functions are still available, their signatures have changed to use `long`, and the internal representation of integers in Python 3.12 has been optimized. This can lead to unexpected behavior and build failures on some platforms.

*   **Outdated Build and Test Infrastructure:** The CI/CD pipeline is configured to use older versions of Python (3.7, 3.8, and 3.9), which means that the incompatibility with newer versions was not detected.

## 2. Proposal for Fix

To address these issues and make the `pywinrt.exe` tool compatible with Python 3.12, the following changes are proposed:

### 2.1. Update C++ Source Code

*   **Replace `PyUnicode_AsWideCharString`:** The `pystring` struct in `src/tool/python/strings/pybase.h` should be updated to use the modern `PyUnicode_AsWideChar` function. This will require pre-allocating a buffer of the correct size and then calling the function to fill it. A `PyUnicode_Check` should also be added to ensure that the object is a valid Unicode object before attempting to convert it.

*   **Update `PyLong_FromLong` and `PyLong_AsLong`:** All calls to `PyLong_FromLong` and `PyLong_AsLong` should be updated to use `long` instead of `int32_t`. This will ensure that the code is compatible with the latest Python C API and the new integer representation.

### 2.2. Update Build and Test Scripts

*   **Update Azure Pipelines Configuration:** The `job-build-projection.yml` and `steps-build-projection.yml` files in `src/package/pywinrt/projection/AzurePipelinesTemplates/` should be updated to use Python 3.12. This will ensure that the CI/CD pipeline builds and tests the project with the correct Python version.

## 3. Build Instructions

To build the `pywinrt.exe` tool, you will need a Windows environment with Visual Studio and CMake installed.

1.  Open a Visual Studio developer command prompt.
2.  Navigate to the root of the xlang repository.
3.  Create a build directory and navigate into it:

    ```
    mkdir build
    cd build
    ```

4.  Run CMake to generate the build files, specifying the Python version you want to use:

    ```
    cmake ../src -DPYTHON_VERSION=3.12
    ```

5.  Build the project:

    ```
    cmake --build .
    ```

This will build the `pywinrt.exe` executable in the `build/tool/python` directory.
