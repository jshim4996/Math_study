# Chapter 03. Set Basics (집합 기초)

> Understanding sets as the fundamental structure for organizing and categorizing data in mathematics and computer science.

---

## What is a Set? (집합이란?)

### Concept (개념)

A **set** (집합) is a well-defined collection of distinct objects called **elements** (원소) or members. Sets are foundational in mathematics and directly map to data structures in programming.

**Key properties:**
- **Unordered**: {1, 2, 3} = {3, 1, 2}
- **No duplicates**: {1, 1, 2} = {1, 2}
- **Well-defined**: Clear criteria for membership

**Notation:**
- Capital letters for sets: A, B, C
- Lowercase for elements: a, b, c
- Membership: a ∈ A (a is in A), b ∉ A (b is not in A)

### Example

```python
# Sets in Python
fruits = {"apple", "banana", "cherry"}

# Check membership
print("apple" in fruits)   # True (apple ∈ fruits)
print("grape" in fruits)   # False (grape ∉ fruits)

# Sets automatically remove duplicates
numbers = {1, 2, 2, 3, 3, 3}
print(numbers)  # {1, 2, 3}

# Order doesn't matter
set_a = {1, 2, 3}
set_b = {3, 1, 2}
print(set_a == set_b)  # True
```

---

## Set Notation Methods (집합 표기법)

### Concept (개념)

There are three main ways to describe a set:

1. **Roster/Enumeration Method** (원소나열법):
   - List all elements: A = {1, 2, 3, 4, 5}
   - Use ellipsis for patterns: B = {2, 4, 6, ..., 100}

2. **Set-Builder Notation** (조건제시법):
   - Describe properties: A = {x | x is a positive integer less than 6}
   - Read as "the set of all x such that..."

3. **Descriptive Method** (서술식):
   - Natural language: "The set of all prime numbers less than 10"

### Example

```python
# Roster method
evens_roster = {2, 4, 6, 8, 10}

# Set-builder notation equivalent in Python
evens_builder = {x for x in range(1, 11) if x % 2 == 0}

# Both produce the same set
print(evens_roster == evens_builder)  # True

# Complex set-builder
# A = {x² | x ∈ Z, 1 ≤ x ≤ 5}
squares = {x**2 for x in range(1, 6)}
print(squares)  # {1, 4, 9, 16, 25}

# Filtering with conditions
# B = {x | x ∈ N, x < 20, x is prime}
def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n**0.5) + 1):
        if n % i == 0:
            return False
    return True

primes_under_20 = {x for x in range(20) if is_prime(x)}
print(primes_under_20)  # {2, 3, 5, 7, 11, 13, 17, 19}
```

---

## Special Sets (특수 집합)

### Concept (개념)

**Empty Set** (공집합): A set with no elements
- Notation: ∅ or {}
- The empty set is a subset of every set

**Universal Set** (전체집합): The set containing all elements under consideration
- Notation: U
- Context-dependent (e.g., all integers, all students, etc.)

**Common Number Sets:**
- **N** (자연수): Natural numbers {1, 2, 3, ...} or {0, 1, 2, ...}
- **Z** (정수): Integers {..., -2, -1, 0, 1, 2, ...}
- **Q** (유리수): Rational numbers
- **R** (실수): Real numbers

### Example

```python
# Empty set
empty = set()  # Note: {} creates empty dict, not set!
print(len(empty))  # 0
print(type(empty))  # <class 'set'>

# Empty set is subset of everything
A = {1, 2, 3}
print(empty.issubset(A))  # True
print(set().issubset(set()))  # True

# Universal set (context-dependent)
# For a dice game, U = {1, 2, 3, 4, 5, 6}
universal_dice = {1, 2, 3, 4, 5, 6}
even_rolls = {2, 4, 6}
odd_rolls = {1, 3, 5}

# Complement relative to universal set
complement_of_even = universal_dice - even_rolls
print(complement_of_even)  # {1, 3, 5} = odd_rolls
```

---

## Subsets (부분집합)

### Concept (개념)

Set A is a **subset** (부분집합) of B if every element of A is also in B.
- **Subset**: A ⊆ B (A is subset of B, possibly equal)
- **Proper subset**: A ⊂ B (A is subset but A ≠ B)

**Properties:**
- Every set is a subset of itself: A ⊆ A
- Empty set is subset of everything: ∅ ⊆ A
- If A ⊆ B and B ⊆ A, then A = B

**Power Set** (멱집합): The set of all subsets of A
- Notation: P(A) or 2^A
- If |A| = n, then |P(A)| = 2^n

### Example

```python
# Subset checking
A = {1, 2}
B = {1, 2, 3, 4}

print(A.issubset(B))      # True (A ⊆ B)
print(A <= B)              # True (alternative syntax)
print(A < B)               # True (proper subset, A ⊂ B)

# Equality through mutual subsets
C = {1, 2}
print(A <= C and C <= A)   # True (A = C)
print(A == C)              # True

# Power set implementation
def power_set(s):
    """Generate all subsets of a set"""
    s = list(s)
    n = len(s)
    result = []

    # Use binary counting: 2^n subsets
    for i in range(2**n):
        subset = set()
        for j in range(n):
            if i & (1 << j):  # Check if j-th bit is set
                subset.add(s[j])
        result.append(subset)

    return result

A = {1, 2, 3}
P_A = power_set(A)
print(f"|A| = {len(A)}, |P(A)| = {len(P_A)}")  # |A| = 3, |P(A)| = 8
print(P_A)
# [set(), {1}, {2}, {1, 2}, {3}, {1, 3}, {2, 3}, {1, 2, 3}]
```

