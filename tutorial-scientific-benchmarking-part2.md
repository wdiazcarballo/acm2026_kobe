# Scientific Benchmarking Tutorial - Part 2
## STREAM Benchmark on Fugaku Supercomputer

**ACM Second Asian School on HPC and AI**
**Date:** February 1, 2026
**Instructor:** Jens Domke, Dr. rer. nat.
**Affiliation:** Supercomputing Performance Research Team, RIKEN R-CCS, Kobe, JP
**Contact:** jens.domke@riken.jp

---

## Table of Contents

1. [Objective](#objective)
2. [Understanding STREAM Benchmark](#understanding-stream-benchmark)
3. [Getting Started](#getting-started)
4. [Step-by-Step Optimization Journey](#step-by-step-optimization-journey)
5. [The Roofline Model](#the-roofline-model)
6. [Final Optimized Configuration](#final-optimized-configuration)
7. [Summary of Lessons Learned](#summary-of-lessons-learned)

---

## Objective

**Goal:** Determine the peak TRIAD bandwidth of the FX1000 A64FX CPU using the STREAM benchmark.

The STREAM benchmark measures sustainable memory bandwidth through four simple vector operations:

| Kernel | Operation | Description |
|--------|-----------|-------------|
| Copy   | `C[i] = A[i]` | Simple copy |
| Scale  | `B[i] = scalar * C[i]` | Multiply by scalar |
| Add    | `C[i] = A[i] + B[i]` | Vector addition |
| Triad  | `A[i] = B[i] + scalar*C[i]` | Multiply-add |

---

## Understanding STREAM Benchmark

### The Four Kernels

```fortran
Copy:   C[i] = A[i];           i=1..N
Scale:  B[i] = scalar * C[i];  i=1..N
Add:    C[i] = A[i] + B[i];    i=1..N
Triad:  A[i] = B[i] + scalar*C[i];  i=1..N
```

### Arithmetic Intensity of TRIAD

For the TRIAD operation `A[i] = B[i] + scalar*C[i]`:
- **Floating-point operations:** 2 per index (1 multiply + 1 add)
- **Bytes transferred:** 24 bytes per index (read B[i], read C[i], write A[i] = 3 x 8 bytes)
- **Arithmetic Intensity:** 2/24 = 1/12 = 0.083 flop/byte

---

## Getting Started

### Step 1: Navigate to Your Working Directory

```bash
cd /vol0300/data/hp250477/Students/<uxxxx_YourName>
```

**Explanation:** Change to your personal working directory on the Fugaku shared filesystem. Replace `<uxxxx_YourName>` with your actual user ID and name.

---

### Step 2: Request an Interactive Job

```bash
pjsub --interact -x PJM_LLIO_GFSCACHE=/vol0003:/vol0004 -g "hp250477" --mpi "max-proc-per-node=1" -L "rscgrp=excl_hp250477_2601-1" -L "elapse=02:00:00" --sparam "wait-time=600" --no-check-directory
```

**Explanation:** Submit an interactive job request to the Fugaku job scheduler. This command:
- `--interact` - Requests an interactive session (not batch)
- `-x PJM_LLIO_GFSCACHE=...` - Sets up filesystem caching for performance
- `-g "hp250477"` - Specifies the project group for accounting
- `--mpi "max-proc-per-node=1"` - Limits MPI processes to 1 per node
- `-L "rscgrp=..."` - Selects the resource group (node allocation)
- `-L "elapse=02:00:00"` - Requests 2 hours of wall-clock time
- `--sparam "wait-time=600"` - Waits up to 10 minutes for resources
- `--no-check-directory` - Skips directory validation

---

### Step 3: Copy the Benchmark Materials

```bash
cp -r /vol0300/data/hp250477/Materials/AI_and_HPC/SB .
```

**Explanation:** Recursively copy (`-r`) the STREAM Benchmark directory `SB` from the shared materials location to your current working directory.

---

### Step 4: Enter the Benchmark Directory

```bash
cd SB/
```

**Explanation:** Change into the newly copied STREAM Benchmark directory.

---

### Step 5: List the Files

```bash
ls -la
```

**Explanation:** List all files (`-a`) in long format (`-l`) to see what's included:
- `stream.F` - STREAM benchmark source code (Fortran77)
- `mysecond.c` - Timer function in C
- `Makefile` - Build configuration
- `roofline.gp` - Gnuplot script for roofline visualization
- `fugaku.data` - Data file for plotting
- `sweep.sh` - Parameter sweep script

---

### Step 6: View the Source Code

```bash
less stream.F
```

**Explanation:** Open the STREAM benchmark Fortran source code in a pager for viewing. Press `q` to exit, arrow keys to scroll.

---

### Step 7: Build the Benchmark

```bash
make -B
```

**Explanation:** Compile the benchmark using the Makefile. The `-B` flag forces a complete rebuild, ignoring timestamps (unconditionally remake all targets).

---

### Step 8: Run the Benchmark (First Attempt)

```bash
./stream
```

**Explanation:** Execute the compiled STREAM benchmark binary. This first run will likely show problems (array too small, showing "Infinity").

---

## Step-by-Step Optimization Journey

### Attempt 1: First Run Analysis

**Expected Output (PROBLEMATIC):**
```
Array size =         64
Copy:        Infinity      0.0000      0.0000      0.0000
Scale:      1073.7418      0.0000      0.0000      0.0000
Add:         Infinity      0.0000      0.0000      0.0000
Triad:       Infinity      0.0000      0.0000      0.0000
```

**Issues identified:**
- Array size too small (64 elements)
- Rate calculation showing "Infinity" (division by zero time)
- Timer granularity problems

---

### Step 9: Edit Source to Fix Array Size

```bash
vim stream.F
```

**Explanation:** Open the source file in the vim editor. Search for the array size parameter (look for `64` or `PARAMETER`) and change it to a much larger value.

**Recommended array size:** `N = 44739242` (approximately 1GB total memory for 3 arrays)

**How to calculate:** `N = 1024*1024*1024 / 3 / 8` where:
- 1GB total memory
- 3 arrays (A, B, C)
- 8 bytes per double precision element

---

### Step 10: Rebuild After Editing

```bash
make -B
```

**Explanation:** Recompile the benchmark with the new array size. The `-B` flag ensures all files are rebuilt.

---

### Step 11: Run with Larger Array

```bash
./stream
```

**Explanation:** Run the benchmark again. You should now see actual numbers instead of "Infinity", but performance will still be low (~14 GB/s).

---

### Attempt 2 Result: ~14 GB/s

This is far below the theoretical peak of 1024 GB/s. The ratio (1024/14 ≈ 70) is suspiciously close to the number of cores (48).

---

### Step 12: Check the Makefile for Parallelization

```bash
vim Makefile
```

**Explanation:** Open the Makefile to check if OpenMP parallelization is enabled. Look for flags like `-fopenmp` (GNU) or `-Kopenmp` (Fujitsu).

---

### Step 13: Rebuild with OpenMP (if needed)

```bash
make -B
```

**Explanation:** Rebuild the benchmark after adding OpenMP flags to the Makefile.

---

### Step 14: Run with 48 OpenMP Threads

```bash
OMP_NUM_THREADS=48 ./stream
```

**Explanation:** Run the benchmark with the `OMP_NUM_THREADS` environment variable set to 48 (the number of cores on A64FX). This enables all cores to participate in the parallel computation.

---

### Attempt 4 Result: ~64 GB/s

Better, but only 5x speedup with 48 cores. The GNU compiler may not be optimal for A64FX.

---

### Step 15: Switch to Fujitsu Compiler

```bash
vim Makefile
```

**Explanation:** Edit the Makefile to change from GNU compilers (`gcc`/`gfortran`) to Fujitsu compilers (`fcc`/`frt`). Change lines like:
- `CC = gcc` → `CC = fcc`
- `FC = gfortran` → `FC = frt`
- `-fopenmp` → `-Kopenmp`

---

### Step 16: Rebuild with Fujitsu Compiler

```bash
make -B
```

**Explanation:** Recompile using the Fujitsu compiler, which is specifically optimized for the A64FX processor.

---

### Step 17: Run with Fujitsu-Compiled Binary

```bash
OMP_NUM_THREADS=48 ./stream
```

**Explanation:** Run the benchmark compiled with the Fujitsu compiler.

---

### Attempt 5 Result: ~91 GB/s

Improvement, but still only ~9% of peak. The issue is likely memory placement (NUMA/first-touch).

---

### Step 18: Fix First-Touch Memory Initialization

```bash
vim stream.F
```

**Explanation:** Edit the source to ensure arrays are initialized in parallel. In NUMA systems like A64FX (with 4 HBM modules), memory is placed near the core that first "touches" (writes to) it. Serial initialization puts all memory on one HBM module.

Add `!$OMP PARALLEL DO` directives around the array initialization loops.

---

### Step 19: Rebuild with First-Touch Fix

```bash
make -B
```

**Explanation:** Recompile after adding parallel initialization.

---

### Step 20: Run with First-Touch Fix

```bash
OMP_NUM_THREADS=48 ./stream
```

**Explanation:** Run the benchmark. If you see `**********` (asterisks), there's a print format overflow - proceed to next fix.

---

### Step 21: Fix Print Format and Iterations

```bash
vim stream.F
```

**Explanation:** Edit the source to:
1. Increase the `NTIMES` parameter (number of iterations) for more accurate timing
2. Fix the print format specifier to handle larger numbers (e.g., change `F10.4` to `F12.4`)

---

### Step 22: Rebuild with Format Fix

```bash
make -B
```

**Explanation:** Recompile after fixing the print format.

---

### Step 23: Run with All Fixes So Far

```bash
OMP_NUM_THREADS=48 ./stream
```

**Explanation:** Run the benchmark with array size, OpenMP, Fujitsu compiler, first-touch, and format fixes applied.

---

### Attempt 7 Result: ~631 GB/s

Significant improvement - now at 62% of peak!

---

### Step 24: Add Prefetch and Zfill Compiler Flags

```bash
vim Makefile
```

**Explanation:** Add Fujitsu-specific optimization flags for memory access patterns:
- `-Kprefetch_sequential=soft` - Enable software prefetching for sequential access
- `-Kprefetch_line=12` - Set prefetch distance (cache lines ahead)
- `-Kzfill=14` - Enable zero-fill optimization for write-only memory

---

### Step 25: Rebuild with Prefetch Flags

```bash
make -B
```

**Explanation:** Recompile with the new prefetch and zfill optimization flags.

---

### Step 26: Run with On-Demand Paging

```bash
OMP_NUM_THREADS=48 XOS_MMM_L_PAGING_POLICY=demand:demand:demand ./stream
```

**Explanation:** Run the benchmark with Fujitsu's paging policy set to "demand" for all memory types. This works better with first-touch initialization than "prepage" mode.

---

### Attempt 9 Result: ~739 GB/s

Now at 72% of peak. Can we do better with parameter tuning?

---

### Step 27: View the Sweep Script

```bash
less sweep.sh
```

**Explanation:** View the parameter sweep script that will test different combinations of `ZFILL` and `PREF` values.

---

### Step 28: Edit the Sweep Script

```bash
vim sweep.sh
```

**Explanation:** Configure the sweep ranges. Both `ZFILL` and `PREF` can be 1-100, but testing all 10,000 combinations is impractical. Test values around the initial guesses (8-18 range).

---

### Step 29: Run the Parameter Sweep

```bash
bash ./sweep.sh
```

**Explanation:** Execute the sweep script to find optimal `ZFILL` and `PREF` values. This will compile and run multiple times with different flag combinations.

**Sample Output:**
```
16 12 Triad: 843370.7042 0.0013 0.0013 0.0013  <-- Best!
```

---

### Step 30: Final Build with Optimal Parameters

```bash
ZFILL=16 PREF=12 make -B
```

**Explanation:** Rebuild with the optimal `ZFILL=16` and `PREF=12` values found by the sweep.

---

### Step 31: Final Optimized Run

```bash
OMP_NUM_THREADS=48 XOS_MMM_L_PAGING_POLICY=demand:demand:demand ./stream
```

**Explanation:** Run the final optimized benchmark.

---

## Final Optimized Configuration

### Final Results

```
STREAMy Version $ Revision: 3.14 $
----------------------------------------------
Array size =    44739242
The total memory requirement is 1023 MB
----------------------------------------------
Number of Threads =       48
----------------------------------------------
Function     Rate (MB/s)  Avg time   Min time  Max time
Copy:        774612.9275   0.0009     0.0009     0.0009
Scale:       771428.4961   0.0009     0.0009     0.0009
Add:         838190.8729   0.0013     0.0013     0.0013
Triad:       843528.6683   0.0013     0.0013     0.0013
----------------------------------------------
Solution Validates!
```

**Final TRIAD bandwidth: ~845 GB/s (82% of theoretical peak)**

---

## The Roofline Model

### Overview

The Roofline Model is a throughput-oriented performance model that:
- Tracks rates, not times
- Uses Little's Law: `concurrency = latency * bandwidth`
- Is architecture-independent (applies to CPUs, GPUs, TPUs, etc.)

### Three Components

1. **Machine Characterization** - Realistic performance potential of the system
2. **Application Execution Monitoring** - Actual measured performance
3. **Theoretical Application Bounds** - How well could the app perform ideally

### Roofline Equation

```
Attainable Gflop/s <= min(Peak Gflop/s, AI * Peak GB/s)
```

Where:
- **AI** = Arithmetic Intensity (flop/byte)
- **Peak Gflop/s** = Theoretical compute ceiling
- **Peak GB/s** = Theoretical memory bandwidth ceiling

### Fugaku A64FX Specifications

| Parameter | Value |
|-----------|-------|
| Theoretical Peak FP64 | ~3.07 TFLOP/s |
| Theoretical HBM2 Bandwidth | ~1024 GB/s |
| Number of Cores | 48 |
| HBM2 Modules | 4 |

---

### Step 32: Copy Roofline Files (On Local System)

```bash
scp <fugaku>:SB/roofline.gp .
```

**Explanation:** Secure copy the gnuplot script from Fugaku to your local machine for visualization.

---

### Step 33: Copy Data File (On Local System)

```bash
scp <fugaku>:SB/fugaku.data .
```

**Explanation:** Secure copy the data file containing your benchmark results.

---

### Step 34: Generate Roofline Plot (On Local System)

```bash
gnuplot roofline.gp
```

**Explanation:** Run gnuplot to generate the roofline visualization showing where your TRIAD results fall relative to the theoretical peak.

---

## Summary of Lessons Learned

### Common Pitfalls in Benchmarking

| Issue | Symptom | Solution |
|-------|---------|----------|
| Array too small | "Infinity" rate | Increase N to exceed cache size |
| No parallelization | Single-core performance | Enable OpenMP, set OMP_NUM_THREADS |
| Wrong compiler | Suboptimal code | Use vendor compiler (fcc/frt) |
| First-touch issue | Using only 1 HBM | Parallelize array initialization |
| Print overflow | `**********` output | Fix format specifiers |
| Wrong paging policy | Performance regression | Use demand paging with first-touch |
| Suboptimal prefetch | ~70% of peak | Tune -Kprefetch_line and -Kzfill |

### Key Optimization Steps for Fugaku

1. **Array Size:** Use ~1GB total (N ≈ 44 million elements)
2. **Compiler:** Use Fujitsu fcc/frt instead of GNU
3. **OpenMP:** Enable with `-Kopenmp` and `OMP_NUM_THREADS=48`
4. **First-Touch:** Initialize arrays in parallel
5. **Paging Policy:** `XOS_MMM_L_PAGING_POLICY=demand:demand:demand`
6. **Prefetch Flags:** `-Kprefetch_sequential=soft -Kprefetch_line=12 -Kzfill=16`

### Performance Summary

| Attempt | Configuration | Triad (GB/s) | % of Peak |
|---------|--------------|--------------|-----------|
| 1 | Default (N=64) | Infinity | - |
| 2 | N=44M, serial | 14 | 1.4% |
| 4 | + OpenMP 48 threads | 64 | 6.3% |
| 5 | + Fujitsu compiler | 91 | 8.9% |
| 7 | + First-touch + fixes | 631 | 62% |
| 9 | + Prefetch/zfill | 739 | 72% |
| 10 | + Tuned parameters | 845 | 82% |

**Total speedup achieved: ~50x from initial naive run!**

---

## References

- STREAM Benchmark: https://www.cs.virginia.edu/stream/
- Roofline Model: https://crd.lbl.gov/departments/computer-science/PAR/research/roofline/
- Fugaku User Guide: https://www.fugaku.r-ccs.riken.jp/

---

*Tutorial created for ACM Second Asian School on HPC and AI, February 2026*
