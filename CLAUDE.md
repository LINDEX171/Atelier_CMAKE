# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a C calculator workshop project (`Atelier_CMAKE`) focused on learning GitHub Actions CI/CD with CMake. It implements a simple command-line calculator as the application under test.

## Build & Test Commands

```bash
# Configure and build
mkdir build && cd build
cmake ..
make

# Run all tests (from build/)
make test

# Run the calculator directly (from build/)
./src/calculator add 2 3
./src/calculator sub 3 -2
./src/calculator mul 5 5
./src/calculator div 1 5
```

The calculator takes three arguments: `<operation> <a> <b>` where operation is `add`, `sub`, `mul`, or `div`.

## Architecture

```
CMakeLists.txt         # Root: defines project, enables testing, registers tests via add_test()
include/
  calculator.h         # Function declarations for all operations
  CMakeLists.txt       # (empty — include_directories handled at root)
src/
  main.c               # Entry point: parses argv[1] as operation, argv[2]/argv[3] as doubles
  CMakeLists.txt       # Builds 'calculator' executable, links against 'lib'
  lib/
    calculator.c       # Implements _add, _sub, _mul, _div
    CMakeLists.txt     # Builds static library 'lib' from *.c files in this directory
```

Tests are registered in the root `CMakeLists.txt` with `add_test()` — they invoke the compiled binary with specific arguments and check the exit code (not stdout).

## Adding a New Operation

To add a new operation (e.g., `car` for square):
1. Declare it in `include/calculator.h`
2. Implement it in `src/lib/calculator.c`
3. Add the dispatch case in `src/main.c` (pattern: `strcmp(op, "car") == 0`)
4. Register a test in root `CMakeLists.txt`: `add_test(t5 src/calculator car 5)`

## GitHub Actions

Workflows live in `.github/workflows/`. The CI pattern is:
- **Compilation** job: clones repo, runs `cmake .. && make && make test`
- Additional jobs use `needs: Compilation` to run after compilation, then invoke the binary directly (e.g., `./src/calculator add 10 5`)
- Parallel test jobs + a final `Deploy` job form the full pipeline
