# Chapter 13. Recurrence Relations (점화식)

> Express sequences mathematically: the foundation of dynamic programming.

---

## Concept (개념)

A **recurrence relation** defines a sequence where each term depends on previous terms.

**Form:** `a_n = f(a_{n-1}, a_{n-2}, ...)`

**Components:**
- **Recurrence equation:** How to compute next term
- **Initial conditions:** Starting values (base cases)

**Common patterns:**
- Linear: `a_n = c₁·a_{n-1} + c₂·a_{n-2}`
- Divide and conquer: `T(n) = aT(n/b) + f(n)`

---

## Fibonacci Sequence (피보나치 수열)

### Concept (개념)

The classic recurrence relation:

```
F(0) = 0
F(1) = 1
F(n) = F(n-1) + F(n-2)  for n ≥ 2
```

Sequence: 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, ...

### Example

```python
def fibonacci_examples():
    """Different ways to compute Fibonacci"""

    # 1. Recursive (inefficient - exponential time)
    def fib_recursive(n):
        if n <= 1:
            return n
        return fib_recursive(n-1) + fib_recursive(n-2)

    # 2. Memoization (top-down DP)
    def fib_memo(n, memo={}):
        if n in memo:
            return memo[n]
        if n <= 1:
            return n
        memo[n] = fib_memo(n-1, memo) + fib_memo(n-2, memo)
        return memo[n]

    # 3. Iterative (bottom-up DP)
    def fib_iterative(n):
        if n <= 1:
            return n
        prev2, prev1 = 0, 1
        for _ in range(2, n + 1):
            curr = prev1 + prev2
            prev2, prev1 = prev1, curr
        return prev1

    # 4. Matrix exponentiation (O(log n))
    def fib_matrix(n):
        if n <= 1:
            return n

        def matrix_mult(A, B):
            return [
                [A[0][0]*B[0][0] + A[0][1]*B[1][0],
                 A[0][0]*B[0][1] + A[0][1]*B[1][1]],
                [A[1][0]*B[0][0] + A[1][1]*B[1][0],
                 A[1][0]*B[0][1] + A[1][1]*B[1][1]]
            ]

        def matrix_pow(M, p):
            if p == 1:
                return M
            if p % 2 == 0:
                half = matrix_pow(M, p // 2)
                return matrix_mult(half, half)
            return matrix_mult(M, matrix_pow(M, p - 1))

        base = [[1, 1], [1, 0]]
        result = matrix_pow(base, n)
        return result[0][1]

    print("Fibonacci sequence:")
    print(f"  First 15 terms: {[fib_iterative(i) for i in range(15)]}")

    # Compare methods
    n = 30
    print(f"\nF({n}) computed by different methods:")
    print(f"  Memoization: {fib_memo(n)}")
    print(f"  Iterative:   {fib_iterative(n)}")
    print(f"  Matrix:      {fib_matrix(n)}")


fibonacci_examples()


# Fibonacci in nature and algorithms
def fibonacci_applications():
    """Real applications of Fibonacci"""

    print("\nFibonacci Applications:")

    # 1. Climbing stairs (how many ways to climb n stairs, 1 or 2 steps at a time)
    def climb_stairs(n):
        """Same as Fibonacci!"""
        if n <= 2:
            return n
        prev2, prev1 = 1, 2
        for _ in range(3, n + 1):
            curr = prev1 + prev2
            prev2, prev1 = prev1, curr
        return prev1

    print(f"\n1. Climbing stairs (1 or 2 steps):")
    for n in range(1, 8):
        print(f"   {n} stairs: {climb_stairs(n)} ways")

    # 2. Tiling problem (2×n board with 2×1 dominoes)
    # Same recurrence as Fibonacci
    print(f"\n2. Tiling 2×n board with dominoes:")
    print(f"   Same as climbing stairs!")

    # 3. Binary strings without consecutive 1s
    def no_consecutive_ones(n):
        """Count n-bit strings with no consecutive 1s"""
        if n == 1:
            return 2  # "0", "1"
        if n == 2:
            return 3  # "00", "01", "10"

        # a[i] = strings ending in 0
        # b[i] = strings ending in 1
        # a[i] = a[i-1] + b[i-1] (can add 0 to anything)
        # b[i] = a[i-1] (can only add 1 after 0)
        a, b = 2, 1  # For n=2: "00","10" end in 0; "01" ends in 1

        for _ in range(3, n + 1):
            a, b = a + b, a

        return a + b

    print(f"\n3. Binary strings without consecutive 1s:")
    for n in range(1, 8):
        print(f"   {n} bits: {no_consecutive_ones(n)} strings")


fibonacci_applications()
```

