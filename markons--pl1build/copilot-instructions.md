## pl1build

> The system comprises two primary components: a benchmark suite and a Levenshtein distance calculator. The benchmark suite evaluates the performance of floating-point and integer computations, while the Levenshtein distance calculator measures the difference between pairs of strings provided via command-line arguments.


START SPECIFICATION:
# Technical Specification

## System Overview
The system comprises two primary components: a benchmark suite and a Levenshtein distance calculator. The benchmark suite evaluates the performance of floating-point and integer computations, while the Levenshtein distance calculator measures the difference between pairs of strings provided via command-line arguments. 

### Main Components
- **Benchmark Suite**: 
  - Source files: `src/benchmark.lst`, `src/benchmark.obj`, `src/benchmark.pli`
  - Primary function: `benchmark`
  - Key algorithms: `taylor_sin`, `gcd`
- **Levenshtein Distance Calculator**:
  - Source files: `src/leven.lst`, `src/leven.obj`, `src/leven.pli`
  - Primary function: `leven`
  - Key algorithms: `levenshtein_distance`, `substring_extractor`
- **Build and Execution Tasks**:
  - Configuration file: `.vscode/tasks.json.old`
  - Tasks: Compile PL/I Task, Link Object File Task, Run Executable Task

## Core Functionality

### Benchmark Suite
#### `benchmark`
**Importance Score: 100**
- **Description**: The main procedure that defines the core behavior of the benchmark. It includes both floating-point and integer benchmarks.
- **Key Components**:
  - **Floating-point benchmark**: Computes the sum of Taylor series approximations for sine over a specified number of iterations.
  - **Integer benchmark**: Computes the sum of GCD calculations between pairs of integers within specified ranges.
  - **Timing**: Measures the elapsed time for each benchmark and handles midnight wrap scenarios.

#### `taylor_sin`
**Importance Score: 90**
- **Description**: Computes the sine of a given value using a Taylor series approximation.
- **Parameters**:
  - `x`: The input value.
  - `terms`: The number of terms to use in the Taylor series.
- **Returns**: The approximated sine value as a float.
- **Key Components**:
  - Iteratively calculates each term of the Taylor series and accumulates the sum.

#### `gcd`
**Importance Score: 85**
- **Description**: Computes the greatest common divisor (GCD) of two integers using the Euclidean algorithm.
- **Parameters**:
  - `a`: The first integer.
  - `b`: The second integer.
- **Returns**: The GCD as a fixed binary integer.
- **Key Components**:
  - Uses a loop to repeatedly apply the Euclidean algorithm until `b` becomes zero.

#### `get_time_ms`
**Importance Score: 80**
- **Description**: Converts a time string in the format `HHMMSSTTT` to milliseconds since midnight.
- **Parameters**:
  - `time_str`: The input time string.
- **Returns**: The time in milliseconds as a float.
- **Key Components**:
  - Extracts hours, minutes, seconds, and thousandths from the input string and converts them to milliseconds.

#### Constants
**Importance Score: 70**
- **Description**: Define workload sizes and step sizes used in the benchmarks.
- **Variables**:
  - `N`: Number of floating-point iterations.
  - `M`: Number of Taylor series terms.
  - `P`: Outer loop size for integer benchmark.
  - `Q`: Inner loop size for integer benchmark.
  - `delta`: Step size for floating-point input.

#### Result and Timing Variables
**Importance Score: 60**
- **Description**: Variables used to store results and timing information for the benchmarks.
- **Variables**:
  - `fp_sum`: Sum of floating-point computations.
  - `int_sum`: Sum of integer computations.
  - `start_time`: Start time as `HHMMSSTTT`.
  - `end_time`: End time as `HHMMSSTTT`.
  - `elapsed`: Elapsed time in milliseconds.

### Levenshtein Distance Calculator
#### `leven`
**Importance Score: 100**
- **Description**: The main procedure that defines the core behavior of the program. It processes command-line arguments, extracts substrings, and calculates the Levenshtein distance between pairs of strings.
- **Key Actions**:
  - Calls `substring_extractor` to parse command-line arguments.
  - Iterates over pairs of arguments to calculate and store the Levenshtein distance.
  - Outputs the number of comparisons and the minimum distance found.

#### `levenshtein_distance`
**Importance Score: 90**
- **Description**: Computes the Levenshtein distance between two strings. This is a critical algorithm in the program, used to measure the difference between strings.
- **Key Actions**:
  - Allocates dynamic arrays `prev`, `curr`, and `temp`.
  - Iterates over the strings to fill the dynamic programming table.
  - Returns the computed Levenshtein distance.

#### `substring_extractor`
**Importance Score: 80**
- **Description**: Extracts substrings from a given parameter string. This function is essential for parsing command-line arguments into individual strings.
- **Key Actions**:
  - Splits the input string based on spaces.
  - Stores the extracted substrings in the `arg` array.
  - Updates `arg_count` to reflect the number of extracted substrings.

#### Core Data Models
- **`parm`, `s1`, `s2`, `arg`**:
  **Importance Score: 70**
  - **Description**: These variables hold string data critical to the program's operation. `parm` is the input string, `s1` and `s2` are the strings whose Levenshtein distance is calculated, and `arg` is an array holding the extracted substrings.
- **`prev`, `curr`, `temp`**:
  **Importance Score: 60**
  - **Description**: Dynamic arrays used in the `levenshtein_distance` procedure to store intermediate results during the calculation of the Levenshtein distance.

### Critical Logic or Algorithms
#### Levenshtein Distance Calculation
- **Importance Score: 95**
- **Description**: The core algorithm implemented in the `levenshtein_distance` procedure. It uses dynamic programming to efficiently compute the minimum number of single-character edits (insertions, deletions, or substitutions) required to change one string into another.

### Main Connection Points with Other System Parts
#### Command-Line Argument Parsing
- **Importance Score: 85**
- **Description**: The program relies on command-line arguments to provide the strings for comparison. The `substring_extractor` procedure is the main interface for parsing these arguments.

### Complex or Non-Obvious Implementations
#### Dynamic Array Allocation and Usage in `levenshtein_distance`
- **Importance Score: 80**
- **Description**: The use of dynamic arrays (`prev`, `curr`, `temp`) to store intermediate results in the Levenshtein distance calculation is a non-trivial implementation detail. It requires careful management to ensure correct results and efficient memory usage.

## Architecture
The system is structured into two main components: the benchmark suite and the Levenshtein distance calculator. 

### Data Flow in Benchmark Suite
1. **Input**: Constants defining workload sizes and step sizes.
2. **Processing**: 
   - `benchmark` procedure calls `taylor_sin` and `gcd` to perform floating-point and integer computations respectively.
   - Timing is measured using `get_time_ms`.
3. **Output**: Results and timing information are stored in variables and can be output using `sysprint`.

### Data Flow in Levenshtein Distance Calculator
1. **Input**: Command-line arguments providing strings for comparison.
2. **Processing**: 
   - `leven` procedure calls `substring_extractor` to parse arguments.
   - `levenshtein_distance` procedure computes the Levenshtein distance between pairs of strings.
3. **Output**: The number of comparisons and the minimum distance found are output.

### Build and Execution Tasks
- **Compile PL/I Task**: Transforms source code into an object file using `plic`.
- **Link Object File Task**: Creates the final executable from the object file using `ld`.
- **Run Executable Task**: Executes the generated program, optionally with parameters.
END SPECIFICATION

---
> Source: [markons/pl1build](https://github.com/markons/pl1build) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
