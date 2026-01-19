# Chapter 05. Relations (관계)

> Understanding mathematical relations as the foundation for database design, graph structures, and object relationships in programming.

---

## What is a Relation? (관계란?)

### Concept (개념)

A **relation** (관계) R from set A to set B is a subset of the Cartesian product A × B. In other words, R ⊆ A × B.

If (a, b) ∈ R, we say "a is related to b" and write aRb.

**Types:**
- **Binary relation** (이항관계): Between two sets (most common)
- **n-ary relation**: Between n sets (used in databases)
- **Relation on a set**: R from A to A (R ⊆ A × A)

### Example

```python
# Defining a relation "less than" on {1, 2, 3}
A = {1, 2, 3}

# R = {(a, b) | a < b}
R = {(a, b) for a in A for b in A if a < b}
print(f"R = {R}")  # {(1, 2), (1, 3), (2, 3)}

# Checking if elements are related
def is_related(a, b, relation):
    return (a, b) in relation

print(is_related(1, 3, R))  # True (1 R 3, since 1 < 3)
print(is_related(3, 1, R))  # False (3 is not less than 1)

# Real-world: Student-Course enrollment relation
students = {"Alice", "Bob", "Charlie"}
courses = {"Math", "CS", "Physics"}

enrollment = {
    ("Alice", "Math"), ("Alice", "CS"),
    ("Bob", "CS"), ("Bob", "Physics"),
    ("Charlie", "Math"), ("Charlie", "Physics")
}
# This is a relation from students to courses
```

---

## Ordered Pairs and Cartesian Product (순서쌍과 곱집합)

### Concept (개념)

An **ordered pair** (순서쌍) (a, b) is a pair where order matters: (a, b) ≠ (b, a) unless a = b.

The **Cartesian product** (곱집합) A × B = {(a, b) | a ∈ A, b ∈ B} is the set of all ordered pairs.

A relation R is simply a subset: R ⊆ A × B

### Example

```python
# Ordered pairs - order matters
pair1 = (1, 2)
pair2 = (2, 1)
print(pair1 == pair2)  # False

# Cartesian product as foundation for relations
A = {"a", "b"}
B = {1, 2}

# All possible pairs
all_pairs = {(a, b) for a in A for b in B}
print(all_pairs)  # {('a', 1), ('a', 2), ('b', 1), ('b', 2)}

# A relation is a subset of this
# Example: "a maps to 1, b maps to 2"
R = {("a", 1), ("b", 2)}
print(R.issubset(all_pairs))  # True

# This is the foundation of functions!
# A function is a special type of relation
```

---

## Matrix Representation (행렬 표현)

### Concept (개념)

A binary relation R from A = {a₁, ..., aₘ} to B = {b₁, ..., bₙ} can be represented as an m × n matrix M where:

M[i][j] = 1 if (aᵢ, bⱼ) ∈ R
M[i][j] = 0 otherwise

This is called the **relation matrix** (관계 행렬) or adjacency matrix.

### Example

```python
import numpy as np

# Define sets and relation
A = ["a", "b", "c"]  # rows
B = [1, 2, 3]        # columns

R = {("a", 1), ("a", 2), ("b", 2), ("c", 3)}

# Create relation matrix
def relation_to_matrix(A, B, R):
    m, n = len(A), len(B)
    M = np.zeros((m, n), dtype=int)

    for i, a in enumerate(A):
        for j, b in enumerate(B):
            if (a, b) in R:
                M[i][j] = 1
    return M

M = relation_to_matrix(A, B, R)
print("Relation Matrix:")
print(M)
# [[1 1 0]
#  [0 1 0]
#  [0 0 1]]

# Reading the matrix:
# Row "a": related to 1 and 2
# Row "b": related to 2 only
# Row "c": related to 3 only

# Convert matrix back to relation
def matrix_to_relation(A, B, M):
    R = set()
    for i, a in enumerate(A):
        for j, b in enumerate(B):
            if M[i][j] == 1:
                R.add((a, b))
    return R

print(matrix_to_relation(A, B, M))  # Original relation
```

---

## Graph Representation (그래프 표현)

### Concept (개념)

A relation R on set A can be visualized as a **directed graph** (유향 그래프):
- Each element in A is a **node** (정점)
- Each pair (a, b) ∈ R is a directed **edge** (간선) from a to b
- Self-loops occur when (a, a) ∈ R

