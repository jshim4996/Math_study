# Chapter 12. Pigeonhole and Inclusion-Exclusion (비둘기집 원리와 포함-배제)

> Powerful counting principles for proving existence and counting overlapping sets.

---

## Pigeonhole Principle (비둘기집 원리)

### Concept (개념)

**Pigeonhole Principle:** If n items are put into m containers where n > m, then at least one container has more than one item.

**Generalized form:** If n items are in m containers, at least one container has ⌈n/m⌉ items.

**Uses:**
- Prove existence (not find the specific case)
- Show that collisions must occur
- Prove properties about sequences

### Example

```python
import math

def pigeonhole_basic():
    """Basic pigeonhole examples"""
    print("Pigeonhole Principle Examples:\n")

    # Basic example
    print("1. Socks in a drawer:")
    print("   10 blue socks, 10 red socks, 10 green socks")
    print("   Pull socks in dark. How many to guarantee a matching pair?")
    colors = 3
    for_pair = colors + 1
    print(f"   Answer: {for_pair} socks (worst case: one of each, then next matches)")

    # Birthday problem
    print("\n2. Birthday guarantee:")
    days = 366
    people_for_match = days + 1
    print(f"   {people_for_match} people guarantee same birthday")

    # Students and grades
    print("\n3. Grades distribution:")
    students = 30
    grades = 5  # A, B, C, D, F
    min_same = math.ceil(students / grades)
    print(f"   {students} students, {grades} possible grades")
    print(f"   At least {min_same} students have same grade")


pigeonhole_basic()


def pigeonhole_generalized():
    """Generalized pigeonhole"""
    print("\nGeneralized Pigeonhole:")

    # Formula: ⌈n/m⌉ items in at least one container
    def min_in_one_container(items, containers):
        return math.ceil(items / containers)

    # Examples
    cases = [
        (100, 12, "students", "months"),  # Birthdays
        (50, 7, "emails", "days"),
        (25, 4, "cards", "suits"),
    ]

    for n, m, item_name, container_name in cases:
        result = min_in_one_container(n, m)
        print(f"  {n} {item_name} in {m} {container_name}: at least {result} in one")


pigeonhole_generalized()


def pigeonhole_hash_collision():
    """Hash collision guarantee"""
    print("\nHash Table Collision (Pigeonhole):")

    # Hash table example
    table_size = 1000
    items = 1001

    print(f"  Table size: {table_size}")
    print(f"  Items to insert: {items}")
    print(f"  Collision GUARANTEED (items > slots)")

    # More realistic
    table_size = 1000
    items = 100
    # No guarantee, but probability high

    # For guarantee of k items in same bucket
    def items_for_k_collision(table_size, k):
        return table_size * (k - 1) + 1

    print(f"\n  For guaranteed collision (2+ same bucket):")
    print(f"    Need {items_for_k_collision(table_size, 2)} items")

    print(f"  For guaranteed 3+ same bucket:")
    print(f"    Need {items_for_k_collision(table_size, 3)} items")


pigeonhole_hash_collision()
```

---

## Inclusion-Exclusion Principle (포함-배제 원리)

### Concept (개념)

Count elements in **union** of overlapping sets by adding and subtracting:

**Two sets:**
```
|A ∪ B| = |A| + |B| - |A ∩ B|
```

**Three sets:**
```
|A ∪ B ∪ C| = |A| + |B| + |C|
            - |A ∩ B| - |A ∩ C| - |B ∩ C|
            + |A ∩ B ∩ C|
```

**General pattern:** Add singles, subtract pairs, add triples, subtract quadruples...

### Example

```python
def inclusion_exclusion_2():
    """Two set inclusion-exclusion"""
    print("Inclusion-Exclusion (Two Sets):\n")

    # Survey example
    total_students = 100
    likes_math = 60
    likes_science = 45
    likes_both = 20

    # Students who like math OR science
    likes_either = likes_math + likes_science - likes_both

    # Students who like neither
    likes_neither = total_students - likes_either

    print("Survey: Math and Science")
    print(f"  Total students: {total_students}")
    print(f"  Like Math: {likes_math}")
    print(f"  Like Science: {likes_science}")
    print(f"  Like Both: {likes_both}")
    print(f"  Like Either: {likes_math} + {likes_science} - {likes_both} = {likes_either}")
    print(f"  Like Neither: {total_students} - {likes_either} = {likes_neither}")


inclusion_exclusion_2()


def inclusion_exclusion_3():
    """Three set inclusion-exclusion"""
    print("\nInclusion-Exclusion (Three Sets):\n")

    # Languages example
    total = 200
    python = 120
    java = 90
    cpp = 70
    py_java = 50
    py_cpp = 40
    java_cpp = 30
    all_three = 15

    # Know at least one
    at_least_one = (python + java + cpp
                    - py_java - py_cpp - java_cpp
                    + all_three)

    # Know none
    know_none = total - at_least_one

    print("Languages known:")
    print(f"  Python: {python}, Java: {java}, C++: {cpp}")
    print(f"  Python & Java: {py_java}")
    print(f"  Python & C++: {py_cpp}")
    print(f"  Java & C++: {java_cpp}")
    print(f"  All three: {all_three}")
    print(f"\n  At least one: {at_least_one}")
    print(f"  None: {know_none}")

    # Only Python (Python but not Java or C++)
    only_python = python - py_java - py_cpp + all_three
    print(f"\n  Only Python: {only_python}")


inclusion_exclusion_3()


def inclusion_exclusion_formula():
    """General formula demonstration"""

    def inclusion_exclusion(sets):
        """
        Calculate |A1 ∪ A2 ∪ ... ∪ An| using inclusion-exclusion.
        sets: list of (set_size, intersections)
        """
        from itertools import combinations

        n = len(sets)
        total = 0
        sign = 1

        for k in range(1, n + 1):
            for combo in combinations(range(n), k):
                # Would need intersection sizes for each combination
                pass

        return total

    print("\nInclusion-Exclusion General Formula:")
    print("  |A ∪ B ∪ C ∪ ...| = ")
    print("    Σ|single sets|")
    print("    - Σ|pairs|")
    print("    + Σ|triples|")
    print("    - Σ|quadruples|")
    print("    ...")


inclusion_exclusion_formula()
```

