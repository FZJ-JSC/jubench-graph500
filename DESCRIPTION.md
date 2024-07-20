# Graph500

## Purpose

Graph500 is a synthetic benchmark focused on graph traversal that mimics the methods used in a significant amount of
data-intensive workloads. The benchmark is used in the supercomputer ranking with the same name. It has three kernels.
The first is the graph construction, and the remaining 2 are used in 2 different classifications/rankings on the
http://graph500.org website. They are BFS (Breadth-First Search) and SSSP (Single Source Shortest Paths). One can
participate in either ranking, independently from the other one.

This benchmark is a tweaked reference implementation of the Graph500 benchmark. This tweaked version is available in
https://github.com/damianam/graph500. The tweaking is limited to make the reference implementation compile using modern
compilers (mostly newer releases of GCC), small bugfixes related to libnuma, and allowing to decide at compile time if
the benchmark should decide the pinning strategy or if this should be determined externally (via the resource manager).

This benchmark does not rely on GPUs, and does not support threading.

It is important to note that this benchmark in this context is used for evaluation of a particular proposal and for fair
comparison of the capabilities of the systems of the different candidates. Therefore, algorithm optimizations to address
shortcomings of the reference implementation are not possible at this stage and the implementation at hand is to be used.

## Source

Archive name: `graph500-bench.tar.gz`

The file holds the source code of the Graph500 variant, instructions to run the benchmark, according JUBE scripts, and configuration files.