---

## Setting Up Recurrences (점화식 세우기)

### Concept (개념)

To set up a recurrence:
1. Define what `a_n` represents
2. Find how `a_n` relates to smaller cases
3. Identify base cases

**Key insight:** Break problem into subproblems.

### Example

```python
def setup_recurrences():
    """Examples of setting up recurrence relations"""

    print("Setting Up Recurrence Relations:\n")

    # 1. Tower of Hanoi
    print("1. Tower of Hanoi (n disks):")
    print("   T(1) = 1")
    print("   T(n) = 2·T(n-1) + 1")
    print("   (Move n-1 to temp, move largest, move n-1 to target)")

    def hanoi(n):
        if n == 1:
            return 1
        return 2 * hanoi(n-1) + 1

    for n in range(1, 8):
        print(f"   T({n}) = {hanoi(n)}")

    # Closed form: T(n) = 2^n - 1
    print(f"   Closed form: T(n) = 2^n - 1")

    # 2. Binary search comparisons
    print("\n2. Binary Search (worst case comparisons):")
    print("   T(1) = 1")
    print("   T(n) = T(n/2) + 1")

    def binary_search_comparisons(n):
        if n <= 1:
            return 1
        return binary_search_comparisons(n // 2) + 1

    for n in [1, 2, 4, 8, 16, 32, 64]:
        print(f"   T({n}) = {binary_search_comparisons(n)}")

    print(f"   Closed form: T(n) = ⌊log₂n⌋ + 1")

    # 3. Merge sort comparisons
    print("\n3. Merge Sort (comparisons):")
    print("   T(1) = 0")
    print("   T(n) = 2·T(n/2) + n")

    def merge_sort_comparisons(n):
        if n <= 1:
            return 0
        return 2 * merge_sort_comparisons(n // 2) + n

    for n in [1, 2, 4, 8, 16, 32]:
        print(f"   T({n}) = {merge_sort_comparisons(n)}")

    print(f"   Closed form: T(n) = n·log₂n")

    # 4. Counting paths in grid
    print("\n4. Paths in m×n grid (only right or down):")
    print("   P(1,n) = 1, P(m,1) = 1")
    print("   P(m,n) = P(m-1,n) + P(m,n-1)")

    def grid_paths(m, n):
        if m == 1 or n == 1:
            return 1
        return grid_paths(m-1, n) + grid_paths(m, n-1)

    print("   Grid paths table:")
    for m in range(1, 5):
        row = [grid_paths(m, n) for n in range(1, 6)]
        print(f"   m={m}: {row}")


setup_recurrences()


def more_recurrence_examples():
    """Additional recurrence setups"""

    print("\nMore Recurrence Examples:\n")

    # 5. Derangements
    print("5. Derangements (permutations with no fixed points):")
    print("   D(1) = 0, D(2) = 1")
    print("   D(n) = (n-1)·(D(n-1) + D(n-2))")

    def derangement(n, memo={}):
        if n == 0:
            return 1
        if n == 1:
            return 0
        if n in memo:
            return memo[n]
        memo[n] = (n - 1) * (derangement(n-1, memo) + derangement(n-2, memo))
        return memo[n]

    for n in range(1, 8):
        print(f"   D({n}) = {derangement(n)}")

    # 6. Catalan numbers
    print("\n6. Catalan Numbers:")
    print("   C(0) = 1")
    print("   C(n) = Σ C(i)·C(n-1-i) for i=0 to n-1")
    print("   Or: C(n) = C(2n,n)/(n+1)")

    def catalan(n, memo={}):
        if n <= 1:
            return 1
        if n in memo:
            return memo[n]

        result = 0
        for i in range(n):
            result += catalan(i, memo) * catalan(n-1-i, memo)
        memo[n] = result
        return result

    print("   Values:", [catalan(i) for i in range(10)])
    print("   Applications: valid parentheses, binary trees, polygon triangulations")


more_recurrence_examples()
```

---

## Recurrence and Dynamic Programming (점화식과 DP)

### Concept (개념)

**Dynamic Programming** is the algorithmic technique for solving recurrences efficiently:

1. **Define subproblems:** What does `dp[i]` represent?
2. **Find recurrence:** How does `dp[i]` relate to smaller indices?
3. **Identify base cases:** Initial values
4. **Determine order:** Bottom-up or top-down?

**Optimization:** Recurrence → DP table → Sometimes space optimization

### Example

