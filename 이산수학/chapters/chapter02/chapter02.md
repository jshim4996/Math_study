# Chapter 02. Conditionals and Equivalence (조건문과 동치)

> Understanding the nuances of conditional statements and logical equivalence for writing correct and optimized code.

---

## The Conditional Statement (조건문)

### Concept (개념)

The **conditional** p → q ("if p then q") is fundamental to both mathematics and programming. Understanding its variations helps in writing correct logical conditions.

**Components:**
- **Antecedent** (전건): The "if" part (p)
- **Consequent** (후건): The "then" part (q)

Remember: p → q is only False when p is True and q is False.

### Example

```python
# Mathematical statement: "If x > 10, then x > 5"
# This is always True (valid implication)

def validate_implication(x):
    p = x > 10  # antecedent
    q = x > 5   # consequent

    # p → q is False only when p=True and q=False
    # But if x > 10, then x > 5 is always True
    # So this implication never fails

    if p and not q:
        return False  # Implication violated
    return True       # Implication holds

# All these return True:
validate_implication(15)  # p=True, q=True
validate_implication(7)   # p=False, q=True
validate_implication(3)   # p=False, q=False
```

---

## Converse, Inverse, and Contrapositive (역, 이, 대우)

### Concept (개념)

Given a conditional p → q, we can form three related statements:

| Name | Korean | Form | Relation to Original |
|------|--------|------|---------------------|
| Original | 원명제 | p → q | - |
| Converse | 역 | q → p | NOT equivalent |
| Inverse | 이 | ¬p → ¬q | NOT equivalent |
| Contrapositive | 대우 | ¬q → ¬p | EQUIVALENT |

**Key insight**: A conditional and its contrapositive are logically equivalent!

| p | q | p → q | q → p | ¬p → ¬q | ¬q → ¬p |
|---|---|-------|-------|---------|---------|
| T | T | T | T | T | T |
| T | F | F | T | T | F |
| F | T | T | F | F | T |
| F | F | T | T | T | T |

### Example

```python
# Original: "If it is raining, then the ground is wet"
# p = raining, q = ground_wet

# Converse (역): "If ground is wet, then it is raining"
# - NOT equivalent! Ground could be wet from sprinklers

# Inverse (이): "If not raining, then ground is not wet"
# - NOT equivalent! Same reason as above

# Contrapositive (대우): "If ground is not wet, then it is not raining"
# - EQUIVALENT to original! This is always valid

def check_rain_logic(raining, ground_wet):
    # Original implication
    original = (not raining) or ground_wet  # p → q ≡ ¬p ∨ q

    # Contrapositive (should match original)
    contrapositive = ground_wet or (not raining)  # ¬q → ¬p ≡ q ∨ ¬p

    return original == contrapositive  # Always True!
```

---

## Logical Equivalence (논리적 동치)

### Concept (개념)

Two propositions are **logically equivalent** (논리적 동치) if they have the same truth values in all cases. We write this as p ≡ q or p ⟺ q.

**Important Equivalences:**

| Equivalence | Name | Korean |
|-------------|------|--------|
| p → q ≡ ¬p ∨ q | Implication | 조건문의 변환 |
| ¬(p ∧ q) ≡ ¬p ∨ ¬q | De Morgan's Law | 드모르간 법칙 |
| ¬(p ∨ q) ≡ ¬p ∧ ¬q | De Morgan's Law | 드모르간 법칙 |
| p ∨ (q ∧ r) ≡ (p ∨ q) ∧ (p ∨ r) | Distributive | 분배법칙 |
| p ∧ (q ∨ r) ≡ (p ∧ q) ∨ (p ∧ r) | Distributive | 분배법칙 |

### Example

```python
# Proving equivalence with truth table
def verify_demorgan():
    """Verify ¬(p ∧ q) ≡ ¬p ∨ ¬q"""
    for p in [True, False]:
        for q in [True, False]:
            left = not (p and q)      # ¬(p ∧ q)
            right = (not p) or (not q)  # ¬p ∨ ¬q

            if left != right:
                return False
    return True

print(verify_demorgan())  # True - De Morgan's Law verified!
```

---

## De Morgan's Laws in Programming (드모르간 법칙)

### Concept (개념)

**De Morgan's Laws** are essential for simplifying and understanding complex conditions:

1. ¬(p ∧ q) ≡ ¬p ∨ ¬q : "NOT (A AND B)" = "NOT A OR NOT B"
2. ¬(p ∨ q) ≡ ¬p ∧ ¬q : "NOT (A OR B)" = "NOT A AND NOT B"

These laws help when:
- Negating complex conditions
- Simplifying nested conditionals
- Understanding guard clauses

### Example