The source code for the benchmark is located in the `src/graph500/` directory. It is equivalent to version [v3.0.2 of the upstream repository](https://github.com/damianam/graph500/tree/v3.0.2).

### Modification

This is a tweaked version of the Graph500 reference implementation, to slightly refresh the code base so it works out of
the box with modern compilers. Either this refreshed version or the reference upstream version can be used in the scope of this procurement; no further modifications of the source code are
allowed, except for correcting similar issues with particular compilers. Specifically, algorithm modifications are not
allowed, so a fair comparison is possible.

The `-DBENCHPIN` compiler switch is optional and can possibly be omitted.

Any modification to the source code is to be communicated as part of the results presentation.

## Building

Compiling Graph500 is straightforward. Simply enter the `src/graph500/src/` directory. There, typing `make` is all that
is needed. That assumes that an MPI compiler and runtime are available in the environment with `mpicc` as a compiler
wrapper, as well as `libnuma`.

`-DBENCHPIN` can be passed on to the compiler, to enable the internal pinning mechanism. This is optional, and depending
on your hardware and resource manager configuration, might be sub-optimal.

### JUBE

The JUBE step `compile` takes care of building the benchmark. It copies the source from `src/` and builds
the benchmark. The provided example is tailored to the JUWELS Booster system. Below, there is an example on how to use
JUBE for compiling, submitting and analyzing the results.

## Execution

After compiling Graph500, two executables are created: `graph500_reference_bfs` and `graph500_reference_bfs_sssp`. The only
difference between them is the inclusion of the SSSP kernel.

### Command Line

This benchmark must be executed using a power of 2 number of MPI tasks.

The benchmark accepts 2 arguments:

- `scale` is the first and main argument and determines the size of the problem. The scale factor 30 is mandatory for the smaller setups required (four nodes and sixteen nodes). For full system runs, other scale factors can be used.
    - The initial point for problem size 30 (`scale`) is 4 nodes, due to the memory requirements.
    - Scale 30 was chosen to maximize scalability to a reasonable number of nodes.
- `edgefactor` is the second argument, and is to be left at 16. No other values are accepted in this procurement.

A table with some common sizes can be found in https://graph500.org/?page_id=12#sec-3_4

A full command line call to the Graph500 benchmark looks like:

```
[mpiexec] graph500_reference_bfs_sssp 30 16
```

Another important factor for running the test is the enablement/disablement of the the validation with the environment
variable `SKIP_VALIDATION=1`. Skipping this verification step is accepted in this benchmark for this procurement.

### Configurations

The requested results must fulfill these requirements:

- The `scale` factor must be 30 in all cases with the single exception of full system results, for which the candidate can choose a scale factor
- `edgefactor` must be 16
- Verification/validation is not mandatory
- It is up to the candidate to decide on a pinning strategy. The candidate can decide to use or avoid the built-in pinning in the benchmark code or any other external tool to pin processes.
- Any other part of the setup must respect Graph500 submission rules and rely on the code and algorithm provided in this benchmark

### JUBE

The JUBE step `execute` calls the aforementioned command line with the correct modules. It also cares about the MPI
distribution by submitting a script to the batch system. The latter is achieved by populating a batch submission script
template (via `platform.xml`) with information specified in the top of the script relating to the number of nodes and
tasks per node. Via dependencies, the JUBE step `execute` calls the JUBE step `compile` automatically.

To submit a fully-contained benchmark run to the batch system you can follow steps analogous to these ones:

```
jutil env activate -A jscbenchmark -p JSCbenchmark
jube run benchmark/jube/jube-graph500-juwelsbooster.yml
```

The provided JUBE YAML file has been tailored to the JUWELS Booster system, but it can be easily adapted to other systems.
Relevant points of the provided configuration for JUWELS Booster:

- The number of tasks should be a power of 2, which in the 48-cores per node setup of the JUWELS Booster results in
  48-32=16 cores idling
- The edge factor is set to 16
- The current configuration relies on GCC and OpenMPI for best performance at the screening time
- The internal pinning mechanism has been disabled in this provided setup. It can be enabled again adding `-DBENCHPIN` to the now empty `CFLAGS` variable in the `compile_opts` section. A variant that tests both possibilities has been left as a comment in the configuration file. If you would like to test both variants please uncomment the following lines (and delete the current used parameters) in the `compile_opts` section:
    ```
      - { name: comp_opts_index, type: int, _: "0,1" }
      - { name: prep_commands, mode: python, _: '["CFLAGS=\"-DBENCHPIN\"", "CFLAGS=\"\""][$comp_opts_index]' }
    ```
- The setup with less than 64 nodes nodes forces Slurm to allocate the job in a single DragonFly group to maximize
network performance and minimize interference from other jobs
- The setup of the benchmark in JUBE considers both the BFS and SSSP kernels

## Verification

The benchmark needs to run through successfully without any errors reported.

The benchmark has a built-in validation which is disabled for benchmark for performance reasons with `export SKIP_VALIDATION=1`. Should there be a need, the according environment variable can be removed / the corresponding JUBE YAML script line commented out (`- { name: env, _: "export SKIP_VALIDATION=1" }`).

## Results

The benchmark prints a summary of the performance measured during the execution. The relevant values are
`harmonic_mean_TEPS` for BFS and SSSP.

Note that if the candidate has a better implementation of the Graph500 benchmark available, it could be used for a
Graph500 submission, **but not as an input in this document**.

### JUBE

If using JUBE, one can generate a table containing all the results with

```
jube result -a  benchmark/jube/results
```

For better automated analysis, the table can be generated in CSV output as well with an additional `-s csv`.

## Commitment

Candidates are requested to provide six values, all of them in GTEPS:

* BFS harmonic mean on four Compute Node
* SSSP harmonic mean on four Compute Node
* BFS harmonic mean on sixteen Compute Nodes
* SSSP harmonic mean on sixteen Compute Nodes
* BFS harmonic mean on a full system, i.e., (almost) all Compute Nodes, as used for the Graph500 list submission. A scale factor larger than 30 is allowed. In case the candidate has an optimized version of the benchmark, that version could be used for a submission to the graph500 list, but in the context of this procurement, the code/algorithm to be used is the one provided here.
* SSSP harmonic mean on a full system, i.e., (almost) all Compute Nodes, as used for the Graph500 list submission. A scale factor larger than 30 is allowed. In case the candidate has an optimized version of the benchmark, that version could be used for a submission to the graph500 list, but in the context of this procurement, the code/algorithm to be used is the one provided here.
