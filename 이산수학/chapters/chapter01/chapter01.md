# Chapter 01. Propositions and Logical Operations (명제와 논리 연산)

> The foundation of mathematical reasoning and computational logic through propositions and logical connectives.

---

## What is a Proposition? (명제란?)

### Concept (개념)

A **proposition** (명제) is a declarative statement that is either true (T) or false (F), but not both. Propositions are the building blocks of logic and form the basis for conditional statements in programming.

- **Valid propositions**: "5 is greater than 3" (True), "The sky is green" (False)
- **Not propositions**: "What time is it?" (question), "x + 5 = 10" (variable - truth depends on x)

In programming, every `if` condition must evaluate to a boolean value, which is essentially a proposition.

### Example

```python
# These are propositions in programming context
is_adult = age >= 18          # Either True or False
is_valid = len(password) > 8  # Either True or False

# The if statement requires a proposition
if is_adult:
    print("Access granted")
```

---

## Logical Connectives (논리 연산자)

### Concept (개념)

Logical connectives combine propositions to form compound propositions (복합 명제).

| Connective | Symbol | Name | Korean |
|------------|--------|------|--------|
| NOT | ¬p | Negation | 부정 |
| AND | p ∧ q | Conjunction | 논리곱 |
| OR | p ∨ q | Disjunction | 논리합 |

**NOT (¬)**: Reverses the truth value
- If p is True, ¬p is False
- If p is False, ¬p is True

**AND (∧)**: True only when both operands are True

**OR (∨)**: True when at least one operand is True

### Example

```python
# Logical operators in Python
p = True
q = False

not_p = not p           # False (¬p)
p_and_q = p and q       # False (p ∧ q)
p_or_q = p or q         # True  (p ∨ q)

# Real-world example: Login validation
valid_username = len(username) >= 4
valid_password = len(password) >= 8
can_login = valid_username and valid_password  # Both must be True
```

---

## Truth Tables (진리표)

### Concept (개념)

A **truth table** (진리표) systematically lists all possible truth values for propositions and their combinations. For n propositions, there are 2^n possible combinations.

**NOT Truth Table:**
| p | ¬p |
|---|-----|
| T | F |
| F | T |

**AND Truth Table:**
| p | q | p ∧ q |
|---|---|-------|
| T | T | T |
| T | F | F |
| F | T | F |
| F | F | F |

**OR Truth Table:**
| p | q | p ∨ q |
|---|---|-------|
| T | T | T |
| T | F | T |
| F | T | T |
| F | F | F |

### Example

```python
# Generating a truth table programmatically
def truth_table_and():
    print("p\tq\tp AND q")
    for p in [True, False]:
        for q in [True, False]:
            print(f"{p}\t{q}\t{p and q}")

# Output:
# p       q       p AND q
# True    True    True
# True    False   False
# False   True    False
# False   False   False
```

---

## Conditional Propositions (조건 명제)

### Concept (개념)

The **conditional** (조건문) p → q reads "if p then q" where:
- p is the **hypothesis** (가정) or antecedent
- q is the **conclusion** (결론) or consequent

**Key insight**: p → q is False ONLY when p is True and q is False.

| p | q | p → q |
|---|---|-------|
| T | T | T |
| T | F | F |
| F | T | T |
| F | F | T |

The last two rows may seem counterintuitive: when the hypothesis is false, the implication is "vacuously true" (공허하게 참).

### Example

```python
# Conditional logic in programming
def check_eligibility(age, has_id):
    # "If you are 18+ AND have ID, then you can enter"
    # p = (age >= 18 and has_id)
    # q = can_enter

    if age >= 18 and has_id:
        return True  # p → q when p is True
    return False

# The implication is "broken" only when:
# - User is 18+ with ID (p = True)
# - But system denies entry (q = False)
```

---

## Compound Propositions (복합 명제)

### Concept (개념)

**Compound propositions** combine multiple propositions using logical connectives. Understanding operator precedence is crucial:

1. ¬ (NOT) - highest precedence
2. ∧ (AND)
3. ∨ (OR)
4. → (Conditional)
5. ↔ (Biconditional) - lowest precedence

Complex conditions in programming follow the same precedence rules.

### Example

```python
# Compound proposition: (p ∧ q) ∨ (¬r)
p = True
q = False
r = True

# Step-by-step evaluation:
# 1. p ∧ q = True ∧ False = False
# 2. ¬r = ¬True = False
# 3. False ∨ False = False

result = (p and q) or (not r)  # False

# Real-world: E-commerce discount eligibility
is_member = True
order_above_100 = False
has_coupon = True

# Discount if: (member AND order > $100) OR has coupon
gets_discount = (is_member and order_above_100) or has_coupon  # True
```

---

## Programming Connection: Short-Circuit Evaluation

### Concept (개념)

Most programming languages use **short-circuit evaluation** (단축 평가) for logical operators:

- **AND**: If the first operand is False, the second is not evaluated
- **OR**: If the first operand is True, the second is not evaluated

This is an optimization based on truth table properties.

### Example

```python
# Short-circuit AND: second part not evaluated if first is False
def expensive_check():
    print("Running expensive operation...")
    return True

result = False and expensive_check()  # "Running..." never prints

# Short-circuit OR: second part not evaluated if first is True
result = True or expensive_check()    # "Running..." never prints

# Practical use: Safe null checking
user = None
# Without short-circuit, this would crash:
if user is not None and user.is_active:
    print("User is active")
```

---

## Summary

| Concept | Symbol | Programming Equivalent |
|---------|--------|----------------------|
| Proposition (명제) | p, q | Boolean variable |
| NOT (부정) | ¬p | `not p`, `!p` |
| AND (논리곱) | p ∧ q | `p and q`, `p && q` |
| OR (논리합) | p ∨ q | `p or q`, `p \|\| q` |
| Conditional (조건문) | p → q | `if p then q` |
| Truth Table (진리표) | - | Test case enumeration |
