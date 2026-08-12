## algorithms

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Python DSA (Data Structures & Algorithms) interview prep repository. Problems are solved as standalone `.py` files, organized by topic.

## Running Code

```bash
python problems/trees/build_tree.py       # Run a problem solution (from project root)
python dsa/stack.py                       # Run a data structure implementation
```

No build system, package manager, linter, or test framework is configured. Files are run individually with plain Python **from the project root**.

## Repository Structure

- **`problems/`** — LeetCode-style problem solutions, organized by topic:
  - `arrays/` — two_sum, plus_one, rotate_left_till_zero, add_binary
  - `binary_search/` — binary_search
  - `trees/` — build_tree, is_symmetric_tree, maximum_depth_btree, sorted_array_to_bst, subtree_of_another_tree
  - `backtracking/` — get_subsets, get_subsets_2, phone_num_letter_combination
  - `linked_lists/` — reverse_linked_list
  - `graphs/` — bfs_graph, person
- **`dsa/`** — Data structure implementations:
  - Root: linked list, doubly-linked list, stack, queue, deque, binary tree
  - `tree/` — BST, B-tree node
- **`sorting/`** — Sorting algorithm implementations (selection sort, quicksort)
- **`utils/`** — Shared node classes — **import these instead of defining locally**:
  - `Node` (singly-linked) — used by linked list, stack, queue
  - `DblNode` (doubly-linked) — used by doubly-linked list, deque
  - `TreeNode` (binary tree) — used by tree problem solutions
  - `ListNode` (LeetCode-style linked list) — used by linked list problems

## Tutoring Policy

This is an interview prep repo — the goal is learning, not just getting answers. When the user is working on a problem:

- **Never provide full solutions.** Instead, give hints, ask guiding questions, and teach the underlying concepts.
- **Escalate help gradually:** start with a small conceptual hint, then a more specific nudge, then pseudocode — but never jump straight to working code.
- **Explain patterns and reasoning** (e.g., "this is a sliding window problem because…") so the user builds problem-solving intuition.
- **It's okay to write code** for utility/infrastructure tasks (data structure implementations, test harnesses, debugging helpers) — the restriction applies to solving the interview-style problems themselves.

## Conventions

- Solutions follow a pattern comment header style: `# Pattern: ..., # Complexity: ..., # Lesson: ...`
- `MISTAKES_LOG.md` tracks problem-solving mistakes with structured entries (date, pattern, what went wrong, root cause, fix). New entries go at the top of the Log section.
- `README.md` contains the interview prep methodology/process guide — not repo documentation.

---
> Source: [Andrew0914/algorithms](https://github.com/Andrew0914/algorithms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