This representation makes properties like reflexivity and transitivity visually apparent.

### Example

```python
# Relation as a directed graph
A = {1, 2, 3, 4}
R = {(1, 2), (2, 3), (3, 1), (1, 1), (4, 4)}

# Adjacency list representation
def relation_to_adjacency_list(R):
    adj = {}
    for (a, b) in R:
        if a not in adj:
            adj[a] = []
        adj[a].append(b)
    return adj

adj = relation_to_adjacency_list(R)
print("Adjacency List:")
for node, neighbors in adj.items():
    print(f"  {node} -> {neighbors}")

# Output:
# 1 -> [2, 1]  (1 relates to 2 and itself)
# 2 -> [3]
# 3 -> [1]
# 4 -> [4]     (self-loop)

# Visual representation:
"""
    ┌─────────────┐
    ↓             │
   (1) ──→ (2) ──→ (3)
    ↻

   (4)
    ↻
"""
```

---

## Properties of Relations (관계의 성질)

### Concept (개념)

Relations on a set can have special properties:

| Property | Definition | Matrix Check |
|----------|------------|--------------|
| **Reflexive** (반사적) | ∀a ∈ A: (a, a) ∈ R | All diagonal = 1 |
| **Symmetric** (대칭적) | (a, b) ∈ R → (b, a) ∈ R | M = M^T |
| **Antisymmetric** (반대칭적) | (a, b) ∈ R ∧ (b, a) ∈ R → a = b | Off-diag: M[i,j]=1 → M[j,i]=0 |
| **Transitive** (추이적) | (a, b) ∈ R ∧ (b, c) ∈ R → (a, c) ∈ R | M² implies M |

### Example

```python
# Check all properties
def check_reflexive(A, R):
    """Every element relates to itself"""
    return all((a, a) in R for a in A)

def check_symmetric(R):
    """If (a,b) in R, then (b,a) in R"""
    return all((b, a) in R for (a, b) in R)

def check_antisymmetric(R):
    """If (a,b) and (b,a) in R, then a = b"""
    for (a, b) in R:
        if a != b and (b, a) in R:
            return False
    return True

def check_transitive(R):
    """If (a,b) and (b,c) in R, then (a,c) in R"""
    for (a, b) in R:
        for (b2, c) in R:
            if b == b2 and (a, c) not in R:
                return False
    return True

# Example: "divides" relation on {1, 2, 4}
A = {1, 2, 4}
divides = {(a, b) for a in A for b in A if b % a == 0}
print(f"Divides relation: {divides}")
# {(1, 1), (1, 2), (1, 4), (2, 2), (2, 4), (4, 4)}

print(f"Reflexive: {check_reflexive(A, divides)}")      # True
print(f"Symmetric: {check_symmetric(divides)}")         # False
print(f"Antisymmetric: {check_antisymmetric(divides)}") # True
print(f"Transitive: {check_transitive(divides)}")       # True
```

---

## Equivalence Relations (동치관계)

### Concept (개념)

An **equivalence relation** (동치관계) is a relation that is:
1. **Reflexive** (반사적)
2. **Symmetric** (대칭적)
3. **Transitive** (추이적)

Equivalence relations partition a set into **equivalence classes** (동치류). Elements in the same class are "equivalent" to each other.

**Examples:**
- Equality (=)
- Same remainder when divided by n (congruence mod n)
- Same file type

### Example

```python
# Equivalence relation: congruence mod 3
A = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}

def congruent_mod_3(a, b):
    return a % 3 == b % 3

# Build the relation
R = {(a, b) for a in A for b in A if congruent_mod_3(a, b)}

print(f"Reflexive: {check_reflexive(A, R)}")   # True
print(f"Symmetric: {check_symmetric(R)}")      # True
print(f"Transitive: {check_transitive(R)}")    # True

# Find equivalence classes
def find_equivalence_classes(A, R):
    classes = {}
    for a in A:
        representative = min(b for b in A if (a, b) in R)
        if representative not in classes:
            classes[representative] = set()
        classes[representative].add(a)
    return list(classes.values())

eq_classes = find_equivalence_classes(A, R)
print(f"Equivalence classes: {eq_classes}")
# [{0, 3, 6, 9}, {1, 4, 7}, {2, 5, 8}]

# Practical: Grouping files by extension
files = ["doc1.txt", "img.png", "doc2.txt", "photo.png", "data.csv"]

def same_extension(f1, f2):
    return f1.split('.')[-1] == f2.split('.')[-1]

# This defines an equivalence relation on files
```

