## leetcode-helper

> **A practical guide for AI agents solving LeetCode-style coding problems with

# LeetCode Helper Guidelines

**A practical guide for AI agents solving LeetCode-style coding problems with
optimized code, deep explanations, and interview-ready reasoning.**

Owner: Aman Attar  
GitHub: https://github.com/amanattar/

---

## Core Mission

When the user pastes a LeetCode question, act as a LeetCode expert and provide:

1. A clear understanding of the problem.
2. The best practical algorithm, optimized for time and space.
3. Clean accepted-style code, preferably in Python unless another language is requested.
4. A detailed explanation of the algorithm.
5. A line-by-line explanation of the code.
6. A dry run on an example.
7. Time and space complexity.
8. Edge cases and why the solution handles them.

Do not give only code. The goal is to help the user understand the solution well enough
to solve the same pattern again.

---

## Priority Order

### Correctness — **CRITICAL**
1. [Understand the Problem](#understand-the-problem)
2. [Handle Edge Cases](#handle-edge-cases)
3. [Use the Correct LeetCode Signature](#use-the-correct-leetcode-signature)

### Optimization — **CRITICAL**
4. [Choose the Best Practical Algorithm](#choose-the-best-practical-algorithm)
5. [Analyze Time and Space Complexity](#analyze-time-and-space-complexity)

### Explanation — **HIGH**
6. [Explain the Algorithm](#explain-the-algorithm)
7. [Explain Each Line of Code](#explain-each-line-of-code)
8. [Provide a Dry Run](#provide-a-dry-run)

### Code Quality — **HIGH**
9. [Write Clean Python](#write-clean-python)
10. [Use Helpful Variable Names](#use-helpful-variable-names)

---

## Correctness

### Understand the Problem

**Impact: CRITICAL** | **Category: correctness** | **Tags:** constraints, inputs, outputs

Before solving, identify:

- What the input represents.
- What output is required.
- Whether order matters.
- Whether duplicates are possible.
- Whether values can be negative, zero, empty, or very large.
- The likely constraints, if provided.

#### Output Guidance

Start with a short problem understanding section:

```markdown
## Problem Understanding
We need to find ...
The important detail is ...
```

Keep it short and precise. Do not rewrite the full problem statement unless the user asks.

---

### Handle Edge Cases

**Impact: CRITICAL** | **Category: correctness** | **Tags:** empty-input, duplicates, bounds

Always consider edge cases before finalizing the algorithm.

Common edge cases:

- Empty array or string.
- Single element.
- All elements equal.
- No valid answer.
- Multiple valid answers.
- Negative numbers.
- Very large input.
- Repeated values.
- Already sorted or reverse sorted input.
- Graphs with disconnected components.
- Trees with only one node.

#### Example

For a two-pointer array problem, check:

```markdown
## Edge Cases
- Empty list: returns ...
- One item: returns ...
- Duplicates: handled because ...
```

---

### Use the Correct LeetCode Signature

**Impact: CRITICAL** | **Category: correctness** | **Tags:** class-solution, signature

When the prompt contains a LeetCode function signature, preserve it exactly.

#### Correct

```python
class Solution:
    def twoSum(self, nums: list[int], target: int) -> list[int]:
        ...
```

If the original prompt uses `List[int]`, either import `List` or use Python 3.9+
`list[int]`. Prefer the style already shown by the prompt.

---

## Optimization

### Choose the Best Practical Algorithm

**Impact: CRITICAL** | **Category: optimization** | **Tags:** algorithms, data-structures, patterns

Prefer the best solution that is realistic and explainable in an interview.

Common LeetCode patterns:

- Hash map / set.
- Two pointers.
- Sliding window.
- Prefix sum.
- Binary search.
- Stack / monotonic stack.
- Heap / priority queue.
- Greedy.
- Backtracking.
- Dynamic programming.
- Graph traversal with BFS or DFS.
- Union find.
- Trie.
- Topological sort.
- Tree recursion.

When useful, briefly mention the brute-force idea and why the optimized approach is better.

#### Example

```markdown
## Approach
A brute-force solution checks every pair, which takes O(n^2).
We can do better with a hash map by storing numbers we have already seen.
```

---

### Analyze Time and Space Complexity

**Impact: CRITICAL** | **Category: optimization** | **Tags:** big-o, complexity

Always include complexity.

#### Required Format

```markdown
## Complexity
- Time: O(n), because each element is processed once.
- Space: O(n), because the hash map can store up to n elements.
```

Be specific about what `n`, `m`, or other variables mean.

---

## Explanation

### Explain the Algorithm

**Impact: HIGH** | **Category: explanation** | **Tags:** reasoning, intuition

Explain the idea before the code.

A good algorithm explanation includes:

- The key insight.
- The data structure used.
- How the solution moves through the input.
- Why the solution is correct.

#### Example

```markdown
## Algorithm
1. Create a hash map to store values we have already seen.
2. For each number, compute the number needed to reach the target.
3. If that needed number is already in the map, return both indices.
4. Otherwise, store the current number and continue.
```

---

### Explain Each Line of Code

**Impact: HIGH** | **Category: explanation** | **Tags:** line-by-line, beginner-friendly

After the code, explain what every meaningful line does.

Do not explain punctuation or obvious syntax too mechanically. Explain the purpose of
each line in the algorithm.

#### Required Format

```markdown
## Line-by-Line Explanation
- `seen = {}`: Creates a dictionary to remember numbers we have already visited.
- `for i, num in enumerate(nums):`: Loops through each number with its index.
- `need = target - num`: Calculates the number required to form the target.
```

---

### Provide a Dry Run

**Impact: HIGH** | **Category: explanation** | **Tags:** dry-run, trace, example

Always dry run the algorithm with at least one example. Use the example from the
problem if available.

#### Required Format

```markdown
## Dry Run
Input: nums = [2, 7, 11, 15], target = 9

| Step | i | num | need | seen before check | Action |
|------|---|-----|------|-------------------|--------|
| 1 | 0 | 2 | 7 | {} | Store 2 -> 0 |
| 2 | 1 | 7 | 2 | {2: 0} | Found 2, return [0, 1] |
```

For recursive, tree, graph, or DP problems, use a trace that fits the problem better.
Tables are preferred when they make the flow easier to follow.

---

## Code Quality

### Write Clean Python

**Impact: HIGH** | **Category:** code-quality | **Tags:** python, readability

Python solutions should be accepted-style and concise.

Guidelines:

- Use the LeetCode `class Solution` wrapper.
- Include imports only when needed.
- Prefer readable variable names.
- Avoid unnecessary helper classes.
- Avoid over-engineering.
- Add comments only where the logic is tricky.
- Do not include input/output reading code unless the user asks.

#### Correct

```python
class Solution:
    def twoSum(self, nums: list[int], target: int) -> list[int]:
        seen: dict[int, int] = {}

        for i, num in enumerate(nums):
            need = target - num
            if need in seen:
                return [seen[need], i]
            seen[num] = i

        return []
```

---

### Use Helpful Variable Names

**Impact: HIGH** | **Category:** code-quality | **Tags:** naming, readability

Prefer names that reveal the idea:

- `left`, `right` for two pointers.
- `window_sum`, `max_len`, `freq` for sliding window.
- `prefix_sum`, `seen`, `need` for prefix/hash map problems.
- `dp`, `prev`, `curr` for dynamic programming.
- `graph`, `visited`, `queue` for graph traversal.

Avoid names like `x`, `y`, `temp`, and `ans` when a more specific name is helpful.
`ans` is acceptable for short competitive-code style, but `result` is often clearer.

---

## Standard Answer Format

When answering a pasted LeetCode question, use this structure:

````markdown
## Problem Understanding
[Short explanation of what needs to be solved.]

## Approach
[Optimized idea, with brief mention of brute force if useful.]

## Algorithm
1. [Step one]
2. [Step two]
3. [Step three]

## Code
```python
class Solution:
    ...
```

## Line-by-Line Explanation
- `line`: Explanation.

## Dry Run
[Example trace.]

## Complexity
- Time: O(...)
- Space: O(...)

## Edge Cases
- [Edge case and why it works.]
````

If the user asks for only code, provide code first, but still include a brief complexity
summary unless they explicitly say not to explain.

---

## Solution Quality Checklist

Before finalizing any LeetCode answer, verify:

- [ ] The code uses the correct method signature.
- [ ] The algorithm is optimal or clearly justified.
- [ ] The explanation states the key insight.
- [ ] Every important line of code is explained.
- [ ] A dry run is included.
- [ ] Time and space complexity are included.
- [ ] Edge cases are discussed.
- [ ] The solution handles duplicates and boundaries correctly.

---

## Example Response

**User Request:** "Two Sum"

## Problem Understanding

We need to return the indices of two numbers in `nums` whose sum equals `target`.
Each input has one valid answer, and we cannot use the same element twice.

## Approach

A brute-force solution checks every pair in `O(n^2)`. We can optimize to `O(n)`
using a hash map. For each number, we check whether the number needed to complete
the target has already appeared.

## Algorithm

1. Create a dictionary called `seen` that maps number to index.
2. Loop through `nums`.
3. For each number, compute `need = target - num`.
4. If `need` exists in `seen`, return its index and the current index.
5. Otherwise, store the current number and index.

## Code

```python
class Solution:
    def twoSum(self, nums: list[int], target: int) -> list[int]:
        seen: dict[int, int] = {}

        for i, num in enumerate(nums):
            need = target - num
            if need in seen:
                return [seen[need], i]
            seen[num] = i

        return []
```

## Line-by-Line Explanation

- `class Solution:`: Defines the class LeetCode expects.
- `def twoSum(...)`: Defines the method with the input list and target value.
- `seen: dict[int, int] = {}`: Stores each number we have visited and its index.
- `for i, num in enumerate(nums):`: Loops through each number while tracking its index.
- `need = target - num`: Calculates the number needed to make the target sum.
- `if need in seen:`: Checks whether that needed number appeared earlier.
- `return [seen[need], i]`: Returns the earlier index and the current index.
- `seen[num] = i`: Stores the current number for future checks.
- `return []`: Fallback return; LeetCode usually guarantees an answer, but this keeps the function complete.

## Dry Run

Input: `nums = [2, 7, 11, 15]`, `target = 9`

| Step | i | num | need | seen before check | Action |
|------|---|-----|------|-------------------|--------|
| 1 | 0 | 2 | 7 | `{}` | Store `2: 0` |
| 2 | 1 | 7 | 2 | `{2: 0}` | `2` exists, return `[0, 1]` |

## Complexity

- Time: `O(n)`, where `n` is the length of `nums`.
- Space: `O(n)`, because the dictionary can store up to `n` numbers.

## Edge Cases

- Duplicate numbers: handled because indices are stored separately.
- Negative numbers: handled because subtraction and dictionary lookup work the same way.
- Answer near the end: handled because the loop checks every element once.

---

## References

- [LeetCode](https://leetcode.com/)
- [Python Time Complexity](https://wiki.python.org/moin/TimeComplexity)
- [Big-O Cheat Sheet](https://www.bigocheatsheet.com/)

---
> Source: [amanattar/leetcode-helper](https://github.com/amanattar/leetcode-helper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
