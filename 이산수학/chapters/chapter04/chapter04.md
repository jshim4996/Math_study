# Chapter 04. Set Operations (집합 연산)

> Mastering set operations for data manipulation, algorithm design, and understanding database operations.

---

## Union (합집합)

### Concept (개념)

The **union** (합집합) of sets A and B contains all elements that are in A, in B, or in both.

**Notation:** A ∪ B

**Properties:**
- Commutative: A ∪ B = B ∪ A
- Associative: (A ∪ B) ∪ C = A ∪ (B ∪ C)
- Identity: A ∪ ∅ = A
- Idempotent: A ∪ A = A
- Domination: A ∪ U = U

### Example

```python
# Union in Python
A = {1, 2, 3}
B = {3, 4, 5}

# Multiple ways to compute union
union1 = A | B              # Using operator
union2 = A.union(B)         # Using method
union3 = {*A, *B}           # Using unpacking (creates new set)

print(union1)  # {1, 2, 3, 4, 5}

# Union of multiple sets
C = {5, 6, 7}
multi_union = A | B | C
print(multi_union)  # {1, 2, 3, 4, 5, 6, 7}

# Practical: Combining user permissions
read_permissions = {"read_posts", "read_comments"}
write_permissions = {"write_posts", "write_comments"}
admin_permissions = {"delete_users", "manage_settings"}

moderator = read_permissions | write_permissions
print(f"Moderator can: {moderator}")

full_admin = read_permissions | write_permissions | admin_permissions
print(f"Admin can: {full_admin}")
```

---

## Intersection (교집합)

### Concept (개념)

The **intersection** (교집합) of sets A and B contains only elements that are in both A and B.

**Notation:** A ∩ B

**Properties:**
- Commutative: A ∩ B = B ∩ A
- Associative: (A ∩ B) ∩ C = A ∩ (B ∩ C)
- Identity: A ∩ U = A
- Idempotent: A ∩ A = A
- Domination: A ∩ ∅ = ∅

**Disjoint Sets** (서로소 집합): A and B are disjoint if A ∩ B = ∅

### Example

```python
# Intersection in Python
A = {1, 2, 3, 4}
B = {3, 4, 5, 6}

# Multiple ways
inter1 = A & B                  # Using operator
inter2 = A.intersection(B)      # Using method

print(inter1)  # {3, 4}

# Checking for disjoint sets
C = {7, 8, 9}
print(A & C)                    # set() - empty, they're disjoint
print(A.isdisjoint(C))          # True

# Practical: Finding common skills
frontend_skills = {"JavaScript", "React", "CSS", "HTML"}
backend_skills = {"Python", "JavaScript", "SQL", "Docker"}

full_stack_required = frontend_skills & backend_skills
print(f"Common skills: {full_stack_required}")  # {"JavaScript"}

# Finding users active on all platforms
twitter_users = {"alice", "bob", "charlie", "diana"}
instagram_users = {"bob", "diana", "eve", "frank"}
linkedin_users = {"charlie", "diana", "bob", "grace"}

all_platform_users = twitter_users & instagram_users & linkedin_users
print(f"Active on all: {all_platform_users}")  # {"bob", "diana"}
```

---

## Set Difference (차집합)

### Concept (개념)

The **difference** (차집합) A - B (or A \ B) contains elements in A that are not in B.

**Notation:** A - B or A \ B

**Properties:**
- NOT commutative: A - B ≠ B - A (generally)
- A - ∅ = A
- A - A = ∅
- A - B = A ∩ B' (where B' is complement of B)

### Example

```python
# Difference in Python
A = {1, 2, 3, 4, 5}
B = {4, 5, 6, 7}

diff1 = A - B               # Using operator
diff2 = A.difference(B)     # Using method

print(f"A - B = {diff1}")   # {1, 2, 3}
print(f"B - A = {B - A}")   # {6, 7}

# Practical: Finding missing items
required_ingredients = {"flour", "sugar", "eggs", "butter", "milk"}
available = {"flour", "eggs", "milk"}

shopping_list = required_ingredients - available
print(f"Need to buy: {shopping_list}")  # {"sugar", "butter"}

# Removing banned users
all_users = {"user1", "user2", "user3", "user4", "user5"}
banned_users = {"user2", "user4"}

active_users = all_users - banned_users
print(f"Active users: {active_users}")  # {"user1", "user3", "user5"}
```