---

## Venn Diagrams (벤 다이어그램)

### Concept (개념)

A **Venn diagram** (벤 다이어그램) is a visual representation of sets using circles within a rectangle (universal set). It helps visualize:
- Set relationships (subset, disjoint)
- Set operations (union, intersection)
- Logical relationships

**Common configurations:**
- One circle: Single set within universal set
- Two circles: Shows overlap (intersection) and unique parts
- Three circles: More complex relationships

### Example

```text
Venn Diagram for A = {1, 2, 3}, B = {2, 3, 4}, U = {1, 2, 3, 4, 5}

    ┌───────────────────────────────┐
    │              U                │
    │    ┌─────────────────┐        │
    │    │   A       ┌─────┼───┐    │
    │    │   {1}     │{2,3}│{4}│ B  │
    │    │           └─────┼───┘    │
    │    └─────────────────┘        │
    │                          {5}  │
    └───────────────────────────────┘

- A only: {1}
- B only: {4}
- A ∩ B: {2, 3}
- Neither: {5}
```

```python
# Visualizing set relationships with code
A = {1, 2, 3}
B = {2, 3, 4}
U = {1, 2, 3, 4, 5}

only_A = A - B                    # {1}
only_B = B - A                    # {4}
both = A & B                      # {2, 3}
neither = U - (A | B)             # {5}

print(f"Only in A: {only_A}")
print(f"Only in B: {only_B}")
print(f"In both A and B: {both}")
print(f"In neither: {neither}")
```

---

## Cardinality (집합의 크기)

### Concept (개념)

The **cardinality** (집합의 크기/원소의 개수) of a set is the number of elements it contains.
- Notation: |A| or n(A) or #A
- Empty set: |∅| = 0

**Finite vs Infinite sets:**
- **Finite** (유한집합): Countable number of elements
- **Infinite** (무한집합): Unlimited elements (e.g., natural numbers)

### Example

```python
# Cardinality examples
A = {1, 2, 3, 4, 5}
B = set()
C = {"apple", "banana"}

print(f"|A| = {len(A)}")  # 5
print(f"|B| = {len(B)}")  # 0 (empty set)
print(f"|C| = {len(C)}")  # 2

# Cardinality and power set relationship
# |P(A)| = 2^|A|
for n in range(5):
    test_set = set(range(n))
    power_size = 2 ** n
    print(f"|A| = {n}, |P(A)| = {power_size}")

# Output:
# |A| = 0, |P(A)| = 1
# |A| = 1, |P(A)| = 2
# |A| = 2, |P(A)| = 4
# |A| = 3, |P(A)| = 8
# |A| = 4, |P(A)| = 16
```

---

## Sets in Programming (프로그래밍에서의 집합)

### Concept (개념)

Sets are used extensively in programming for:
- **Deduplication**: Removing duplicates from lists
- **Membership testing**: O(1) average lookup time
- **Mathematical operations**: Union, intersection, etc.
- **Database operations**: Basis for SQL operations

### Example

```python
# Deduplication
items = [1, 2, 2, 3, 3, 3, 4, 4, 4, 4]
unique_items = list(set(items))  # [1, 2, 3, 4]

# Fast membership testing
valid_ids = {1001, 1002, 1003, 1004, 1005}

def is_valid_id(id):
    return id in valid_ids  # O(1) average

# vs list (O(n))
valid_ids_list = [1001, 1002, 1003, 1004, 1005]

def is_valid_id_slow(id):
    return id in valid_ids_list  # O(n)

# Finding common elements
users_monday = {"Alice", "Bob", "Charlie"}
users_tuesday = {"Bob", "Diana", "Charlie"}
users_wednesday = {"Charlie", "Eve", "Bob"}

# Users who came all three days
loyal_users = users_monday & users_tuesday & users_wednesday
print(loyal_users)  # {"Bob", "Charlie"}

# Users who came at least once
all_users = users_monday | users_tuesday | users_wednesday
print(all_users)  # {"Alice", "Bob", "Charlie", "Diana", "Eve"}
```

---

## Summary

| Concept | Korean | Symbol/Notation |
|---------|--------|-----------------|
| Set | 집합 | A, B, C |
| Element | 원소 | a ∈ A |
| Not element | 원소가 아님 | a ∉ A |
| Empty set | 공집합 | ∅ or {} |
| Universal set | 전체집합 | U |
| Subset | 부분집합 | A ⊆ B |
| Proper subset | 진부분집합 | A ⊂ B |
| Power set | 멱집합 | P(A), 2^A |
| Cardinality | 원소의 개수 | \|A\| |
