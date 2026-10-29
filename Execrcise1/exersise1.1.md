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
