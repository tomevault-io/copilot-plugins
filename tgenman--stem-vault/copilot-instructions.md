## stem-vault

> Example of a page called "Arrangements without repetition"



Example of a page called "Arrangements without repetition"
```
---
aliases:
  - Размещение без повторений
  - Расстановки без повторений
anki: true
parent:
  - "[[519.101 Combinatorics MOC]]"
connected:
  - "[[Factorial]]"
  - "[[Tuple]]"
  - "[[Arrangements with repetition]]"
created: 2023-10-23 16:47
tags:
  - content/definition
---

> [!tip] An arrangement without repetition $A^{k}_{n}$
> is an ordered $n$-[[Tuple|tuple]] $(x_1,..., x_n)$ elements of $A$, where each elements occurs exactly once
>$A_n^k= V_{n, k} = \frac{n!}{(n-k)!} = n \cdot (n-1) \cdot ... \cdot (n - k + 1) = (n)_k$

# Example
For instance, if we need to choose and arrange 3 books from a set of 5 distinct books (A, B, C, D, E), the number of ways to do this is 
$A_5^3=\frac{5!}{(5−3)!}=60$

![[Pasted image 20250513181511.png|300]]

# Anki
TARGET DECK: math::combinatorics
START
Math_TWO_side
FRONT:  Arrangements without repetition
BACK:  is an ordered $n$-[[Tuple|tuple]] $(x_1,..., x_n)$ elements of $A$, where each elements occurs exactly once
FORMULA: $A_n^k= V_{n, k} = \frac{n!}{(n-k)!} = n \cdot (n-1) \cdot ... \cdot (n - k + 1) = (n)_k$
ADDITIONAL: Properties:
- property 1
- property 2
- property 3
PICTURE: ![[Pasted image 20250513181511.png]]
ID: 1698069638670
END

```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tgenman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
