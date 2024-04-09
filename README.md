# Unified Memory Framework

[![Basic builds](https://github.com/oneapi-src/unified-memory-framework/actions/workflows/basic.yml/badge.svg?branch=main)](https://github.com/oneapi-src/unified-memory-framework/actions/workflows/basic.yml)
[![CodeQL](https://github.com/oneapi-src/unified-memory-framework/actions/workflows/codeql.yml/badge.svg?branch=main)](https://github.com/oneapi-src/unified-memory-framework/actions/workflows/codeql.yml)
[![SpellCheck](https://github.com/oneapi-src/unified-memory-framework/actions/workflows/spellcheck.yml/badge.svg?branch=main)](https://github.com/oneapi-src/unified-memory-framework/actions/workflows/spellcheck.yml)
[![GitHubPages](https://github.com/oneapi-src/unified-memory-framework/actions/workflows/docs.yml/badge.svg?branch=main)](https://github.com/oneapi-src/unified-memory-framework/actions/workflows/docs.yml)
[![Benchmarks](https://github.com/oneapi-src/unified-memory-framework/actions/workflows/benchmarks.yml/badge.svg?branch=main)](https://github.com/oneapi-src/unified-memory-framework/actions/workflows/benchmarks.yml)
[![Nightly](https://github.com/oneapi-src/unified-memory-framework/actions/workflows/nightly.yml/badge.svg?branch=main)](https://github.com/oneapi-src/unified-memory-framework/actions/workflows/nightly.yml)
[![OpenSSF Scorecard](https://api.securityscorecards.dev/projects/github.com/oneapi-src/unified-memory-framework/badge)](https://securityscorecards.dev/viewer/?uri=github.com/oneapi-src/unified-memory-framework)
[![Coverity build](https://github.com/oneapi-src/unified-memory-framework/actions/workflows/coverity.yml/badge.svg?branch=main)](https://github.com/oneapi-src/unified-memory-framework/actions/workflows/coverity.yml)
[![Coverity report](https://scan.coverity.com/projects/29761/badge.svg?flat=0)](https://scan.coverity.com/projects/oneapi-src-unified-memory-framework)
[![Bandit](https://github.com/oneapi-src/unified-memory-framework/actions/workflows/bandit.yml/badge.svg?branch=main)](https://github.com/oneapi-src/unified-memory-framework/actions/workflows/bandit.yml)

## Introduction

The Unified Memory Framework (UMF) is a library for constructing allocators and memory pools. It also contains broadly useful abstractions and utilities for memory management. UMF allows users to manage multiple memory pools characterized by different attributes, allowing certain allocation types to be isolated from others and allocated using different hardware resources as required.

## Usage

For a quick introduction to UMF usage, see [Example usage](https://oneapi-src.github.io/unified-memory-framework/example-usage.html),
which includes the code of the basic [example](https://github.com/oneapi-src/unified-memory-framework/blob/main/examples/basic/basic.c).

## Build

### Requirements

Required packages:
- libhwloc-dev (Linux) / hwloc (Windows)
- C compiler
- [CMake](https://cmake.org/) >= 3.14.0

For development and contributions:
- clang-format-15.0 (can be installed with `python -m pip install clang-format==15.0.7`)
- cmake-format-0.6 (can be installed with `python -m pip install cmake-format==0.6.13`)

For building tests, multithreaded benchmarks and Disjoint Pool:
- C++ compiler with C++17 support

For Level Zero memory provider tests:
- Level Zero headers and libraries
- compatible GPU with installed driver

### Linux

Executable and binaries will be in **build/bin**

```bash
$ mkdir build
$ cd build
$ cmake {path_to_source_dir}
$ make
```

### Windows

Generating Visual Studio Project. EXE and binaries will be in **build/bin/{build_config}**

```bash
$ mkdir build
$ cd build
$ cmake {path_to_source_dir} -G "Visual Studio 15 2017 Win64"
```

### Benchmark

UMF comes with a single-threaded micro benchmark based on [ubench](https://github.com/sheredom/ubench.h).
In order to build the benchmark, the `UMF_BUILD_BENCHMARKS` CMake configuration flag has to be turned `ON`.

UMF also provides multithreaded benchmarks that can be enabled by setting both
`UMF_BUILD_BENCHMARKS` and `UMF_BUILD_BENCHMARKS_MT` CMake
configuration flags to `ON`. Multithreaded benchmarks require a C++ support.

### Sanitizers

List of sanitizers available on Linux:
- AddressSanitizer
- UndefinedBehaviorSanitizer
- ThreadSanitizer
   - Is mutually exclusive with other sanitizers.
- MemorySanitizer
   - Requires linking against MSan-instrumented libraries to prevent false positive reports. More information [here](https://github.com/google/sanitizers/wiki/MemorySanitizerLibcxxHowTo).

List of sanitizers available on Windows:
- AddressSanitizer

Listed sanitizers can be enabled with appropriate [CMake options](#cmake-standard-options).

### CMake standard options

List of options provided by CMake:

| Name | Description | Values | Default |
| - | - | - | - |
| UMF_BUILD_SHARED_LIBRARY | Build UMF as shared library | ON/OFF | OFF |
| UMF_BUILD_LEVEL_ZERO_PROVIDER | Build Level Zero memory provider | ON/OFF | ON |
| UMF_BUILD_LIBUMF_POOL_DISJOINT | Build the libumf_pool_disjoint static library | ON/OFF | OFF |
| UMF_BUILD_LIBUMF_POOL_JEMALLOC | Build the libumf_pool_jemalloc static library | ON/OFF | OFF |
| UMF_BUILD_LIBUMF_POOL_SCALABLE | Build the libumf_pool_scalable static library | ON/OFF | OFF |
| UMF_BUILD_TESTS | Build UMF tests | ON/OFF | ON |
| UMF_BUILD_GPU_TESTS | Build UMF GPU tests | ON/OFF | OFF |
| UMF_BUILD_BENCHMARKS | Build UMF benchmarks | ON/OFF | OFF |
| UMF_BUILD_EXAMPLES | Build UMF examples | ON/OFF | ON |
| UMF_ENABLE_POOL_TRACKING | Build UMF with pool tracking | ON/OFF | ON |
| UMF_DEVELOPER_MODE | Treat warnings as errors and enables additional checks | ON/OFF | OFF |
| UMF_FORMAT_CODE_STYLE | Add clang, cmake, and black -format-check and -format-apply targets to make | ON/OFF | OFF |
| USE_ASAN | Enable AddressSanitizer checks | ON/OFF | OFF |
| USE_UBSAN | Enable UndefinedBehaviorSanitizer checks | ON/OFF | OFF |
| USE_TSAN | Enable ThreadSanitizer checks | ON/OFF | OFF |
| USE_MSAN | Enable MemorySanitizer checks | ON/OFF | OFF |
| USE_VALGRIND | Enable Valgrind instrumentation | ON/OFF | OFF |

## Architecture: memory pools and providers

A UMF memory pool is a combination of a pool allocator and a memory provider. A memory provider is responsible for coarse-grained memory allocations and management of memory pages, while the pool allocator controls memory pooling and handles fine-grained memory allocations.

Pool allocator can leverage existing allocators (e.g. jemalloc or tbbmalloc) or be written from scratch.

UMF comes with predefined pool allocators (see include/pool) and providers (see include/provider). UMF can also work with user-defined pools and providers that implement a specific interface (see include/umf/memory_pool_ops.h and include/umf/memory_provider_ops.h).

More detailed documentation is available here: https://oneapi-src.github.io/unified-memory-framework/

### Memory providers

#### OS memory provider

A memory provider that provides memory from an operating system.

##### Requirements

Required packages for tests (Linux-only yet):
   - libnuma-dev

#### Level Zero memory provider (Linux-only yet)

A memory provider that provides memory from L0 device.

##### Requirements

1) Linux or Windows OS
2) The `UMF_BUILD_LEVEL_ZERO_PROVIDER` option turned `ON` (by default)

Additionally, required for tests:

3) Linux OS
4) The `UMF_BUILD_GPU_TESTS` option turned `ON`
5) System with Level Zero compatible GPU
6) Required packages:
   - liblevel-zero-dev

### Memory pool managers

#### proxy_pool (part of libumf)

This memory pool is distributed as part of libumf. It forwards all requests to the underlying
memory provider. Currently umfPoolRealloc, umfPoolCalloc and umfPoolMallocUsableSize functions
are not supported by the proxy pool.

#### libumf_pool_disjoint

TODO: Add a description

##### Requirements

To enable this feature, the `UMF_BUILD_LIBUMF_POOL_DISJOINT` option needs to be turned `ON`.

#### libumf_pool_jemalloc

libumf_pool_jemalloc is a [jemalloc](https://github.com/jemalloc/jemalloc)-based memory pool manager built as a separate static library.
The `UMF_BUILD_LIBUMF_POOL_JEMALLOC` option has to be turned `ON` to build this library.

##### Requirements

1) The `UMF_BUILD_LIBUMF_POOL_JEMALLOC` option turned `ON`
2) Required packages:
   - libjemalloc-dev (Linux) or jemalloc (Windows)

#### libumf_pool_scalable

libumf_pool_scalable is a [oneTBB](https://github.com/oneapi-src/oneTBB)-based memory pool manager built as a separate static library.
The `UMF_BUILD_LIBUMF_POOL_SCALABLE` option has to be turned `ON` to build this library.

##### Requirements

1) The `UMF_BUILD_LIBUMF_POOL_SCALABLE` option turned `ON`
2) Required packages:
   - libtbb-dev (libtbbmalloc.so.2) on Linux or tbb (tbbmalloc.dll) on Windows

### Memspaces (Linux-only)

TODO: Add general information about memspaces.

#### Host all memspace

Memspace backed by all available NUMA nodes discovered on the platform. Can be retrieved
using umfMemspaceHostAllGet.

#### Highest capacity memspace

Memspace backed by all available NUMA nodes discovered on the platform sorted by capacity.
Can be retrieved using umfMemspaceHighestCapacityGet.

## Contributions

All contributions to the UMF project are most welcome! Before submitting
an issue or a Pull Request, please read [Contribution Guide](./CONTRIBUTING.md).
