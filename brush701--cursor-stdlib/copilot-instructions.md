## cursor-stdlib

> Best practices for Python performance optimization to write more efficient and scalable code.

# Python Performance Optimization Best Practices

Guidelines for writing efficient and performant Python code.

<rule>
name: python_performance
description: Best practices for Python performance optimization to write more efficient and scalable code
filters:
  - type: file_extension
    pattern: "\\.py$"
  - type: event
    pattern: "file_create|file_modify"

actions:
  - type: suggest
    message: |
      ## Python Performance Optimization Best Practices

      ### Use Appropriate Data Structures

      ```python
      # Lists vs. Sets for membership testing
      # ❌ Slow (O(n) time complexity):
      names = ["Alice", "Bob", "Charlie", "David"]
      if "Charlie" in names:  # Linear search
          print("Found Charlie")
          
      # ✅ Fast (O(1) time complexity):
      names = {"Alice", "Bob", "Charlie", "David"}  # Set
      if "Charlie" in names:  # Constant-time lookup
          print("Found Charlie")
          
      # Dictionaries for fast lookups
      # ❌ Inefficient:
      users = [(1, "Alice"), (2, "Bob"), (3, "Charlie")]
      user_id = 2
      user_name = None
      for id, name in users:
          if id == user_id:
              user_name = name
              break
              
      # ✅ Efficient:
      users = {1: "Alice", 2: "Bob", 3: "Charlie"}
      user_name = users.get(user_id)  # O(1) lookup
      ```

      ### Use List Comprehensions and Generator Expressions

      ```python
      # ❌ Less efficient and less readable:
      squares = []
      for i in range(1000):
          squares.append(i * i)
          
      # ✅ More efficient and more readable:
      squares = [i * i for i in range(1000)]
      
      # When you don't need all values at once, use generators
      # ✅ Memory efficient for large datasets:
      squares_gen = (i * i for i in range(1000000))
      for square in squares_gen:
          # Process one square at a time
          process(square)
      ```

      ### Avoid Unnecessary Computation in Loops

      ```python
      # ❌ Inefficient:
      for i in range(1000):
          result = expensive_function()  # Same result each time
          data[i] = result * i
          
      # ✅ More efficient:
      expensive_result = expensive_function()  # Compute once
      for i in range(1000):
          data[i] = expensive_result * i
          
      # ❌ Inefficient - len() called on each iteration:
      items = [1, 2, 3, 4, 5]
      for i in range(len(items)):
          print(f"Processing {i} of {len(items)}")
          
      # ✅ More efficient:
      items = [1, 2, 3, 4, 5]
      items_len = len(items)
      for i in range(items_len):
          print(f"Processing {i} of {items_len}")
      ```

      ### Use Built-in Functions and Libraries

      ```python
      # ❌ Reinventing the wheel:
      def get_max_value(numbers):
          max_val = numbers[0]
          for num in numbers[1:]:
              if num > max_val:
                  max_val = num
          return max_val
          
      # ✅ Using built-ins:
      def get_max_value(numbers):
          return max(numbers)
          
      # ❌ Slow custom implementation:
      def filter_even(numbers):
          result = []
          for num in numbers:
              if num % 2 == 0:
                  result.append(num)
          return result
          
      # ✅ Using built-ins:
      def filter_even(numbers):
          return list(filter(lambda x: x % 2 == 0, numbers))
          
      # Even better with list comprehension:
      def filter_even(numbers):
          return [x for x in numbers if x % 2 == 0]
      ```

      ### Use String Joining Efficiently

      ```python
      # ❌ Inefficient string concatenation:
      result = ""
      for i in range(1000):
          result += str(i) + ", "  # Creates new string each time
          
      # ✅ Efficient string joining:
      parts = []
      for i in range(1000):
          parts.append(str(i))
      result = ", ".join(parts)  # Single join operation
      
      # Even better with list comprehension:
      result = ", ".join(str(i) for i in range(1000))
      ```

      ### Profile Before Optimizing

      ```python
      # Simple timing measurement
      import time
      
      def measure_time(func, *args, **kwargs):
          start = time.time()
          result = func(*args, **kwargs)
          end = time.time()
          print(f"{func.__name__} took {end - start:.6f} seconds")
          return result
          
      # Using the built-in profiler
      import cProfile
      import pstats
      
      def profile_function(func, *args, **kwargs):
          profiler = cProfile.Profile()
          profiler.enable()
          result = func(*args, **kwargs)
          profiler.disable()
          stats = pstats.Stats(profiler).sort_stats('cumulative')
          stats.print_stats(10)  # Print top 10 functions by cumulative time
          return result
      ```

      ### Use Caching for Expensive Operations

      ```python
      # Simple manual caching
      _factorial_cache = {}
      
      def factorial(n):
          if n in _factorial_cache:
              return _factorial_cache[n]
              
          if n <= 1:
              result = 1
          else:
              result = n * factorial(n - 1)
              
          _factorial_cache[n] = result
          return result
          
      # Better: Using functools.lru_cache
      from functools import lru_cache
      
      @lru_cache(maxsize=128)
      def fibonacci(n):
          if n <= 1:
              return n
          return fibonacci(n-1) + fibonacci(n-2)
      ```

      ### Optimize I/O Operations

      ```python
      # ❌ Inefficient file reading:
      with open("large_file.txt", "r") as f:
          for line in f:
              # Process each line one at a time
              process_line(line)
              
      # ✅ More efficient for binary data:
      with open("large_binary_file.bin", "rb") as f:
          # Read in chunks
          chunk_size = 8192  # 8K chunks
          while True:
              chunk = f.read(chunk_size)
              if not chunk:
                  break
              process_chunk(chunk)
              
      # ❌ Inefficient database querying:
      for id in ids:
          # N individual queries
          result = cursor.execute("SELECT * FROM items WHERE id = ?", (id,))
          process_result(result)
          
      # ✅ More efficient batched querying:
      # Single query with all IDs
      placeholders = ",".join("?" for _ in ids)
      query = f"SELECT * FROM items WHERE id IN ({placeholders})"
      results = cursor.execute(query, ids)
      for result in results:
          process_result(result)
      ```

      ### Consider Parallelism for CPU-Bound Tasks

      ```python
      # Using multiprocessing for CPU-bound tasks
      from multiprocessing import Pool
      
      def process_item(item):
          # CPU-intensive operation
          return complex_calculation(item)
      
      def process_all_items(items):
          # Using multiple processes
          with Pool(processes=4) as pool:
              results = pool.map(process_item, items)
          return results
      ```

      ### Use NumPy for Numerical Operations

      ```python
      # ❌ Inefficient numerical computation:
      def matrix_multiply(a, b):
          result = [[0 for _ in range(len(b[0]))] for _ in range(len(a))]
          for i in range(len(a)):
              for j in range(len(b[0])):
                  for k in range(len(b)):
                      result[i][j] += a[i][k] * b[k][j]
          return result
          
      # ✅ Efficient with NumPy:
      import numpy as np
      
      def matrix_multiply(a, b):
          a_array = np.array(a)
          b_array = np.array(b)
          return a_array @ b_array  # Matrix multiplication
      ```

examples:
  - input: |
      def find_duplicates(items):
          duplicates = []
          for i in range(len(items)):
              for j in range(i+1, len(items)):
                  if items[i] == items[j] and items[i] not in duplicates:
                      duplicates.append(items[i])
          return duplicates
    output: |
      from typing import List, TypeVar, Set
      from collections import Counter
      
      T = TypeVar('T', bound=hash)
      
      def find_duplicates(items: List[T]) -> List[T]:
          """
          Find duplicate items in a list.
          
          Args:
              items: List of hashable items to check for duplicates
              
          Returns:
              List of items that appear more than once
          """
          # Method 1: Using Counter (most readable)
          counts = Counter(items)
          return [item for item, count in counts.items() if count > 1]
          
          # Method 2: Using a set for O(n) time complexity
          # seen = set()
          # duplicates = set()
          # for item in items:
          #     if item in seen:
          #         duplicates.add(item)
          #     else:
          #         seen.add(item)
          # return list(duplicates)

metadata:
  priority: high
  version: 1.0
</rule> 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brush701) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
