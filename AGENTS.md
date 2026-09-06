# MORI

Modular RDMA Interface — a C++/Python GPU communication library for P2P, RDMA/IBGDA, and SDMA.

## Project layout

```
src/              C++ implementation
  application/    MORI-EP, MORI-IO, MORI-CCL, MORI-UMBP
  cco/            Core communication objects (transport, queues, GDA/LSA)
  collective/     Collective primitives
  io/             P2P communication engine/backend/session
  jit/            JIT helpers
  metrics/        Profiling and telemetry
  ops/            Device-side operator implementations
  pybind/         Python bindings
  shmem/          OpenSHMEM-style symmetric-memory APIs
  umbp/           Unified memory & bandwidth pool
python/           Python package (`amd_mori`)
  mori/
    ccl/          Collective communication Python API
    cco/          CCO Python API
include/          Public C++ headers
examples/         Python and C++ usage examples
benchmark/        Microbenchmarks
tests/            C++ gtests and Python pytest tests
csrc/             (deprecated — kept for compatibility)
docker/           CI and development containers
```

## Setup commands

```bash
# Clone with submodules
git clone --recursive https://github.com/awdemos/mori.git
# or, if already cloned:
git submodule sync && git submodule update --init --recursive

# Install build deps
python3 -m pip install -r requirements-build.txt

# Build and install the Python package (develop mode)
pip install -e .

# Build with tests/examples/benchmarks enabled
BUILD_TESTS=ON BUILD_EXAMPLES=ON BUILD_BENCHMARK=ON MORI_WITH_MPI=ON pip install .
```

## Build/test/lint commands

```bash
# Python tests
python3 -m pytest tests/python/

# C++ tests (after building with BUILD_TESTS=ON)
# run via the CI helper:
tools/run_cco_tests.sh ./build 4

# CCO SDMA tests (single test binary, 8 ranks)
./build/tests/cpp/cco/test_sdma_put 8

# Benchmarks
./build/benchmark/

# Pre-commit hooks
pre-commit run --all-files

# Python formatting/linting is enforced by pre-commit (black, ruff, etc.)
```

## Key conventions

- C++ code is formatted with `.clang-format`; linting uses `.clang-tidy`.
- Python package name is `amd_mori`; the CLI entry point is `mori`.
- Versioning uses `setuptools_scm` from git tags; fallback is `0.1.0`.
- Python target is `>=3.10`.
- CMake is the primary build system; `setup.py` wraps it for Python installs.
- `BUILD_TESTS`, `BUILD_EXAMPLES`, `BUILD_BENCHMARK`, `MORI_WITH_MPI`, `BUILD_UMBP`, and `BUILD_CCO_SDMA` are common CMake/ env toggles.
- CI runs on self-hosted runners with AMD GPUs and RDMA devices; the default GitHub Actions runner cannot execute the GPU/RDMA tests.

## Gotchas

- Submodules under `3rdparty/` are required; initialize them before building.
- `pyproject.toml` caps `setuptools<84` because the custom `build_ext` lifecycle invokes `build_extension()` directly. Removing this cap requires fixing the lifecycle.
- FlyDSL device bindings are optional: install with `pip install amd_mori[flydsl]`.
- The CCO CI skips GDA-FULL tests on runners without intranode cross-rail RDMA (`MORI_CCO_SKIP_GDA_FULL=1`).
- SDMA tests must not SKIP; a SKIP indicates SDMA queues are unavailable and is treated as a failure in CI.
- `BUILD_UMBP=OFF` is used when UMBP gtest discovery breaks under newer CMake.
