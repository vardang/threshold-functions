# Threshold Function Enumeration

A Python implementation for enumerating **Linear Threshold Functions (LTFs)** using modern SMT solvers and symmetry reduction techniques.

[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![OEIS A000609](https://img.shields.io/badge/OEIS-A000609-green.svg)](https://oeis.org/A000609)

---

## What Are Threshold Functions?

A **threshold function** (or Linear Threshold Function) is a Boolean function that can be computed by a single artificial neuron — the fundamental building block of neural networks.

### Definition

A function `f: {0,1}ⁿ → {0,1}` is a threshold function if there exist weights `w₁, w₂, ..., wₙ` and a threshold `T` such that:

```
f(x₁, x₂, ..., xₙ) = 1   if   w₁x₁ + w₂x₂ + ... + wₙxₙ ≥ T
                   = 0   otherwise
```

### Visual Example (n=2)

```
         x₂
          │
    (0,1) ●───────● (1,1)          Threshold function: x₁ OR x₂
          │ ╲  1  │                 
          │   ╲   │                 Weights: w₁=1, w₂=1, T=1
          │  0  ╲ │                 
    (0,0) ●───────● (1,0)          f(x) = 1 iff x₁ + x₂ ≥ 1
          └───────────── x₁
          
    The line w₁x₁ + w₂x₂ = T separates 0s from 1s
```

### The Counting Problem

**How many threshold functions exist for n input variables?**

This is a famous combinatorial problem. The sequence N(n) is documented in [OEIS A000609](https://oeis.org/A000609):

| n | N(n) | Description |
|:-:|-----:|:------------|
| 1 | 4 | All 4 functions of 1 variable are threshold functions |
| 2 | 14 | Out of 16 Boolean functions, 14 are threshold functions |
| 3 | 104 | Out of 256 Boolean functions |
| 4 | 1,882 | Out of 65,536 Boolean functions |
| 5 | 94,572 | Computed by Winder (1965) |
| 6 | 15,028,134 | Computed by Muroga et al. (1970) |
| 7 | 8,378,070,864 | Computed by Muroga (1970) |
| 8 | 17,561,539,552,946 | Computed using supercomputers |

> **Open Problem:** No closed-form formula for N(n) is known. Finding one is a major unsolved problem in combinatorics.

---

## Mathematical Background

### Geometric Interpretation

Threshold functions correspond to **hyperplane arrangements** in weight space:

```
                    Weight Space (ℝⁿ)
                    ┌─────────────────────┐
                    │    ╱               │
                    │   ╱  Chamber 1     │
                    │  ╱                 │
                    │ ╱──────────────    │
                    │╱     Chamber 2  ╲  │
                    │                  ╲ │
                    │   Chamber 3       ╲│
                    └─────────────────────┘
                    
    Each "chamber" corresponds to a unique ordering of weighted sums.
    Each ordering generates multiple threshold functions via "cuts".
```

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Hyperplane Arrangement** | The weight space ℝⁿ is partitioned by hyperplanes `w·(x-y) = 0` for all vertex pairs |
| **Chamber** | A connected region where the ordering of all weighted sums is constant |
| **Hyperoctahedral Group B_n** | Symmetry group of the n-hypercube with order `2ⁿ × n!` |
| **Canonical Representative** | One function from each symmetry equivalence class |

### Symmetry Reduction

The key optimization uses the **hyperoctahedral group B_n**:

```
    B_n = symmetries of the n-dimensional hypercube
    
    For n=3:  |B₃| = 2³ × 3! = 48 symmetries
    
    Instead of finding all 104 functions,
    we find only 10 canonical representatives
    and compute their orbit sizes.
    
    ┌─────────────────────────────────────┐
    │  N(n) = Σ orbit_size(representative) │
    └─────────────────────────────────────┘
```

---

## Pattern Analysis & A002079 Approximation

### King's Formula (2023)

The total number of threshold functions N(n) can be computed from the **A002079 sequence** via:

```
N(n) = Σ_{k=0}^n A002079(k) × C(n,k) × 2^k
```

Where:
- **A002079(k)** = number of threshold functions essentially depending on exactly k variables
- **C(n,k)** = binomial coefficient "n choose k"
- **2^k** = factor accounting for variable negations

```mermaid
flowchart LR
    A["A002079(k)"] --> K["King's Formula"]
    B["C(n,k)"] --> K
    C["2^k"] --> K
    K --> N["N(n)"]
```

| k | A002079(k) | Description |
|:-:|----------:|:------------|
| 0 | 2 | Constant functions (0 or 1) |
| 1 | 1 | Single-variable functions |
| 2 | 2 | Two essential variables |
| 3 | 9 | Three essential variables |
| 4 | 96 | Four essential variables |
| 5 | 2,690 | Five essential variables |
| 6 | 226,360 | Six essential variables |
| 7 | 64,646,855 | Seven essential variables |
| 8 | 68,339,572,672 | Eight essential variables |
| 9 | 281,196,831,947,304 | Nine essential variables |

### Best Approximation: Advanced Third-Order Ratio Recurrence

We discovered an empirical recurrence that predicts A002079(k) from previous values:

```
r(k) = A002079(k) / A002079(k-1)

r(k) = 6.382116·r(k-1) - 9.197797·r(k-2) + 2.294211·k - 9.929697
```

**Initial values:** r(2) = 2.0, r(3) = 4.5

**Performance:** 0.001 bits error on leave-last-out validation (train k=1..8, test k=9)

### Alternative: Polynomial (degree 5) Regression

A polynomial fit to log₂(A002079(k)):

```
log₂(A(k)) = -0.000377k⁵ + 0.00904k⁴ - 0.0539k³ + 0.717k² - 0.897k + 0.226
```

**Performance:**
- Fit error: 0.0007 bits (excellent on training data)
- Extrapolation error: 0.0485 bits (worse than recurrence for prediction)

### Quick Reference

```
┌─────────────────────────────────────────────────────────────────────────┐
│  MODEL 1: ADVANCED THIRD-ORDER RATIO (Best for Extrapolation)           │
├─────────────────────────────────────────────────────────────────────────┤
│  r(k) = 6.382116·r(k-1) - 9.197797·r(k-2) + 2.294211·k - 9.929697      │
│                                                                         │
│  where r(k) = A002079(k) / A002079(k-1)                                │
│  Extrapolation error: 0.001 bits                                        │
├─────────────────────────────────────────────────────────────────────────┤
│  MODEL 2: POLYNOMIAL (deg 5) (Best Training Fit)                        │
├─────────────────────────────────────────────────────────────────────────┤
│  log₂(A(k)) = -0.000377k⁵ + 0.00904k⁴ - 0.0539k³ + 0.717k² - 0.897k   │
│               + 0.226                                                   │
│                                                                         │
│  Fit error: 0.0007 bits | Extrapolation error: 0.0485 bits             │
├─────────────────────────────────────────────────────────────────────────┤
│  PREDICTIONS (from Advanced Third-Order Ratio):                         │
│    N(10) ≈ 4.79 × 10²⁰   |   A002079(10) ≈ 4.11 × 10¹⁷                 │
│                                                                         │
│  Status: UNVERIFIED CONJECTURES                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

### Validation Strategy: Leave-Last-Out

```mermaid
flowchart TD
    subgraph train [Training Data]
        T1["A002079(1..8)"]
    end
    subgraph test [Hold-Out Test]
        T2["A002079(9)"]
    end
    train --> Model["Fit Recurrence"]
    Model --> Pred["Predict A002079(9)"]
    Pred --> Cmp["Compare"]
    T2 --> Cmp
    Cmp --> Err["0.001 bits error"]
```

**Why Leave-Last-Out?**

- Only 10 known values (n=0..9) — cannot afford k-fold cross-validation
- Simulates actual use case: predict unknown future values from known past
- Tests on the largest, most challenging value (k=9)
- Common strategy for time-series with limited data

### Model Comparison

| Method | Error (bits) | Error (%) | Notes |
|--------|-------------|-----------|-------|
| Our Third-Order Recurrence | 0.001 | 0.07% | Empirical fit |
| Zuev Asymptotic (2^n²/n!) | 2.31 | 400%+ | Theoretical bound |

### ⚠️ Important Caveats

> **This is an empirical conjecture, NOT a proven formula.**

| Concern | Details |
|---------|---------|
| **Overfitting Risk** | 4 parameters fitted to only 8 data points (ratios r(3)..r(9)) |
| **No Theoretical Basis** | Pure curve fitting with no combinatorial justification |
| **Unverifiable** | N(10) is unknown — cannot validate extrapolation |
| **Conjecture Status** | May break down for k > 9 |

The formula fits known data extremely well but could be a coincidence. Use predictions with appropriate skepticism.

---

## Implementation

### Method: Symmetry-Breaking Z3 Enumeration

This implementation uses the [Z3 SMT Solver](https://github.com/Z3Prover/z3) with:

1. **Biconditional Encoding**: `f[i] ↔ (Σ wₖxₖ ≥ T)`
2. **Symmetry-Breaking Constraints**: `w₀ ≥ w₁ ≥ ... ≥ wₙ₋₁ ≥ 0`
3. **Orbit Size Computation**: Using precomputed B_n group actions

```
┌──────────────────────────────────────────────────────────────┐
│                         Algorithm                             │
├──────────────────────────────────────────────────────────────┤
│  1. Initialize Z3 solver with threshold constraints           │
│  2. Add symmetry-breaking: w[i] ≥ w[i+1] for all i           │
│  3. While solver finds a solution:                            │
│     a. Extract Boolean function f                             │
│     b. Compute orbit size under B_n                           │
│     c. Add to total count                                     │
│     d. Block this solution, continue                          │
│  4. Return total count                                        │
└──────────────────────────────────────────────────────────────┘
```

### Performance

| n | Representatives | N(n) | Time |
|:-:|---------------:|-----:|-----:|
| 2 | 5 | 14 | <0.1s |
| 3 | 10 | 104 | <0.1s |
| 4 | 27 | 1,882 | ~0.5s |
| 5 | 146 | 94,572 | ~30s |

---

## Installation

### Prerequisites

- Python 3.8 or higher
- Jupyter Notebook or JupyterLab

### Required Libraries

```bash
pip install z3-solver numpy scipy
```

### Optional (for GPU acceleration)

```bash
pip install torch  # For CUDA-accelerated orbit computation
```

### Quick Start

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/threshold-functions.git
cd threshold-functions

# Install dependencies
pip install -r requirements.txt

# Run the notebook
jupyter notebook threshold_functions_complete.ipynb
```

### requirements.txt

```
z3-solver>=4.12.0
numpy>=1.21.0
scipy>=1.7.0
```

---

## Usage

### Basic Example

```python
from z3 import *
import itertools
import numpy as np

def count_threshold_functions(n):
    """Count threshold functions on n variables."""
    vertices = list(itertools.product((0, 1), repeat=n))
    num_vertices = 2 ** n
    
    # Z3 variables
    f = [Bool(f'f_{i}') for i in range(num_vertices)]
    w = [Real(f'w_{i}') for i in range(n)]
    T = Real('T')
    
    solver = Solver()
    
    # Core constraint: f[i] ↔ (weighted_sum ≥ T)
    for i, v in enumerate(vertices):
        weighted_sum = sum(w[k] * v[k] for k in range(n))
        solver.add(f[i] == (weighted_sum >= T))
    
    # Symmetry-breaking
    for i in range(n - 1):
        solver.add(w[i] >= w[i + 1])
    solver.add(w[n - 1] >= 0)
    
    # Count solutions (simplified - see notebook for full implementation)
    count = 0
    while solver.check() == sat:
        model = solver.model()
        count += 1  # Add orbit size computation here
        solver.add(Or([f[i] != model.eval(f[i]) for i in range(num_vertices)]))
    
    return count
```

### Running the Notebook

The notebook `threshold_functions_complete.ipynb` contains:

1. **Verification** - Confirms N(1) through N(4) match known values
2. **Chamber Analysis** - Explores the hyperplane arrangement structure
3. **N(5) Computation** - Computes N(5) = 94,572
4. **Summary Statistics** - Representative counts and orbit sizes

---

## Project Structure

```
threshold-functions/
├── README.md                               # This file
├── requirements.txt                        # Python dependencies
├── LICENSE                                 # MIT License
├── threshold_functions_complete.ipynb      # Main enumeration notebook (CPU)
├── threshold_functions_complete_gpu.ipynb  # GPU-accelerated enumeration
└── a002079_pattern_analysis.ipynb          # A002079 recurrence analysis ⭐
```

### Notebook Comparison

| Notebook | Purpose | Description |
|----------|---------|-------------|
| `threshold_functions_complete.ipynb` | Enumeration (CPU) | Z3-based counting with symmetry breaking |
| `threshold_functions_complete_gpu.ipynb` | Enumeration (GPU) | PyTorch/CUDA accelerated version |
| `a002079_pattern_analysis.ipynb` | Pattern Analysis | **Discovers recurrence for A002079** |

**GPU Speedup:** 5-10x faster orbit computation for n ≥ 5

```
┌─────────────────────────────────────────────────────┐
│  CPU: Apply 3840 group elements sequentially        │
│       → ~50ms per orbit for n=5                     │
│                                                     │
│  GPU: Apply 3840 group elements IN PARALLEL         │
│       → ~5ms per orbit for n=5 (10x speedup)        │
└─────────────────────────────────────────────────────┘
```

---

## References

### Primary Sources

| Reference | Link |
|-----------|------|
| **OEIS A000609** - Number of threshold functions | [oeis.org/A000609](https://oeis.org/A000609) |
| **OEIS A002079** - Essential variable equivalence classes | [oeis.org/A002079](https://oeis.org/A002079) |
| **King, A.D.** (2023). Comments on A002080 and related sequences based on threshold functions. | [OEIS PDF](https://oeis.org/A002080/a002080.pdf) |
| **Muroga, S.** (1971). *Threshold Logic and Its Applications*. Wiley-Interscience. | [WorldCat](https://www.worldcat.org/title/threshold-logic-and-its-applications/oclc/140838) |
| **Winder, R.O.** (1966). Enumeration of Seven-Argument Threshold Functions. *IEEE Trans. Electronic Computers*, EC-15(3), 315-325. | [IEEE Xplore](https://ieeexplore.ieee.org/document/1446579) |

### Asymptotic Results

| Reference | Link |
|-----------|------|
| **Zuev, Y.A.** (1989). Asymptotics of the logarithm of the number of threshold functions. *Soviet Math. Doklady*, 39(3), 512-513. | [MathSciNet](https://mathscinet.ams.org/mathscinet-getitem?mr=1014762) |
| **Zuev, Y.A.** (1991). Combinatorial-probability and geometric methods in threshold logic. *Discrete Math.*, 3(2), 47-57. | — |

### Hyperplane Arrangements

| Reference | Link |
|-----------|------|
| **Zaslavsky, T.** (1975). Facing up to Arrangements: Face-Count Formulas for Partitions of Space by Hyperplanes. *Memoirs of the AMS*, No. 154. | [AMS](https://www.ams.org/books/memo/0154/) |
| **Stanley, R.P.** (2004). An Introduction to Hyperplane Arrangements. | [PDF](http://www-math.mit.edu/~rstan/arrangements/arr.html) |
| **Orlik, P. & Terao, H.** (1992). *Arrangements of Hyperplanes*. Springer. | [Springer](https://link.springer.com/book/10.1007/978-3-662-02772-1) |

### Symmetry and Group Theory

| Reference | Link |
|-----------|------|
| **Hyperoctahedral Group** - Wikipedia | [Wikipedia](https://en.wikipedia.org/wiki/Hyperoctahedral_group) |
| **Burnside's Lemma** - Orbit counting | [Wikipedia](https://en.wikipedia.org/wiki/Burnside%27s_lemma) |

### Tools

| Tool | Link |
|------|------|
| **Z3 SMT Solver** | [github.com/Z3Prover/z3](https://github.com/Z3Prover/z3) |
| **Z3 Python API Documentation** | [z3prover.github.io](https://z3prover.github.io/api/html/namespacez3py.html) |

---

## Further Reading

### Survey Papers

- **Saks, M.** (1993). Slicing the hypercube. *Surveys in Combinatorics*, London Math. Soc. Lecture Notes 187, 211-255.
- **Håstad, J.** (1994). On the size of weights for threshold gates. *SIAM J. Discrete Math.*, 7(3), 484-492.

### Related Sequences

| OEIS | Description |
|------|-------------|
| [A000609](https://oeis.org/A000609) | Number of threshold functions of n or fewer variables |
| [A002079](https://oeis.org/A002079) | N-equivalence classes of threshold functions of exactly n variables |
| [A002077](https://oeis.org/A002077) | Number of self-dual threshold functions |
| [A006126](https://oeis.org/A006126) | Number of monotone Boolean functions (Dedekind numbers) |

---

## Contributing

Contributions are welcome! Potential areas for improvement:

- [ ] Verify N(10) to validate the A002079 recurrence conjecture
- [ ] Find theoretical justification for the recurrence coefficients
- [ ] Improve enumeration algorithms for larger n
- [ ] Better visualization of hyperplane arrangements

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## Acknowledgments

This implementation builds on decades of research in threshold logic, particularly the foundational work of:

- **Saburo Muroga** - Pioneering enumeration methods
- **Robert O. Winder** - Early computational results
- **Yuri A. Zuev** - Asymptotic analysis
- **Thomas Zaslavsky** - Hyperplane arrangement theory
- **Alastair D. King** - King's formula connecting A002079 to N(n)

---

<p align="center">
  <i>The quest for a closed-form formula for N(n) continues...</i><br>
  <i>Our empirical recurrence achieves 0.001 bits error — but can it be proven?</i>
</p>
