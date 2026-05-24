## westmarch-matching-fast

> This project implements a fast edit distance (Levenshtein distance) calculator using C++ with Python bindings via Pybind11.

## Project: Fast Edit Distance Calculator

This project implements a fast edit distance (Levenshtein distance) calculator using C++ with Python bindings via Pybind11.

### Core Components:
1.  `edit_distance.cpp`: Contains the C++ implementation of `calculateEditDistance` and `findKClosestWords`.
2.  `bindings.cpp`: Uses Pybind11 to expose the C++ functions to Python. The Python module is named `edit_distance_cpp`.
3.  `setup.py`: Script to build the C++ extension module.
4.  `main.py`: Python script demonstrating the usage of the compiled module.
5.  `test_edit_distance.py`: Unittest suite for testing the functionality.

### Build Instructions:

1.  **Prerequisites**:
    *   A C++11 (or newer) compatible compiler (e.g., g++ on Linux, Clang on macOS, MSVC on Windows).
    *   Python (3.6+ recommended).
    *   `pip` for installing Python packages.

2.  **Install Pybind11**:
    If not already installed, Pybind11 is required for building. The `setup.py` script lists it in `setup_requires`. You can also install it manually:
    ```bash
    pip install pybind11
    ```

3.  **Build the C++ Extension**:
    Navigate to the root directory of the project (where `setup.py` is located) and run:
    ```bash
    python setup.py build_ext --inplace
    ```
    This command compiles the C++ code and creates a shared object file (e.g., `edit_distance_cpp.cpython-XX-architecture.so` on Linux/macOS, or `edit_distance_cpp.cpXX-win_amd64.pyd` on Windows) in the current directory. The exact name depends on your Python version and OS.

### Running the Code:

1.  **Run the Example**:
    After successfully building the module, you can run the example script:
    ```bash
    python main.py
    ```
    This will use the compiled C++ module to find closest words for sample data.

2.  **Run Tests**:
    To verify the implementation, run the unit tests:
    ```bash
    python test_edit_distance.py
    ```
    Ensure the C++ module is built and accessible in the current directory or Python path before running tests. The tests will be skipped if the module cannot be imported.

### Important Notes for AI Agent:

*   The `edit_distance.cpp` file contains the core algorithm. The Wagner-Fischer algorithm is used for Levenshtein distance.
*   The `bindings.cpp` file maps C++ `std::vector<std::string>` to Python lists automatically due to `<pybind11/stl.h>`.
*   The `findKClosestWords` function in C++ sorts results based on distance. If multiple words have the same distance, their relative order in the output list might depend on their original order in the vocabulary and the stability of the sort algorithm used (`std::sort` is not guaranteed to be stable for equivalent elements, but it sorts `WordDistance` objects where distance is the primary key). The tests account for potential variations in order for items with identical distances if they are not the primary sorted item.
*   If making changes to the C++ code (`edit_distance.cpp` or `bindings.cpp`), you **must** re-run the build command (`python setup.py build_ext --inplace`) for the changes to take effect in the Python module.
*   The Python scripts (`main.py`, `test_edit_distance.py`) expect the compiled module to be importable as `edit_distance_cpp`.

### Code Structure Verification:
After any modifications, ensure the following commands run successfully:
1. `python setup.py build_ext --inplace`
2. `python main.py` (check for reasonable output)
3. `python test_edit_distance.py` (all tests should pass)

If tests related to `find_k_closest_words_cpp` fail due to order differences for same-distance words, verify that the primary sorting criterion (distance) is correct and that the set of words returned for a given distance is correct. The current tests for `find_k_closest_words_cpp` are designed to be somewhat flexible regarding the order of same-distance words (beyond the first one).

---
> Source: [moria-ai-labs/westmarch_matching_fast](https://github.com/moria-ai-labs/westmarch_matching_fast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-23 -->
