## engnieering-calculative-methods

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BUAA 工程计算方法 (Engineering Calculative Methods) 2026 Spring exam review repository. Contains reference code, lecture slides, knowledge summaries, and Excel experiment files across 10 chapters of numerical analysis. MIT-licensed, authored by Liu Chunyi.

## Build & Run

No build system — manual compilation and execution:

```bash
# C++ (requires C++11 or later for `>>` in nested templates)
g++ -std=c++11 文件名.cpp -o 输出名 && ./输出名

# Python (NumPy constrained to <2.0)
pip install -r requirements.txt
python3 Guass求积公式.py
```

Compiled binaries live in `Code/output/` (gitignored).

## Repository Structure

```
Code/       # Reference implementations (C++ .cpp + one Python .py)
PPT/        # Lecture slides (PDF, 14 files across 10 chapters)
Summary/    # Chapter knowledge summaries (Markdown with LaTeX, one folder per chapter)
Excel/      # Classroom experiment spreadsheets
```

`Summary/README.md` contains the canonical chapter-to-file mapping table linking each chapter's PPT, Code, and Excel files.

## Code Conventions

### Parameter Modification via Macros

Code files are designed for rapid exam reuse: modify `#define` macros and global constants at the top of each file, recompile, run. Do not refactor away this pattern — it's intentional for exam workflow.

### Sorting Algorithm Interface

All sort functions in `排序算法.cpp` share a common signature:

```cpp
void xxxSort(vector<ELEMTYPE>& arr, int left, int right)
// left:  inclusive start index
// right: inclusive end index
```

Element type is controlled by `typedef int ELEMTYPE;` at the top — change it to switch between `int`, `double`, etc.

### C++11 Requirement

Nested template syntax like `vector<vector<double>>` requires C++11 (right-angle bracket fix). Always compile with `-std=c++11` or later.

### Python: NumPy <2.0

`requirements.txt` pins `numpy>=1.21,<2.0`. NumPy 2.x introduces breaking changes; do not widen this constraint without verifying compatibility.

## Chapter → File Mapping

| Ch | Topic | Code Files |
|----|-------|------------|
| 1 | 数值误差 | `秦九韶法.cpp` |
| 2 | 非线性方程数值解法 | `非线性方程的数值解法.cpp` (bisection, Newton, secant, general iteration) |
| 3 | 线性方程组数值求解 | `线性方程组的数值求解.cpp` (Jacobi, Gauss-Seidel, SOR), `线性方程组的数值求解.py` (矩阵形式) |
| 4 | 非线性方程组数值求解 | `非线性方程组的数值解法.py` (Newton, steepest descent) |
| 5 | 插值方法 | `插值方法.cpp` (Lagrange, Newton, Hermite, cubic spline) |
| 6 | 函数数值拟合 | `函数的数值拟合.py` (least squares fitting) |
| 7 | 数值积分 | `数值积分方法.cpp`, `Guass求积公式.py` (Newton-Cotes, Romberg, Gauss) |
| 8 | 常微分方程数值解法 | `常微分方程的数值解法.cpp` (Euler, RK2/RK4, predictor-corrector) |
| 9 | 质量与可靠性统计 | `正态分布.cpp` (normal PDF/CDF) |
| 10 | 线性搜索与排序 | `线性搜索算法.cpp` (golden section), `排序算法.cpp` (7 sorting algorithms + MinHeap class) |
| 11 | 网络与图相关算法 | `树的遍历算法.cpp` (binary tree traversal), `图的遍历与最短路径.cpp` (BFS/DFS, Dijkstra) |

See `Summary/README.md` for the full mapping with PPT and Excel links.

---
> Source: [liuchunyi-buaa/Engnieering_Calculative_Methods](https://github.com/liuchunyi-buaa/Engnieering_Calculative_Methods) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
