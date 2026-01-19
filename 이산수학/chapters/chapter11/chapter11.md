# Chapter 11. Permutations and Combinations (순열과 조합)

> Order matters vs order doesn't: the two fundamental ways to select items.

---

## Permutations - Order Matters (순열)

### Concept (개념)

A **permutation** is an arrangement where **order matters**.

"ABC" and "CBA" are different permutations of the same letters.

**Formula:** Permutations of r items from n items:
```
P(n, r) = n! / (n-r)!
```

### Example

```python
from math import factorial, perm

def permutation(n, r):
    """Calculate P(n,r) = n! / (n-r)!"""
    return factorial(n) // factorial(n - r)


# Basic permutation
n, r = 5, 3
result = permutation(n, r)

print(f"P({n}, {r}) = {result}")
print(f"  = {n}! / ({n}-{r})!")
print(f"  = {n} × {n-1} × {n-2}")
print(f"  = {n * (n-1) * (n-2)}")


# Race positions
def race_medals():
    """10 runners, how many ways to award gold/silver/bronze?"""
    runners = 10
    medals = 3  # 1st, 2nd, 3rd

    ways = permutation(runners, medals)
    print(f"\nRace medal arrangements:")
    print(f"  {runners} runners, {medals} positions")
    print(f"  P(10, 3) = 10 × 9 × 8 = {ways}")


race_medals()


# Password permutations
def password_perms():
    """4-character password from 26 letters, no repeats"""
    letters = 26
    length = 4

    # Order matters (abcd ≠ dcba)
    ways = permutation(letters, length)
    print(f"\n4-letter password (no repeats):")
    print(f"  P(26, 4) = 26 × 25 × 24 × 23 = {ways:,}")


password_perms()


# List all permutations
from itertools import permutations

def show_permutations():
    items = ['A', 'B', 'C']
    r = 2

    print(f"\nAll 2-permutations of {items}:")
    for i, p in enumerate(permutations(items, r)):
        print(f"  {i+1}: {''.join(p)}")


show_permutations()
```

---

## Combinations - Order Doesn't Matter (조합)

### Concept (개념)

A **combination** is a selection where **order doesn't matter**.

{A, B, C} and {C, B, A} are the same combination.

**Formula:** Combinations of r items from n items:
```
C(n, r) = n! / (r! × (n-r)!)
       = P(n, r) / r!
```

Also written as "n choose r" or (n r).

### Example

```python
from math import factorial, comb

def combination(n, r):
    """Calculate C(n,r) = n! / (r! × (n-r)!)"""
    return factorial(n) // (factorial(r) * factorial(n - r))


# Basic combination
n, r = 5, 3
result = combination(n, r)

print(f"C({n}, {r}) = {result}")
print(f"  Compare to P(5,3) = {n * (n-1) * (n-2)}")
print(f"  Difference: divide by {r}! = {factorial(r)}")


# Team selection
def team_selection():
    """Choose 3 people from 10 for a team"""
    people = 10
    team_size = 3

    # Order doesn't matter ({A,B,C} = {C,B,A})
    ways = combination(people, team_size)
    print(f"\nTeam selection:")
    print(f"  Choose {team_size} from {people}")
    print(f"  C(10, 3) = {ways}")


team_selection()


# Compare P vs C
def permutation_vs_combination():
    """Show difference between P and C"""
    n, r = 4, 2

    p = permutation(n, r)
    c = combination(n, r)

    print(f"\nCompare P(4,2) vs C(4,2):")
    print(f"  P(4,2) = {p} (order matters)")
    print(f"  C(4,2) = {c} (order doesn't matter)")

    from itertools import permutations, combinations

    print(f"\n  2-permutations of [A,B,C,D]:")
    for x in permutations(['A','B','C','D'], 2):
        print(f"    {''.join(x)}")

    print(f"\n  2-combinations of [A,B,C,D]:")
    for x in combinations(['A','B','C','D'], 2):
        print(f"    {set(x)}")


permutation_vs_combination()
```

---

## Factorial (n!) (팩토리얼)

### Concept (개념)

**Factorial** is the product of all positive integers up to n:

```
n! = n × (n-1) × (n-2) × ... × 2 × 1
0! = 1 (by definition)
```

**Properties:**
- n! = n × (n-1)!
- Grows very fast (10! = 3,628,800)
- Used in permutations and combinations

