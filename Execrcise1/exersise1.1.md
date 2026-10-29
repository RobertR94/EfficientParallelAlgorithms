# 1.1.1 Version Control

## General Purpose

1. **Tracking changes**: Maintain a full history of modifications and commit messages over time.
2. **Parallel development**: Work on features, bug fixes, and experiments in isolated branches without disrupting main code.
3. **Reversion & recovery**: Safely inspect, roll back, or recover from earlier project states.
4. **Collaboration**: Coordinate work across teams via remote repositories (e.g., GitHub, GitLab).

## In the Context of C++

- **`.gitignore` configuration**:
  - Exclude platform- and compiler-specific build artifacts (e.g., `.o`, `.obj`, `.so`, `.dll`, `.exe`).
  - Exclude build output directories (e.g., `build/`, `bin/`) and IDE configuration files (e.g., `.vscode/`, `.idea/`).

- **Build reproducibility**:
  - Commit build configuration files (e.g., `CMakeLists.txt`, `CMakePresets.json`) so the project can be built reliably across environments.

- **Dependency management**:
  - Use Git submodules, CMake's `FetchContent`, or package managers (e.g., `vcpkg`, Conan) to manage and track external C++ libraries.

# 1.1.2 Function of Compiler and Linker

## Preprocessor

**Summary**: The preprocessor performs purely text-based transformations (expanding macros, resolving `#include` directives,
and conditional compilation) on source files to produce a single flattened translation unit for the compiler.

- **Role & Execution**:
  - Operates directly on source text before compilation.
  - Removes all source comments (replaces them with whitespace).
  - Can be inspected directly using compiler flags like `g++ -E` or `clang++ -E`.

- **Directive Resolution**:
  - **Header Inclusion (`#include`)**: Textually copies and pastes the entire contents of the referenced header file into the source file to create a single
    **Translation Unit (TU)**.
    - `#include <...>` for system / standard library headers.
    - `#include "..."` for project-local / user headers.
  - **Macro Expansion (`#define`, `#undef`)**: Performs simple token and pattern substitution across the code.
  - **Conditional Compilation (`#ifdef`, `#ifndef`, `#if`, `#elif`, `#else`, `#endif`)**: Selectively includes or excludes blocks of code
    based on conditions (e.g., platform checks, debug vs. release mode via `NDEBUG`).
  - **Header Guards (`#pragma once` or `#ifndef` guards)**: Prevents circular dependencies and duplicate declarations when headers are
    included multiple times across translation units.

## Compiler

**Summary**: The compiler parses and type-checks the preprocessed code, applies optimizations, and translates high-level C++
into target-specific machine code packaged as an object file (`.o` / `.obj`).

- **Compilation Pipeline**:
  - **Frontend (Parsing & Semantic Analysis)**:
    - Tokenizes the preprocessed source and constructs an Abstract Syntax Tree (AST).
    - Performs strict syntax and static type checks.
    - Executes compile-time evaluations (e.g., `constexpr`, `consteval`) and instantiates C++ templates.
  - **Middle-End (Optimization)**:
    - Translates AST into an Intermediate Representation (IR).
    - Applies code optimizations (e.g., function inlining, loop unrolling, auto-vectorization/SIMD, dead code elimination,
      constant folding) driven by optimization flags (e.g., `-O0`, `-O2`, `-O3`).
  - **Backend (Code Generation)**:
    - Translates optimized IR into architecture-specific assembly and binary machine instructions.

- **Output (Object Files)**:
  - Produces one **Object File** (`.o` on Linux/macOS, `.obj` on Windows) per translation unit.
  - Contains compiled machine instructions, debug symbols (if enabled via `-g`), and a **symbol table** containing:
    - _Defined symbols_: Functions and variables implemented in this translation unit.
    - _Undefined (external) symbols_: References to functions/variables declared (e.g., via headers) but implemented in other translation
      units or external libraries, which must be resolved by the linker.

- **Useful Compiler Flags**:
  - `-c`: Compile source files into object files without running the linker.
  - `-S`: Stop after compilation and emit human-readable assembly code (`.s`).
  - `-O2` / `-O3`: Enable standard / high-performance compiler optimizations.
  - `-Wall -Wextra -Wpedantic`: Enable standard compiler diagnostics and warnings.

