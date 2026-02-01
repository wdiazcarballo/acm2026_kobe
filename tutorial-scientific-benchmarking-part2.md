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
2. [Getting Started](#getting-started)
3. [Understanding STREAM Benchmark](#understanding-stream-benchmark)
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

## Getting Started

### Step 1: Request an Interactive Job

```bash
# Navigate to your working directory
cd /vol0300/data/hp250477/Students/<uxxxx_YourName>

# Request an interactive job on Fugaku
pjsub --interact \
  -x PJM_LLIO_GFSCACHE=/vol0003:/vol0004 \
  -g "hp250477" \
  --mpi "max-proc-per-node=1" \
  -L "rscgrp=excl_hp250477_2601-1" \
  -L "elapse=02:00:00" \
  --sparam "wait-time=600" \
  --no-check-directory
```

### Step 2: Copy the Materials

```bash
# Copy the benchmark materials
cp -r /vol0300/data/hp250477/Materials/AI_and_HPC/SB .

# Navigate to the benchmark directory
cd SB/

# List the files
ls -la
```

**Files included:**
- `stream.F` - STREAM benchmark source code (Fortran77)
- `mysecond.c` - Timer function
- `Makefile` - Build configuration
- `roofline.gp` - Gnuplot script for roofline visualization
- `fugaku.data` - Data file for plotting
- `sweep.sh` - Parameter sweep script

### Step 3: Initial Build and Test

```bash
# View the source code
less stream.F

# Build the benchmark
make -B

# Run the benchmark
./stream
```

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

## Step-by-Step Optimization Journey

### Attempt 1: First Run (FAILED)

**Problem:** Array size too small (64 elements)

```
Array size =         64
Copy:        Infinity      0.0000      0.0000      0.0000
Scale:      1073.7418      0.0000      0.0000      0.0000
Add:         Infinity      0.0000      0.0000      0.0000
Triad:       Infinity      0.0000      0.0000      0.0000
```

**Issues identified:**
- Array size too small
- Rate calculation showing "Infinity"
- Timer granularity problems

### Attempt 2: Fix Array Size

**Step 4:** Edit `stream.F` to find and modify the array size parameter.

```bash
vim stream.F
# Look for array size definition and change to a larger value
```

**Recommended array size calculation:**
- For ~1GB memory: `N = 1024*1024*1024/3/8 ≈ 44739242`
- Must be large enough to exceed all cache levels

**Step 5-6:** Recompile and run

```bash
make -B
./stream
```

**Result:** ~14 GB/s (still far from peak of 1024 GB/s)

---

### Attempt 3: Analyze with Roofline Model

**Step 7:** Calculate Arithmetic Intensity for TRIAD

**Step 8:** Plot the roofline (on local system with gnuplot)

```bash
# Copy files to local system
scp <fugaku>:SB/roofline.gp .
scp <fugaku>:SB/fugaku.data .

# Edit the data files with your measurements
# Generate the plot
gnuplot roofline.gp
```

**Observation:** Performance is ~70x below peak. Fugaku has 48 cores...

---

### Attempt 4: Enable OpenMP Parallelization

**Step 9-10:** Check for parallelization issues

```bash
vim Makefile
# Look for OpenMP flags (-fopenmp or similar)
```

**Step 11:** Recompile with OpenMP and run with 48 threads

```bash
make -B
OMP_NUM_THREADS=48 ./stream
```

**Result:** ~64 GB/s (only 5x speedup with 48 cores - still not great)

---

### Attempt 5: Switch to Fujitsu Compiler

**Step 12-13:** The GNU compiler may not be optimized for A64FX

```bash
vim Makefile
# Change compiler from gcc/gfortran to fcc/frt (Fujitsu compilers)
```

**Step 14:** Recompile and run

```bash
make -B
OMP_NUM_THREADS=48 ./stream
```

**Result:** ~91 GB/s (improvement, but still only ~9% of peak)

---

### Attempt 6: Fix First-Touch Memory Policy

**Step 15-16:** Memory may be allocated on only one NUMA node

The A64FX has 4 HBM2 memory modules. Without proper first-touch initialization, all memory might be allocated to a single module.

```bash
vim stream.F
# Add OpenMP parallel initialization for arrays
```

**Step 17:** Recompile and run

```bash
make -B
OMP_NUM_THREADS=48 ./stream
```

**Result:** Shows `**********` (overflow) - something went wrong!

---

### Attempt 7: Fix Iteration Count and Print Format

**Step 18-19:** Check for iteration count and print format issues

```bash
vim stream.F
# Check NTIMES parameter (iterations)
# Check print format specifiers
```

**Step 20:** Recompile and run

```bash
make -B
OMP_NUM_THREADS=48 ./stream
```

**Result:** ~631 GB/s (significant improvement - 60% of peak)

---

### Attempt 8: Try Prepage Memory Policy (FAILED)

**Step 21-22:** Experiment with Fugaku's page allocation

```bash
make -B
OMP_NUM_THREADS=48 \
XOS_MMM_L_PAGING_POLICY=prepage:prepage:prepage \
./stream
```

**Result:** Back to ~91 GB/s - prepage broke first-touch!

---

### Attempt 9: Add Compiler Optimization Flags

**Step 24-25:** Add prefetch and zfill flags

```bash
vim Makefile
# Add compiler flags:
# -Kprefetch_sequential=soft
# -Kprefetch_line=12
# -Kzfill=14
```

**Step 26:** Recompile and run with on-demand paging

```bash
make -B
OMP_NUM_THREADS=48 \
XOS_MMM_L_PAGING_POLICY=demand:demand:demand \
./stream
```

**Result:** ~739 GB/s (72% of peak)

---

### Attempt 10: Parameter Sweep for Optimal Flags

**Step 27-28:** Perform a pruned full factorial experiment

```bash
vim sweep.sh
# Configure parameter ranges for ZFILL and PREF
# Both flags can be 1-100, so we test a subset around known good values
```

**Step 29:** Run the sweep

```bash
bash ./sweep.sh
```

**Sample output:**
```
14 18 Triad: 595556.6729 0.0018 ...
16  8 Triad: 830462.7624 0.0013 ...
16 10 Triad: 840067.0696 0.0013 ...
16 12 Triad: 843370.7042 0.0013 ...  <-- Best!
16 14 Triad: 731341.2732 0.0015 ...
```

**Best configuration found:** `ZFILL=16, PREF=12`

---

## Final Optimized Configuration

### Final Build and Run

```bash
# Build with optimal flags
ZFILL=16 PREF=12 make -B

# Run with optimal environment
OMP_NUM_THREADS=48 \
XOS_MMM_L_PAGING_POLICY=demand:demand:demand \
./stream
```

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

## Quick Reference: Final Working Commands

```bash
# 1. Request interactive job
pjsub --interact -x PJM_LLIO_GFSCACHE=/vol0003:/vol0004 \
  -g "hp250477" --mpi "max-proc-per-node=1" \
  -L "rscgrp=excl_hp250477_2601-1" -L "elapse=02:00:00" \
  --sparam "wait-time=600" --no-check-directory

# 2. Copy and enter directory
cp -r /vol0300/data/hp250477/Materials/AI_and_HPC/SB .
cd SB/

# 3. Build with optimal flags
ZFILL=16 PREF=12 make -B

# 4. Run with optimal environment
OMP_NUM_THREADS=48 \
XOS_MMM_L_PAGING_POLICY=demand:demand:demand \
./stream
```

---

## References

- STREAM Benchmark: https://www.cs.virginia.edu/stream/
- Roofline Model: https://crd.lbl.gov/departments/computer-science/PAR/research/roofline/
- Fugaku User Guide: https://www.fugaku.r-ccs.riken.jp/

---

*Tutorial created for ACM Second Asian School on HPC and AI, February 2026*