### Example

```python
from math import factorial

def factorial_examples():
    print("Factorial values:")
    for n in range(11):
        print(f"  {n}! = {factorial(n):,}")

    print("\nFactorial grows FAST:")
    for n in [15, 20, 25]:
        print(f"  {n}! = {factorial(n):,}")


factorial_examples()


def factorial_recursive(n):
    """Recursive definition of factorial"""
    if n <= 1:
        return 1
    return n * factorial_recursive(n - 1)


def factorial_iterative(n):
    """Iterative factorial"""
    result = 1
    for i in range(2, n + 1):
        result *= i
    return result


# Verify both methods
print("\nFactorial implementations:")
for n in [5, 7, 10]:
    rec = factorial_recursive(n)
    itr = factorial_iterative(n)
    print(f"  {n}!: recursive={rec}, iterative={itr}")
```

---

## nPr, nCr Calculation (nPr, nCr 계산)

### Concept (개념)

**nPr** (permutation):
```
nPr = n! / (n-r)!
    = n × (n-1) × ... × (n-r+1)   [r terms]
```

**nCr** (combination):
```
nCr = n! / (r! × (n-r)!)
    = nPr / r!
```

**Useful identities:**
- nC0 = nCn = 1
- nCr = nC(n-r) (symmetry)
- nCr = (n-1)C(r-1) + (n-1)Cr (Pascal's identity)

### Example

```python
from math import factorial

def nPr(n, r):
    if r > n:
        return 0
    return factorial(n) // factorial(n - r)

def nCr(n, r):
    if r > n:
        return 0
    return factorial(n) // (factorial(r) * factorial(n - r))


# Calculate various values
print("nPr and nCr values:")
for n in range(1, 6):
    p_vals = [f"P({n},{r})={nPr(n,r)}" for r in range(n+1)]
    c_vals = [f"C({n},{r})={nCr(n,r)}" for r in range(n+1)]
    print(f"  n={n}: {', '.join(c_vals)}")


# Pascal's Triangle
def pascals_triangle(rows):
    """Generate Pascal's triangle (nCr values)"""
    print("\nPascal's Triangle:")
    for n in range(rows):
        row = [nCr(n, r) for r in range(n + 1)]
        padding = " " * (rows - n - 1) * 2
        print(padding + " ".join(f"{x:3}" for x in row))


pascals_triangle(8)


# Verify identities
def verify_identities():
    print("\nCombination identities:")

    # nCr = nC(n-r)
    n, r = 7, 2
    print(f"  C(7,2) = {nCr(7,2)}, C(7,5) = {nCr(7,5)} (symmetry)")

    # Pascal's identity
    print(f"  C(5,2) = C(4,1) + C(4,2)")
    print(f"    {nCr(5,2)} = {nCr(4,1)} + {nCr(4,2)}")


verify_identities()
```

---

## Permutations/Combinations with Repetition (중복 순열, 중복 조합)

### Concept (개념)

**Permutation with repetition:**
```
n^r (n choices for each of r positions)
```

**Combination with repetition:**
```
C(n+r-1, r) (stars and bars method)
```

**Permutation of multiset:**
```
n! / (n₁! × n₂! × ... × nₖ!)
```

### Example

```python
from math import factorial
from itertools import combinations_with_replacement, product

# Permutation with repetition
def perm_with_rep():
    """PIN code: 4 digits, repeats allowed"""
    digits = 10
    positions = 4

    # Each position: 10 choices
    ways = digits ** positions

    print("Permutation with repetition:")
    print(f"  4-digit PIN: 10^4 = {ways:,}")

    # 3-letter words from {A,B}: 2^3 = 8
    letters = 2
    length = 3
    ways2 = letters ** length
    print(f"  3-letter from {{A,B}}: 2^3 = {ways2}")
    print(f"    Words: {list(''.join(p) for p in product('AB', repeat=3))}")


perm_with_rep()


# Combination with repetition
def comb_with_rep():
    """Choose 3 items from {A,B,C} with repetition allowed"""
    n = 3  # Types of items
    r = 3  # Items to choose

    # Formula: C(n+r-1, r)
    ways = factorial(n + r - 1) // (factorial(r) * factorial(n - 1))

    print("\nCombination with repetition:")
    print(f"  Choose 3 from {{A,B,C}} (repeats ok)")
    print(f"  C(3+3-1, 3) = C(5, 3) = {ways}")

    # List them
    items = ['A', 'B', 'C']
    combos = list(combinations_with_replacement(items, 3))
    print(f"  Combinations: {combos}")


comb_with_rep()


# Multiset permutation
def multiset_permutation():
    """Arrange letters in MISSISSIPPI"""
    word = "MISSISSIPPI"

    # Count each letter
    from collections import Counter
    counts = Counter(word)
    print(f"\nMultiset permutation: '{word}'")
    print(f"  Letter counts: {dict(counts)}")

    # Formula: n! / (n1! × n2! × ...)
    n = len(word)
    denominator = 1
    for count in counts.values():
        denominator *= factorial(count)

    arrangements = factorial(n) // denominator

    print(f"  Total length: {n}")
    print(f"  Arrangements: {n}! / (4! × 4! × 2! × 1!) = {arrangements:,}")


multiset_permutation()
```

---

## Combinatorics and Time Complexity (알고리즘 시간복잡도와 조합론)

### Concept (개념)

Understanding combinatorics helps analyze algorithm complexity:

- **n!** permutations: Brute-force traveling salesman
- **2^n** subsets: Subset problems
- **C(n,r)**: Selection problems

### Example

```python
from math import factorial

def complexity_analysis():
    """Relate combinatorics to algorithm complexity"""

    print("Algorithm complexity examples:\n")

    # Brute force TSP
    cities = 10
    routes = factorial(cities)
    print(f"Traveling Salesman ({cities} cities):")
    print(f"  Brute force checks {routes:,} routes")
    print(f"  Complexity: O(n!)\n")

    # Subset sum
    items = 20
    subsets = 2 ** items
    print(f"Subset Sum ({items} items):")
    print(f"  Check all {subsets:,} subsets")
    print(f"  Complexity: O(2^n)\n")

    # Team selection
    people = 100
    team = 5
    teams = factorial(people) // (factorial(team) * factorial(people - team))
    print(f"Team Selection (choose {team} from {people}):")
    print(f"  Possible teams: {teams:,}")
    print(f"  Complexity: O(C(n,k))")


complexity_analysis()


# Practical implications
def practical_limits():
    """Show practical limits of brute force"""
    print("\nPractical limits (operations per second = 10^9):\n")

    ops_per_sec = 10 ** 9

    # Factorial growth
    print("n! growth (all permutations):")
    for n in [10, 12, 15, 20]:
        ops = factorial(n)
        seconds = ops / ops_per_sec
        time_str = f"{seconds:.2f} sec" if seconds < 60 else f"{seconds/60:.1f} min" if seconds < 3600 else f"{seconds/3600:.1f} hr" if seconds < 86400 else f"{seconds/86400:.1f} days"
        print(f"  n={n:2}: {ops:>15,} operations ({time_str})")

    # Exponential growth
    print("\n2^n growth (all subsets):")
    for n in [20, 30, 40, 50]:
        ops = 2 ** n
        seconds = ops / ops_per_sec
        time_str = f"{seconds:.2f} sec" if seconds < 60 else f"{seconds/60:.1f} min" if seconds < 3600 else f"{seconds/3600:.1f} hr" if seconds < 86400 else f"{seconds/86400:.0f} days"
        print(f"  n={n:2}: {ops:>15,} operations ({time_str})")


practical_limits()
```

---

## Summary

| Concept | Korean | Formula |
|---------|--------|---------|
| Permutation | 순열 | P(n,r) = n!/(n-r)! |
| Combination | 조합 | C(n,r) = n!/(r!(n-r)!) |
| Factorial | 팩토리얼 | n! = n × (n-1) × ... × 1 |
| With Repetition | 중복 | Perm: n^r, Comb: C(n+r-1,r) |
| Multiset Perm | 다중집합 | n!/(n₁!×n₂!×...) |

**Key difference:** Order matters (permutation) vs doesn't matter (combination).

---

## Practice Problems

1. How many 3-digit numbers using {1,2,3,4,5} if digits can repeat? Cannot repeat?

2. Choose a committee of 4 from 10 people. How many ways?

3. How many arrangements of "BANANA"?

4. Distribute 10 identical balls into 3 distinct boxes. How many ways?