---

## Application Problems (응용 문제)

### Example

```python
def derangement():
    """
    Derangement: permutation where no element is in its original position.
    Uses inclusion-exclusion to count.
    """
    from math import factorial

    def count_derangements(n):
        """Count derangements using inclusion-exclusion"""
        total = 0
        for k in range(n + 1):
            sign = (-1) ** k
            total += sign * factorial(n) // factorial(k)
        return total

    # Alternative formula: D(n) = (n-1) * (D(n-1) + D(n-2))
    def derangement_recursive(n, memo={}):
        if n == 0:
            return 1
        if n == 1:
            return 0
        if n in memo:
            return memo[n]

        memo[n] = (n - 1) * (derangement_recursive(n - 1, memo) +
                            derangement_recursive(n - 2, memo))
        return memo[n]

    print("Derangements (no element in original position):\n")
    print("  n  |  D(n)  | n! | Probability")
    print("  ---+--------+----+------------")

    for n in range(1, 9):
        d = count_derangements(n)
        f = factorial(n)
        prob = d / f if f > 0 else 0
        print(f"  {n}  |  {d:5} | {f:3} | {prob:.4f}")

    # Limit approaches 1/e
    print(f"\n  Limit as n→∞: 1/e ≈ {1/2.718281828:.4f}")


derangement()


def counting_multiples():
    """Count integers with certain divisibility properties"""
    print("\nCounting multiples using Inclusion-Exclusion:\n")

    # Count integers from 1 to n divisible by at least one of given divisors
    def count_divisible(n, divisors):
        from itertools import combinations
        from math import gcd
        from functools import reduce

        def lcm(a, b):
            return a * b // gcd(a, b)

        def lcm_list(nums):
            return reduce(lcm, nums)

        total = 0
        for k in range(1, len(divisors) + 1):
            for combo in combinations(divisors, k):
                l = lcm_list(combo)
                count = n // l
                if k % 2 == 1:
                    total += count
                else:
                    total -= count

        return total

    # Example: 1 to 100, divisible by 2, 3, or 5
    n = 100
    divisors = [2, 3, 5]

    result = count_divisible(n, divisors)
    not_divisible = n - result

    print(f"Integers 1 to {n}:")
    print(f"  Divisible by 2, 3, or 5: {result}")
    print(f"  Not divisible by any: {not_divisible}")

    # Verify
    direct_count = sum(1 for i in range(1, n+1)
                       if i % 2 == 0 or i % 3 == 0 or i % 5 == 0)
    print(f"  (Verified by counting: {direct_count})")


counting_multiples()


def onto_functions():
    """Count onto (surjective) functions using inclusion-exclusion"""
    from math import factorial, comb

    def count_onto(n, m):
        """
        Count onto functions from set of n elements to set of m elements.
        Uses inclusion-exclusion: subtract functions that miss at least one element.
        """
        if n < m:
            return 0  # Can't be onto if domain smaller

        total = 0
        for k in range(m + 1):
            sign = (-1) ** k
            # Functions that miss at least k specific elements
            total += sign * comb(m, k) * (m - k) ** n

        return total

    print("\nOnto (Surjective) Functions:\n")
    print("Functions from {1,...,n} to {1,...,m} that hit every element in codomain\n")

    for n in range(1, 6):
        for m in range(1, min(n + 2, 5)):
            onto = count_onto(n, m)
            total = m ** n
            print(f"  f: {{{n}}} → {{{m}}}: {onto} onto functions (of {total} total)")


onto_functions()
```

---

## Summary

| Principle | Korean | Key Idea |
|-----------|--------|----------|
| Pigeonhole | 비둘기집 | n items in m boxes → one has ⌈n/m⌉ |
| Inc-Exc (2 sets) | 포함-배제 | \|A∪B\| = \|A\| + \|B\| - \|A∩B\| |
| Inc-Exc (general) | 일반형 | Add odds, subtract evens |

**When to use:**
- Pigeonhole: Prove something MUST exist
- Inclusion-Exclusion: Count overlapping sets

---

## Practice Problems

1. 25 balls into 6 boxes. Prove at least one box has 5+ balls.

2. 100 students: 60 take math, 55 take physics, 30 take both. How many take neither?

3. How many integers 1-1000 are divisible by 3 or 5 but not 15?

4. In any group of 13 people, prove at least 2 have birthdays in the same month.