---

## Symmetric Difference (대칭차)

### Concept (개념)

The **symmetric difference** (대칭차) contains elements in either A or B, but not in both.

**Notation:** A △ B or A ⊕ B

**Equivalent to:** (A - B) ∪ (B - A) = (A ∪ B) - (A ∩ B)

**Properties:**
- Commutative: A △ B = B △ A
- Associative: (A △ B) △ C = A △ (B △ C)
- A △ ∅ = A
- A △ A = ∅

This operation is like XOR for sets!

### Example

```python
# Symmetric difference in Python
A = {1, 2, 3, 4}
B = {3, 4, 5, 6}

sym_diff1 = A ^ B                          # Using operator
sym_diff2 = A.symmetric_difference(B)      # Using method
sym_diff3 = (A - B) | (B - A)              # Using definition

print(sym_diff1)  # {1, 2, 5, 6}

# Verifying equivalence
print(sym_diff1 == (A | B) - (A & B))  # True

# Practical: Finding differences between versions
old_features = {"login", "search", "profile", "chat"}
new_features = {"login", "search", "dashboard", "notifications"}

# What changed?
changed = old_features ^ new_features
print(f"Changed features: {changed}")
# {"profile", "chat", "dashboard", "notifications"}

# Broken down:
removed = old_features - new_features  # {"profile", "chat"}
added = new_features - old_features    # {"dashboard", "notifications"}
```

---

## Complement (여집합)

### Concept (개념)

The **complement** (여집합) of set A contains all elements in the universal set U that are not in A.

**Notation:** A' or Ā or A^c

