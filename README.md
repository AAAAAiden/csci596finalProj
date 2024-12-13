# Parallel Molecular Dynamics (PMD) Code with MPI+OpenMP

This repository presents an enhanced version of a Parallel Molecular Dynamics (PMD) simulation code for Lennard-Jones systems. The original code relied solely on MPI for parallelization. In this improved version, we introduce OpenMP for a hybrid MPI+OpenMP approach, potentially leveraging modern multi-core CPUs more effectively.

## The Problem in the Original Code

1. **Lack of Intra-Process Parallelism:**
   The original code used MPI for parallelization but did not exploit multiple CPU cores within a single node effectively. Each MPI process executed its portion of the simulation sequentially. As a result, if you run the code on a multi-core node with one MPI rank, you could not fully utilize all the available CPU cores on that node.

2. **Potential Fork-Join Overhead (If Naively Parallelized):**
   A common pitfall when adding OpenMP is to place the `#pragma omp parallel` directive inside the main MD loop. Doing so would repeatedly create and destroy threads at each time step, introducing significant overhead and potentially negating the benefits of parallelization. While not an issue in the original code itself (which had no OpenMP), this is a problem that could arise if OpenMP were introduced incorrectly.

3. **Load Imbalance Due to Static Domain Decomposition:**
   The original domain decomposition via MPI was static. If particles cluster unevenly, some MPI processes inevitably handle more atoms than others, causing those processes to become bottlenecks. Without additional techniques, this can severely limit scaling and performance as the simulation grows.

## Key Improvements in the Modified Code

1. **Hybrid Parallelization (MPI+OpenMP):**
   - **What’s New:**  
     We now run multiple OpenMP threads per MPI rank. This hybrid approach can help fully utilize multi-core architectures and reduce MPI communication overhead by having fewer MPI ranks per node.

2. **Single Parallel Region for the Time-Stepping Loop:**
   - **Benefit:**  
     By using one `#pragma omp parallel` region outside the main loop, we ensure threads are created once and persist throughout the simulation. This avoids the excessive overhead of repeated thread creation and destruction.

3. **Dynamic Work Scheduling with OpenMP:**
   - **How it Helps:**  
     Using `#pragma omp for schedule(dynamic)` distributes the workload among threads on-the-fly. If some steps are more computationally expensive than others, dynamic scheduling reduces idle time and improves load balance.

4. **Better Load Balancing Among Threads:**
   - **Strategy:**  
     By tuning domain subdivision parameters (`vthrd[]`, `thbk[]`) and experimenting with OpenMP schedules, we can ensure that each thread handles a roughly equal amount of work, mitigating load imbalance within a single MPI domain.

## Future Directions and Performance Analysis

1. **Measuring Performance Across Multiple Platforms:**
   Benchmark and profile the code on various HPC clusters and cloud platforms (e.g., Tiber). Identifying where most execution time is spent can guide further optimizations.

2. **Advanced Load Balancing Techniques:**
   Experiment with dynamic or adaptive domain decomposition for MPI ranks. Combining this with OpenMP’s dynamic scheduling could further improve scaling in highly non-uniform particle distributions.

3. **Optimizations and Tuning:**
   Consider applying vectorization, more efficient neighbor list constructions, or different data layouts (SoA vs. AoS) for additional performance gains.

## Getting Started

1. **Prerequisites:**
   - An MPI implementation (e.g., MPICH, OpenMPI)
   - A C compiler with OpenMP support (e.g., `gcc` with `-fopenmp`)

2. **Build the Code:**
   ```bash
   mpicc -fopenmp -o pmd pmd.c