---

## Partial Order Relations (부분 순서 관계)

### Concept (개념)

A **partial order** (부분순서) is a relation that is:
1. **Reflexive** (반사적)
2. **Antisymmetric** (반대칭적)
3. **Transitive** (추이적)

A set with a partial order is called a **poset** (partially ordered set, 부분순서집합).

**Examples:**
- ≤ on numbers
- ⊆ on sets
- Divides relation on integers
- Inheritance hierarchy in OOP

### Example

```python
# Partial order: subset relation on power set
A = {1, 2}
power_set = [set(), {1}, {2}, {1, 2}]

subset_relation = {
    (frozenset(a), frozenset(b))
    for a in power_set
    for b in power_set
    if set(a).issubset(set(b))
}

print("Subset relation (⊆):")
for (a, b) in sorted(subset_relation, key=lambda x: (len(x[0]), len(x[1]))):
    print(f"  {set(a)} ⊆ {set(b)}")

# Hasse diagram representation (simplified)
"""
      {1, 2}
       /  \
     {1}  {2}
       \  /
        {}
"""

# Practical: Task dependencies (partial order)
tasks = ["A", "B", "C", "D"]
# B depends on A, C depends on A, D depends on B and C
dependencies = {("A", "A"), ("B", "B"), ("C", "C"), ("D", "D"),
                ("A", "B"), ("A", "C"), ("A", "D"),
                ("B", "D"), ("C", "D")}

# Topological sort gives a valid execution order
# Result: A, B, C, D or A, C, B, D
```

---

## Closure of Relations (관계의 폐포)

### Concept (개념)

The **closure** (폐포) of a relation R with respect to a property P is the smallest relation containing R that has property P.

- **Reflexive closure**: Add (a, a) for all a
- **Symmetric closure**: Add (b, a) for every (a, b)
- **Transitive closure**: Add (a, c) when path from a to c exists

Transitive closure is particularly important for reachability in graphs.

### Example

```python
def reflexive_closure(A, R):
    """Add all (a, a) pairs"""
    return R | {(a, a) for a in A}

def symmetric_closure(R):
    """Add (b, a) for every (a, b)"""
    return R | {(b, a) for (a, b) in R}

def transitive_closure(R):
    """Warshall's algorithm"""
    closure = set(R)
    changed = True
    while changed:
        changed = False
        new_pairs = set()
        for (a, b) in closure:
            for (b2, c) in closure:
                if b == b2 and (a, c) not in closure:
                    new_pairs.add((a, c))
                    changed = True
        closure |= new_pairs
    return closure

# Example
A = {1, 2, 3, 4}
R = {(1, 2), (2, 3), (3, 4)}

print(f"Original: {R}")
print(f"Reflexive closure: {reflexive_closure(A, R)}")
print(f"Symmetric closure: {symmetric_closure(R)}")
print(f"Transitive closure: {transitive_closure(R)}")
# Transitive: {(1,2), (2,3), (3,4), (1,3), (2,4), (1,4)}

# Practical: Reachability in a graph
# "Can I get from node 1 to node 4?"
# Check if (1, 4) is in transitive closure
```

---

## Summary

| Concept | Korean | Key Point |
|---------|--------|-----------|
| Relation | 관계 | R ⊆ A × B |
| Ordered Pair | 순서쌍 | (a, b) ≠ (b, a) |
| Relation Matrix | 관계 행렬 | M[i][j] = 1 if related |
| Reflexive | 반사적 | (a, a) ∈ R for all a |
| Symmetric | 대칭적 | (a,b) ∈ R → (b,a) ∈ R |
| Antisymmetric | 반대칭적 | (a,b),(b,a) ∈ R → a=b |
| Transitive | 추이적 | (a,b),(b,c) ∈ R → (a,c) ∈ R |
| Equivalence Relation | 동치관계 | Ref + Sym + Trans |
| Partial Order | 부분순서 | Ref + Antisym + Trans |
| Transitive Closure | 추이적 폐포 | Reachability |