```python
# Original complex condition
def can_access_v1(user):
    if not (user.is_admin and user.is_active):
        return False
    return True

# Using De Morgan's Law: ¬(p ∧ q) ≡ ¬p ∨ ¬q
def can_access_v2(user):
    if (not user.is_admin) or (not user.is_active):
        return False
    return True

# Both are equivalent! v2 is often clearer for early returns

# Another example: Validation
def is_invalid_input(x):
    # Original: NOT (in range AND is integer)
    return not (0 <= x <= 100 and isinstance(x, int))

# Equivalent using De Morgan:
def is_invalid_input_v2(x):
    # (NOT in range) OR (NOT integer)
    return x < 0 or x > 100 or not isinstance(x, int)
```

---

## Code Optimization with Logical Equivalence (코드 최적화)

### Concept (개념)

Understanding logical equivalence enables:
1. **Condition simplification**: Reduce complex boolean expressions
2. **Short-circuit optimization**: Reorder conditions for efficiency
3. **Guard clause patterns**: Transform nested ifs to early returns

**Common optimizations:**
- `not (not p)` → `p` (Double negation)
- `p and True` → `p` (Identity)
- `p or False` → `p` (Identity)
- `p and False` → `False` (Domination)
- `p or True` → `True` (Domination)

### Example

```python
# Before optimization: Nested and complex
def process_order_v1(order):
    if order is not None:
        if order.is_valid:
            if order.payment_confirmed:
                if not order.is_cancelled:
                    return ship_order(order)
    return None

# After optimization: Guard clauses using contrapositive thinking
def process_order_v2(order):
    # Using contrapositive: "If NOT shippable, return early"
    if order is None:
        return None
    if not order.is_valid:
        return None
    if not order.payment_confirmed:
        return None
    if order.is_cancelled:
        return None

    return ship_order(order)

# Even more concise using AND
def process_order_v3(order):
    if (order is not None and
        order.is_valid and
        order.payment_confirmed and
        not order.is_cancelled):
        return ship_order(order)
    return None
```

---

## Biconditional (쌍조건문)

### Concept (개념)

The **biconditional** p ↔ q ("p if and only if q", 필요충분조건) is True when both sides have the same truth value.

| p | q | p ↔ q |
|---|---|-------|
| T | T | T |
| T | F | F |
| F | T | F |
| F | F | T |

**Equivalent forms:**
- p ↔ q ≡ (p → q) ∧ (q → p)
- p ↔ q ≡ (p ∧ q) ∨ (¬p ∧ ¬q)

This is like the XNOR gate in digital logic or equality check in programming.

### Example

```python
# Biconditional in programming: Both conditions must match
def check_biconditional(p, q):
    # p ↔ q is True when p and q have same truth value
    return p == q  # Simple equality check!

# Practical example: Access control
def sync_status(local_saved, server_saved):
    """
    System is in sync if and only if:
    - Both local and server are saved, OR
    - Both local and server are unsaved
    """
    # This is a biconditional!
    in_sync = local_saved == server_saved
    return in_sync

# Real-world: "You pass if and only if you score >= 60"
# pass ↔ (score >= 60)
# This means: pass implies score >= 60, AND score >= 60 implies pass
```

---

## Proof Techniques Preview: Contrapositive (대우를 이용한 증명)

### Concept (개념)

Since p → q ≡ ¬q → ¬p, we can prove an implication by proving its contrapositive. This is often easier when:
- The original statement involves existence
- Direct proof is complex
- The negation is simpler to work with

### Example

```python
# Theorem: "If n² is even, then n is even"
# Direct proof is tricky. Use contrapositive:
# Contrapositive: "If n is odd, then n² is odd"

def prove_by_contrapositive():
    """
    Proof: If n is odd, then n = 2k + 1 for some integer k
    n² = (2k + 1)² = 4k² + 4k + 1 = 2(2k² + 2k) + 1
    This is of form 2m + 1, so n² is odd.

    Since contrapositive is true, original is true!
    """
    # Verification
    for n in range(1, 100):
        n_even = (n % 2 == 0)
        n_squared_even = (n * n % 2 == 0)

        # Original: n²_even → n_even
        original = (not n_squared_even) or n_even

        # Contrapositive: n_odd → n²_odd
        contrapositive = n_even or (not n_squared_even)

        assert original == contrapositive == True

prove_by_contrapositive()
print("Theorem verified!")
```

---

## Summary

| Concept | Korean | Key Point |
|---------|--------|-----------|
| Conditional | 조건문 | p → q, False only when p=T, q=F |
| Converse | 역 | q → p, NOT equivalent to original |
| Inverse | 이 | ¬p → ¬q, NOT equivalent |
| Contrapositive | 대우 | ¬q → ¬p, EQUIVALENT to original |
| Logical Equivalence | 논리적 동치 | Same truth table |
| De Morgan's Laws | 드모르간 법칙 | ¬(p∧q)≡¬p∨¬q, ¬(p∨q)≡¬p∧¬q |
| Biconditional | 쌍조건문 | p ↔ q, True when both match |
