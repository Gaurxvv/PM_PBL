# Classical Pattern Matching Benchmark Framework

A modular, deterministic benchmarking framework for evaluating classical string-matching algorithms across **matching accuracy**, **wall-clock execution latency**, **hardware-independent operation counts**, and **asymptotic input-size scaling**.

## Overview

Wall-clock timing alone is vulnerable to OS scheduling jitter and instrumentation overhead. This framework isolates algorithmic behavior from runtime noise using a **split-execution architecture**:
1. **Uninstrumented Timing Mode:** Captures pure `preprocess()` and `search()` latency with garbage collection suspended and warm-up cycles completed.
2. **Instrumented Telemetry Mode:** Runs a separate verification pass to record exact character comparisons, shift counts, and index parity against a ground-truth oracle.

### Benchmarked Algorithms
* **Naïve (Brute Force):** Baseline sliding-window search ($O(NM)$ worst-case, $O(1)$ auxiliary space).
* **Knuth-Morris-Pratt (KMP):** Prefix-function (LPS) automaton guaranteeing linear $O(N + M)$ worst-case time without backtracking.
* **Boyer-Moore-Horspool (BMH):** Right-to-left character matching with bad-character skip tables achieving sublinear $O(N/M)$ average-case scans.
* **Rabin-Karp:** Rolling polynomial hash matching with exact character verification on hash collisions.

---

## Architecture & Evaluation Dimensions

### 1. Deterministic Workload Suite (`DatasetFactory`)
Generated with a fixed random seed (`seed=42`) to ensure reproducibility across hardware and Python versions:
* **Input-Size Scaling ($N$):** Exponentially scaled text lengths from $10^3$ to $5 \times 10^5$ characters ($M = 32$) for log-log power-law regression.
* **Pattern-Size Scaling ($M$):** Pattern lengths $M \in \{4, 16, 64, 256, 1024\}$ over a fixed $100\text{ KB}$ corpus to observe skip-distance gains.
* **Alphabet Density ($\vert{}\Sigma\vert{}$):** Low-cardinality DNA sequences (`ACGT`, $\vert{}\Sigma\vert{} = 4$) vs. printable ASCII ($\vert{}\Sigma\vert{} = 90$).
* **Adversarial & Edge Cases:** Periodic worst-case inputs ($T = \text{"a"}^N, P = \text{"a"}^{M-1}\text{"b"}$) and dense overlapping matches ($T = \text{"a"}^N, P = \text{"a"}^M$).

### 2. Core Metrics
| Metric | Description |
| :--- | :--- |
| **Index Parity (`is_accurate`)** | Asserts $I_{\text{algo}} == I_{\text{oracle}}$ across all exact, overlapping, and boundary occurrences. |
| **Preprocessing & Search Latency** | High-resolution `perf_counter_ns` measurements reporting median, P95 tail latency, and Interquartile Range (IQR). |
| **Comparison Efficiency ($\eta$)** | Ratio of character comparisons to text length ($\eta = \frac{\text{comparisons}}{N}$), distinguishing sublinear ($\eta < 1$) from quadratic ($\eta \approx M$) scans. |
| **Effective Throughput** | Normalized scan bandwidth in $\text{MB/s}$. |
| **Peak Auxiliary Memory** | Traced via `tracemalloc` during preprocessing to capture table/automaton footprint. |
| **Empirical Scaling Exponent ($b$)** | Ordinary Least Squares (OLS) slope of $\log(T_{\text{search}}) = a + b\log(N)$ with $R^2$ goodness-of-fit. |
