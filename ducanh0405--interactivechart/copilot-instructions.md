## interactivechart

> You are working on a Data Structures and Algorithms (DSA) university project: an Interactive Chart Application for data visualization built with React + TypeScript.

You are working on a Data Structures and Algorithms (DSA) university project: an Interactive Chart Application for data visualization built with React + TypeScript.

## PROJECT IDENTITY
- **Course**: Data Structures and Algorithms (DSA)
- **Project**: DataViz - Interactive Chart Application
- **Purpose**: Educational project demonstrating DSA concepts through data visualization
- **Tech Stack**: React 18, TypeScript, Vite, Plotly.js, Tailwind CSS, Supabase

## CORE DSA CONCEPTS TO IMPLEMENT & OPTIMIZE

### 1. Data Structure Usage
**Arrays & Lists**
- Use arrays for storing CSV data rows and chart data points
- Implement efficient array operations for data filtering and sorting
- Apply dynamic arrays for flexible data storage

**Hash Tables (Objects/Maps)**
- Use Map/Object for O(1) column lookup by name
- Implement hash-based caching for chart recommendations
- Store dashboard layouts with hash-based keys

**Trees**
- Parse CSV hierarchical data into tree structures
- Implement decision trees for AI chart recommendations
- Use tree traversal for data profiling and analysis

**Graphs**
- Model data relationships as graphs for correlation analysis
- Implement graph algorithms for dashboard layout optimization
- Use adjacency lists for representing data dependencies

**Queues & Stacks**
- Implement undo/redo stack for dashboard operations
- Use queue for processing large CSV files in chunks
- Apply stack for navigation history management

### 2. Algorithm Implementation Requirements

**Sorting Algorithms** (Time Complexity Matters!)
- Implement efficient sorting for chart data display
- Use QuickSort/MergeSort (O(n log n)) for large datasets
- Apply Counting Sort for categorical data (O(n + k))
- Document time complexity in comments

**Searching Algorithms**
- Binary search for sorted data lookup (O(log n))
- Hash-based search for O(1) column access
- Implement efficient filtering algorithms for chart gallery

**Graph Algorithms**
- DFS/BFS for data relationship exploration
- Shortest path algorithms for optimal dashboard layouts
- Minimum spanning tree for data correlation visualization

**Dynamic Programming**
- Memoization for expensive chart calculations
- Optimize chart recommendation algorithm with DP
- Cache intermediate results to avoid recomputation

**Greedy Algorithms**
- Apply greedy approach for dashboard auto-layout
- Optimize space allocation for chart grid placement
- Use greedy heuristics for chart type selection

### 3. Performance & Complexity Analysis

**MANDATORY: Always document algorithm complexity**
- Every algorithm must have JSDoc noting time/space complexity and main data structures.
- Prefer O(n log n) or better on critical paths; avoid O(n²) for datasets > 10,000 rows.
- Use memoization/caching to prevent redundant work across renders or repeated uploads.
- Benchmark targets: CSV parsing ≤2s for 10MB, chart render ≤500ms, layout calc ≤100ms, search/filter ≤50ms.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ducanh0405) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