## Linker

**Summary**: The linker combines multiple compiled object files and external libraries into a single binary, resolving cross-file symbol references and
assigning concrete memory addresses.

- **Object & Library Aggregation**:
  - Merges multiple compiled object files (`.o` / `.obj`) and external static or shared libraries.

- **Symbol Resolution & Relocation**:
  - Resolves undefined external symbols (function calls, global variables) to their definitions across translation units.
  - Assigns concrete memory addresses to code and data (relocation).

- **Target Output**:
  - Produces the final executable binary or library (shared: `.so`, `.dylib`, `.dll`; static: `.a`, `.lib`).

# 1.1.3 Header-Only Library

**Summary**: A header-only library is implemented entirely within header files (`.h` / `.hpp`), eliminating separate
compilation and linking stages while relying on `inline` declarations and templates to satisfy the One Definition Rule (ODR).

- **Concept**:
  - The entire library implementation resides in header files (`.h` or `.hpp`), requiring no pre-compiled binary artifacts
    (`.a`, `.so`, `.lib`, `.dll`).
  - Seamless integration: add headers to the include path (e.g., `-I/path/to/include` or
    CMake `target_include_directories`) and `#include` them directly.

- **C++ Idiosyncrasies & Technical Rules**:
  - **One Definition Rule (ODR)**: Non-templated functions and methods must be declared `inline`
    (or `inline constexpr`), allowing the linker to safely merge duplicate definitions across translation units instead of throwing collision errors (`multiple definition of ...`).
  - **Templates**: Function and class templates are naturally header-only because the compiler requires their
    full implementation at the point of instantiation.

- **Pros & Cons**:
  - **Pros**:
    - _Trivial deployment_: Simple copy/include distribution without platform-specific binary build or linking steps.
    - _Aggressive optimization_: Exposing complete source code enables powerful compiler optimizations like
      cross-module inlining and dead-code elimination.
  - **Cons**:
    - _Longer compile times_: The compiler repeatedly parses and compiles the same header code across every
      translation unit that includes it.
    - _Cascading rebuilds & potential bloat_: Any change to a header triggers recompilation of all dependent
      files, and heavy template instantiations can inflate binary size.

# 1.1.4 Performance Measurement

**Summary**: Performance measurement distinguishes between elapsed real-world time (Wall-Clock Time) and total processor compute work (CPU Time) using monotonic timers and profilers, requiring careful benchmark design to avoid distortions from compiler optimizations and cold caches.

- **Wall-Clock Time vs. CPU Time**:
  - **Wall-Clock Time**:
    - Actual elapsed real-world time from start to completion.
    - Essential for measuring parallel speedup, scalability, and concurrency across multiple threads.
  - **CPU Time**:
    - Cumulative execution time spent by all CPU cores actively executing process threads (user time + system time).
    - Can exceed Wall-Clock Time in parallel programs ($\text{CPU Time} \approx N_{\text{threads}} \times \text{Wall Time}$) or drop below it during I/O waits and sleeps.

- **Tools & Methods**:
  - **Code-Level Timing (`std::chrono`)**:
    - Prefer `std::chrono::steady_clock` for duration benchmarks—it is strictly monotonic and immune to system time or NTP adjustments.
    - `std::chrono::high_resolution_clock` provides the highest precision clock available on the implementation.
  - **System-Level Profiling**:
    - CLI tools like `/usr/bin/time` (measures real, user, sys time) and `perf stat` to inspect hardware performance counters (cache misses, branch mispredictions, IPC).
  - **Dedicated Benchmarking Frameworks**:
    - Tools like Google Benchmark or Catch2 to automate repetitions, filter measurement noise, and compute statistical aggregates.

- **Benchmarking Pitfalls & Mitigations**:
  - **Compiler Optimizations (Dead-Code Elimination)**:
    - At `-O2` / `-O3`, compilers often eliminate unused benchmark computation entirely.
    - *Mitigation*: Accumulate results, pass values to `benchmark::DoNotOptimize()`, or use volatile assembly barriers (`asm volatile("" : "+r"(val));`).
  - **Warmup & Cache Effects**:
    - Initial iterations frequently suffer from cold CPU caches and first-touch page faults.
    - *Mitigation*: Run dedicated warmup iterations before measurement and average over multiple runs.
