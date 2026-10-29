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

The preprocessor is the first stage in the C++ build pipeline. It performs purely text-based transformations on the source code before actual syntactic analysis and compilation begin.

- **Role & Execution**:
  - Operates directly on source text before compilation.
  - Removes all source comments (replaces them with whitespace).
  - Can be inspected directly using compiler flags like `g++ -E` or `clang++ -E`.

- **Directive Resolution**:
  - **Header Inclusion (`#include`)**: Textually copies and pastes the entire contents of the referenced header file into the source file to create a single **Translation Unit (TU)**.
    - `#include <...>` for system / standard library headers.
    - `#include "..."` for project-local / user headers.
  - **Macro Expansion (`#define`, `#undef`)**: Performs simple token and pattern substitution across the code.
  - **Conditional Compilation (`#ifdef`, `#ifndef`, `#if`, `#elif`, `#else`, `#endif`)**: Selectively includes or excludes blocks of code based on conditions (e.g., platform checks, debug vs. release mode via `NDEBUG`).
  - **Header Guards (`#pragma once` or `#ifndef` guards)**: Prevents circular dependencies and duplicate declarations when headers are included multiple times across translation units.

## Compiler

The compiler takes the preprocessed translation unit (e.g., `.cpp`, `.cxx`, `.cc`) and translates it into target-specific machine code, packaging it into an object file.

- **Compilation Pipeline**:
  - **Frontend (Parsing & Semantic Analysis)**:
    - Tokenizes the preprocessed source and constructs an Abstract Syntax Tree (AST).
    - Performs strict syntax and static type checks.
    - Executes compile-time evaluations (e.g., `constexpr`, `consteval`) and instantiates C++ templates.
  - **Middle-End (Optimization)**:
    - Translates AST into an Intermediate Representation (IR).
    - Applies code optimizations (e.g., function inlining, loop unrolling, auto-vectorization/SIMD, dead code elimination, constant folding) driven by optimization flags (e.g., `-O0`, `-O2`, `-O3`).
  - **Backend (Code Generation)**:
    - Translates optimized IR into architecture-specific assembly and binary machine instructions.

- **Output (Object Files)**:
  - Produces one **Object File** (`.o` on Linux/macOS, `.obj` on Windows) per translation unit.
  - Contains compiled machine instructions, debug symbols (if enabled via `-g`), and a **symbol table** containing:
    - *Defined symbols*: Functions and variables implemented in this translation unit.
    - *Undefined (external) symbols*: References to functions/variables declared (e.g., via headers) but implemented in other translation units or external libraries, which must be resolved by the linker.

- **Useful Compiler Flags**:
  - `-c`: Compile source files into object files without running the linker.
  - `-S`: Stop after compilation and emit human-readable assembly code (`.s`).
  - `-O2` / `-O3`: Enable standard / high-performance compiler optimizations.
  - `-Wall -Wextra -Wpedantic`: Enable standard compiler diagnostics and warnings.
