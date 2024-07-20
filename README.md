# JUPITER Benchmark Suite: Graph500

[![DOI](https://zenodo.org/badge/831462825.svg)](https://zenodo.org/badge/latestdoi/831462825) [![Static Badge](https://img.shields.io/badge/DOI%20(Suite)-10.5281%2Fzenodo.12737073-blue)](https://zenodo.org/badge/latestdoi/764615316)

This benchmark is part of the [JUPITER Benchmark Suite](https://github.com/FZJ-JSC/jubench). See the repository of the suite for some general remarks.

This repository contains the Graph500 benchmark. [`DESCRIPTION.md`](DESCRIPTION.md) contains details for compilation, execution, and evaluation.

The source code of a Graph500 implementation is included in the `./src/` subdirectory as a submodule from [github.com/damianam/graph500](https://github.com/damianam/graph500). The included version is adapted to enable the reference implementation for modern compilers (mostly newer releases of GCC), add small bugfixes related to libnuma, and allow decision at compile time if the benchmark should decide the pinning strategy or if this should be determined externally (via the resource manager).

This benchmark does not rely on GPUs, and does not support threading.