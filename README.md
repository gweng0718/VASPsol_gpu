# VASP Solvation GPU Support

This repository contains a GPU-accelerated implementation of the VASP solvation module (based on `VASPsol` functionality) using OpenACC.

## Features

- **OpenACC Acceleration**: The core solvation logic (`solvation.F`) has been ported to GPU using OpenACC directives, suitable for `nvc` / NVIDIA compilers.
- **VASP Integration**: Designed to be integrated into the VASP source code tree.
- **Validation**: Includes test cases for comparing CPU vs GPU performance and accuracy.

## Installation

1.  Copy `solvation.F` into your VASP source directory, replacing the existing file if necessary (or adding it to the build list).
2.  Enable OpenACC in your makefile (`makefile.include`). Typically add `-acc -gpu=cc70,cc80` (adjust compute capability as needed) and `-D_OPENACC` to your preprocessor flags.
3.  Compile VASP as usual.

## Usage

Usage is identical to the standard VASP solvation module. Define solvation parameters in `INCAR` (e.g., `LSOL=.TRUE.`, `EB_K`, etc.).

## Testing & Comparison

The `tests/` directory contains input files for validating the GPU implementation against CPU results.

### Directory Structure
- `tests/cpu/`: Input files tuned for CPU execution.
- `tests/gpu/`: Input files tuned for GPU execution.

### Configuration Differences
To optimize performance on GPU, some `INCAR` parallelization parameters are adjusted in the test suite:

| Parameter | CPU Value | GPU Value | Reason |
|-----------|-----------|-----------|--------|
| `KPAR`    | `16`      | `2`       | Reduced K-point parallelization to fit in GPU memory. |
| `NCORE`   | `2`       | `1`       | Optimized for GPU compute patterns (often `NCORE=1` is preferred). |
| `NPAR`    | `8`       | `8`       | set the same |

### Running Tests
1.  Run the CPU benchmark in `tests/cpu`.
2.  Run the GPU benchmark in `tests/gpu`.
3.  Compare `OUTCAR` energies and solvation contributions to ensure agreement.

### Performance Benchmark
Based on the provided test cases:

| Architecture | Elapsed Time (s) |
|--------------|------------------|
| **CPU**      | 461.294          |
| **GPU**      | 377.169          |

*Note: The GPU speedup observed here (~1.2x) is for specific test parameters (`KPAR`, `NCORE`) and may vary significantly with system size and hardware.*