**Properties:**
- (A')' = A (Double complement)
- A ∪ A' = U
- A ∩ A' = ∅
- U' = ∅
- ∅' = U

### Example

```python
# Complement requires defining universal set
U = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
A = {2, 4, 6, 8, 10}  # Even numbers

A_complement = U - A
print(f"A' = {A_complement}")  # {1, 3, 5, 7, 9} - odd numbers

# Verifying properties
print(f"A ∪ A' = {A | A_complement}")      # {1,2,3,4,5,6,7,8,9,10} = U
print(f"A ∩ A' = {A & A_complement}")      # set() = ∅
print(f"(A')' = {U - A_complement}")       # {2,4,6,8,10} = A

# Practical: Boolean flag context
all_users = {"user1", "user2", "user3", "user4", "user5"}
premium_users = {"user2", "user4"}

free_users = all_users - premium_users  # Complement
print(f"Free users: {free_users}")  # {"user1", "user3", "user5"}
```

---

## De Morgan's Laws for Sets (집합에서의 드모르간 법칙)

### Concept (개념)

De Morgan's Laws apply to sets just as they do to logic:

1. **(A ∪ B)' = A' ∩ B'** : Complement of union = Intersection of complements
2. **(A ∩ B)' = A' ∪ B'** : Complement of intersection = Union of complements

These laws are crucial for:
- Simplifying complex set expressions
- Database query optimization
- Understanding negation in queries

### Example

```python
# Verifying De Morgan's Laws for sets
U = set(range(1, 11))  # {1, 2, 3, ..., 10}
A = {1, 2, 3, 4}
B = {3, 4, 5, 6}

# Law 1: (A ∪ B)' = A' ∩ B'
left1 = U - (A | B)           # (A ∪ B)'
right1 = (U - A) & (U - B)    # A' ∩ B'
print(f"(A ∪ B)' = {left1}")
print(f"A' ∩ B' = {right1}")
print(f"Equal: {left1 == right1}")  # True

# Law 2: (A ∩ B)' = A' ∪ B'
left2 = U - (A & B)           # (A ∩ B)'
right2 = (U - A) | (U - B)    # A' ∪ B'
print(f"(A ∩ B)' = {left2}")
print(f"A' ∪ B' = {right2}")
print(f"Equal: {left2 == right2}")  # True
```

---

## Database Operations Relation (데이터베이스 연산과의 관계)

### Concept (개념)

Set operations directly correspond to SQL operations:

| Set Operation | SQL Equivalent |
|---------------|----------------|
| A ∪ B | UNION |
| A ∩ B | INTERSECT |
| A - B | EXCEPT / MINUS |
| A ⊆ B | Subquery with NOT EXISTS |

Understanding sets helps in:
- Writing efficient queries
- Optimizing database operations
- Understanding query execution plans

### Example

```python
# Simulating database operations
# Table: Customers who bought products

electronics_buyers = {"Alice", "Bob", "Charlie", "Diana"}
book_buyers = {"Bob", "Eve", "Charlie", "Frank"}
clothing_buyers = {"Alice", "Charlie", "Grace", "Eve"}

# SQL: SELECT * FROM electronics_buyers UNION SELECT * FROM book_buyers
all_tech_or_book = electronics_buyers | book_buyers
print(f"Bought electronics OR books: {all_tech_or_book}")

# SQL: SELECT * FROM electronics_buyers INTERSECT SELECT * FROM book_buyers
both_categories = electronics_buyers & book_buyers
print(f"Bought electronics AND books: {both_categories}")  # {"Bob", "Charlie"}

# SQL: SELECT * FROM electronics_buyers EXCEPT SELECT * FROM book_buyers
only_electronics = electronics_buyers - book_buyers
print(f"Bought ONLY electronics: {only_electronics}")  # {"Alice", "Diana"}

# Complex query: Customers who bought from all 3 categories
vip_customers = electronics_buyers & book_buyers & clothing_buyers
print(f"VIP (all categories): {vip_customers}")  # {"Charlie"}

# Customers who bought electronics but never clothing
target_market = electronics_buyers - clothing_buyers
print(f"Target for clothing ads: {target_market}")  # {"Bob", "Diana"}
```

---

## Cartesian Product (곱집합)

### Concept (개념)

The **Cartesian product** (곱집합) A × B is the set of all ordered pairs (a, b) where a ∈ A and b ∈ B.

**Notation:** A × B = {(a, b) | a ∈ A and b ∈ B}

**Properties:**
- |A × B| = |A| × |B|
- NOT commutative: A × B ≠ B × A (unless A = B)
- A × ∅ = ∅

### Example

```python
from itertools import product

# Cartesian product
A = {1, 2, 3}
B = {"a", "b"}

# Manual implementation
cart_prod = {(a, b) for a in A for b in B}
print(cart_prod)
# {(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b'), (3, 'a'), (3, 'b')}

# Using itertools
cart_prod2 = set(product(A, B))
print(cart_prod2)  # Same result

# Size verification
print(f"|A × B| = {len(cart_prod)}")  # 6 = 3 × 2

# Practical: All possible configurations
sizes = {"S", "M", "L"}
colors = {"Red", "Blue", "Green"}

all_variants = set(product(sizes, colors))
print(f"Total product variants: {len(all_variants)}")  # 9

# Database: This is like a CROSS JOIN
# SELECT * FROM sizes CROSS JOIN colors
```

---

## Summary

| Operation | Korean | Symbol | Python |
|-----------|--------|--------|--------|
| Union | 합집합 | A ∪ B | `A \| B` or `A.union(B)` |
| Intersection | 교집합 | A ∩ B | `A & B` or `A.intersection(B)` |
| Difference | 차집합 | A - B | `A - B` or `A.difference(B)` |
| Symmetric Diff | 대칭차 | A △ B | `A ^ B` or `A.symmetric_difference(B)` |
| Complement | 여집합 | A' | `U - A` |
| Cartesian Product | 곱집합 | A × B | `itertools.product(A, B)` |
