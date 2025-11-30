# Walkthrough - Python 3.9+ Compatibility Fixes

## 1. The Problem (Simply Explained)
The code was using an "old way" of talking to Python that worked in older versions (Python 3.7/3.8) but is dangerous or broken in newer ones (Python 3.9+).

Specifically, it was using a format code `u` to read text strings. In the past, Python stored strings in a specific way that `u` could read directly. But modern Python optimizes how it stores strings, so reading them directly like that can cause crashes or garbage data.

## 2. The Fix
We switched to a "safe way" that asks Python to convert the string for us.

*   **Old Code**: "Give me the raw memory address of this string." (Dangerous!)
*   **New Code**: "Here is a converter function. Please use it to turn this Python string into the C++ string I need." (Safe!)

We also fixed some number types (like `int8` and `char`) to use safer standard formats (`h` for short integer, `C` for character) instead of the custom/unsafe ones (`y`, `u`).

## 3. How to Test on Windows (PowerShell)

Since you are testing on Windows, you will use Visual Studio and PowerShell.

### Step 1: Build the Tool (`pywinrt.exe`)
1.  Open **Visual Studio** (2019 or 2022).
2.  Open the `src` folder as a CMake project (File > Open > CMake...).
3.  Select `xlang.exe` (or `pywinrt`) as the startup item and build it.
    *   *Alternatively, from Developer PowerShell:*
        ```powershell
        cd src
        mkdir build
        cd build
        cmake ..
        cmake --build . --target pywinrt --config Release
        ```
    *   This creates `pywinrt.exe` (usually in `src\build\Release` or `src\out\build\x64-Release\tool\python`).

### Step 2: Generate the Python Code
Use the tool to generate the Python extension code.

```powershell
# Adjust paths to match your build location
$PyWinRT = ".\src\build\Release\pywinrt.exe"
$WinMDPath = "C:\Program Files (x86)\Windows Kits\10\UnionMetadata\10.0.19041.0" # Example SDK path

# Generate code
& $PyWinRT -input $WinMDPath -output .\generated_code -module winrt -include Windows.Foundation
```

### Step 3: Compile the Python Extension
Compile the generated code into a Python module (`.pyd`).

```powershell
cd generated_code
python setup.py build_ext --inplace
```

### Step 4: Run the Test
Run the test script.

```powershell
# Go to the test directory
cd ..\src\test\python

# Add the generated code to PYTHONPATH so Python can find the 'winrt' module
$env:PYTHONPATH = "..\..\..\generated_code"

# Run the test
python testuri.py
```

### Expected Result
If the fix works, `testuri.py` should pass all tests. This confirms that:
1.  Strings are being passed correctly from Python to C++.
2.  No crashes occur during string conversion.