```python
def dp_from_recurrence():
    """Converting recurrences to DP solutions"""

    print("From Recurrence to DP:\n")

    # Example 1: Coin change (minimum coins)
    print("1. Coin Change - Minimum coins for amount")
    print("   Recurrence: dp[i] = min(dp[i - coin] + 1) for each coin")
    print("   Base: dp[0] = 0")

    def min_coins(coins, amount):
        dp = [float('inf')] * (amount + 1)
        dp[0] = 0

        for i in range(1, amount + 1):
            for coin in coins:
                if coin <= i and dp[i - coin] != float('inf'):
                    dp[i] = min(dp[i], dp[i - coin] + 1)

        return dp[amount] if dp[amount] != float('inf') else -1

    coins = [1, 5, 10, 25]
    for amount in [11, 15, 30, 43]:
        result = min_coins(coins, amount)
        print(f"   Amount {amount}: {result} coins")

    # Example 2: Longest Increasing Subsequence
    print("\n2. Longest Increasing Subsequence (LIS)")
    print("   Recurrence: dp[i] = max(dp[j] + 1) for j < i where a[j] < a[i]")
    print("   Base: dp[i] = 1 for all i")

    def lis(arr):
        n = len(arr)
        dp = [1] * n

        for i in range(1, n):
            for j in range(i):
                if arr[j] < arr[i]:
                    dp[i] = max(dp[i], dp[j] + 1)

        return max(dp)

    test_arr = [10, 22, 9, 33, 21, 50, 41, 60, 80]
    print(f"   Array: {test_arr}")
    print(f"   LIS length: {lis(test_arr)}")

    # Example 3: 0/1 Knapsack
    print("\n3. 0/1 Knapsack")
    print("   Recurrence: dp[i][w] = max(dp[i-1][w], dp[i-1][w-wt[i]] + val[i])")
    print("   Base: dp[0][w] = 0 for all w")

    def knapsack(weights, values, capacity):
        n = len(weights)
        dp = [[0] * (capacity + 1) for _ in range(n + 1)]

        for i in range(1, n + 1):
            for w in range(capacity + 1):
                # Don't take item i
                dp[i][w] = dp[i-1][w]
                # Take item i (if possible)
                if weights[i-1] <= w:
                    dp[i][w] = max(dp[i][w],
                                  dp[i-1][w - weights[i-1]] + values[i-1])

        return dp[n][capacity]

    weights = [2, 3, 4, 5]
    values = [3, 4, 5, 6]
    capacity = 8
    print(f"   Weights: {weights}, Values: {values}, Capacity: {capacity}")
    print(f"   Max value: {knapsack(weights, values, capacity)}")


dp_from_recurrence()


def space_optimization():
    """Optimizing space in DP"""

    print("\nSpace Optimization in DP:\n")

    # Original Fibonacci: O(n) space
    def fib_array(n):
        if n <= 1:
            return n
        dp = [0] * (n + 1)
        dp[1] = 1
        for i in range(2, n + 1):
            dp[i] = dp[i-1] + dp[i-2]
        return dp[n]

    # Optimized: O(1) space
    def fib_optimized(n):
        if n <= 1:
            return n
        prev2, prev1 = 0, 1
        for _ in range(2, n + 1):
            curr = prev1 + prev2
            prev2, prev1 = prev1, curr
        return prev1

    print("Fibonacci space optimization:")
    print(f"  Array method (O(n) space): F(10) = {fib_array(10)}")
    print(f"  Two variables (O(1) space): F(10) = {fib_optimized(10)}")

    # When can we optimize?
    print("\nSpace optimization possible when:")
    print("  - dp[i] only depends on dp[i-1] (and maybe dp[i-2])")
    print("  - We only need the final answer, not the path")
    print("  - For 2D: dp[i][j] depends only on previous row")


space_optimization()
```

---

## Summary

| Concept | Korean | Key Point |
|---------|--------|-----------|
| Recurrence | 점화식 | a_n depends on previous terms |
| Initial Conditions | 초기 조건 | Base cases for recursion |
| Fibonacci | 피보나치 | F(n) = F(n-1) + F(n-2) |
| DP Connection | DP 연결 | Recurrence → DP table |

**Setting up recurrences:**
1. Define what a_n means
2. Break into subproblems
3. Find the relationship
4. Set base cases

---

## Practice Problems

1. Set up recurrence for: Ways to tile a 3×n floor with 1×3 tiles.

2. The recurrence T(n) = T(n-1) + n with T(1) = 1. Find T(10).

3. A frog can jump 1, 2, or 3 steps. Write recurrence for ways to reach step n.

4. Convert this recurrence to DP: `a_n = a_{n-1} + 2·a_{n-2}` with a_0 = 1, a_1 = 1.
