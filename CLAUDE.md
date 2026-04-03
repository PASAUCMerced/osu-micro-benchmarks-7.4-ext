# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Fork of OSU Micro-Benchmarks (OMB) v7.4 extended with multi-pair one-sided MPI latency and bandwidth tests (`osu_get_multi_lat`, `osu_get_mbw`). These benchmarks measure MPI communication performance (latency, bandwidth, message rate) across point-to-point, one-sided, collective, and startup operations.

## Build Commands

```bash
# Basic MPI build
./configure CC=/path/to/mpicc CXX=/path/to/mpicxx
make
make install

# With CUDA support
./configure CC=/path/to/mpicc CXX=/path/to/mpicxx \
    --enable-cuda \
    --with-cuda-include=/path/to/cuda/include \
    --with-cuda-libpath=/path/to/cuda/lib

# With ROCm support
./configure CC=/path/to/mpicc CXX=/path/to/mpicxx \
    --enable-rocm --with-rocm=/path/to/rocm/install
```

Uses GNU Autotools (autoconf/automake/libtool). After modifying `configure.ac` or any `Makefile.am`, regenerate with `autoreconf -ivf`.

## Running Benchmarks

Benchmarks are MPI programs run via `mpirun`/`mpiexec`:
```bash
mpirun -np 2 ./c/mpi/pt2pt/standard/osu_latency
mpirun -np 2 ./c/mpi/one-sided/osu_get_mbw
```

## Code Architecture

- **`c/mpi/pt2pt/`** - Point-to-point benchmarks (latency, bandwidth, multi-latency, message rate). Split into `standard/`, `persistent/`, `congestion/`.
- **`c/mpi/one-sided/`** - RMA benchmarks (put/get latency, bandwidth). Contains the fork-added `osu_get_multi_lat.c` and `osu_get_mbw.c`.
- **`c/mpi/collective/`** - Collective benchmarks split into `blocking/`, `non_blocking/`, `neighborhood/`, `persistent/`.
- **`c/mpi/startup/`** - MPI initialization benchmarks.
- **`c/util/`** - Shared utility library used by all C benchmarks. Key files:
  - `osu_util.c/h` - Core utilities (timing, memory allocation, argument parsing)
  - `osu_util_mpi.c/h` - MPI-specific helpers
  - `osu_util_graph.c/h` - Graph/output utilities
  - `osu_util_papi.c/h` - PAPI performance counter integration
  - `osu_util_validation.c` - Data validation routines
- **`c/xccl/`** - NCCL/RCCL collective and pt2pt benchmarks.
- **`c/openshmem/`, `c/upc/`, `c/upcxx/`** - Non-MPI communication library benchmarks.
- **`python/`** - Python MPI benchmark equivalents (no compilation needed).

Each benchmark is a standalone C source file that links against the shared `c/util/` utilities. New benchmarks are added by creating a `.c` file and registering it in the appropriate `Makefile.am` (add to `_PROGRAMS` and define `_SOURCES`).

## Key Configure Options

- `--enable-cuda` / `--enable-rocm` - GPU memory buffer support
- `--enable-ncclomb` / `--enable-rcclomb` - NCCL/RCCL collective benchmarks (mutually exclusive)
- `--enable-openacc` - OpenACC support
- `--enable-papi` - Hardware performance counters
- `--enable-mpi4` - MPI-4 Session support (auto-detected by default)
