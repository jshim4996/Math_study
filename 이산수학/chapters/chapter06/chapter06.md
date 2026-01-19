# Chapter 06. Functions (함수)

> Bridging mathematical functions and programming functions to understand mappings, transformations, and computational behavior.

---

## What is a Function? (함수란?)

### Concept (개념)

A **function** (함수) f from set A to set B (written f: A → B) is a special relation where each element in A is related to exactly one element in B.

**Key requirements:**
1. **Total**: Every element in A must be mapped
2. **Unique**: Each element in A maps to exactly one element in B

**Notation:**
- f: A → B (f maps from A to B)
- f(a) = b (f maps a to b)
- a ↦ b (a maps to b)

### Example

```python
# Valid function: each input has exactly one output
def square(x):
    return x * x

# This is a function from integers to integers
# f: Z → Z, f(x) = x²

# As a mapping (dictionary)
f = {1: 1, 2: 4, 3: 9, 4: 16, 5: 25}

# Checking if a relation is a function
def is_function(A, relation):
    """Check if relation is a valid function from A"""
    mapped = {}
    for (a, b) in relation:
        if a in mapped:
            if mapped[a] != b:
                return False  # Maps to multiple values
        mapped[a] = b

    # Check if all elements of A are mapped
    return set(mapped.keys()) == A

A = {1, 2, 3}
R1 = {(1, 'a'), (2, 'b'), (3, 'c')}  # Valid function
R2 = {(1, 'a'), (1, 'b'), (2, 'c')}  # Invalid: 1 maps to both 'a' and 'b'
R3 = {(1, 'a'), (2, 'b')}            # Invalid: 3 is not mapped

print(is_function(A, R1))  # True
print(is_function(A, R2))  # False
print(is_function(A, R3))  # False
```

---

## Domain, Codomain, and Range (정의역, 공역, 치역)

### Concept (개념)

For a function f: A → B:

| Term | Korean | Definition |
|------|--------|------------|
| **Domain** | 정의역 | Set A, all possible inputs |
| **Codomain** | 공역 | Set B, all potential outputs |
| **Range/Image** | 치역 | Actual outputs: {f(a) | a ∈ A} |

**Important:** Range ⊆ Codomain (always)

### Example

```python
# Example: f(x) = x² where domain is {-2, -1, 0, 1, 2}
domain = {-2, -1, 0, 1, 2}
codomain = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10}  # Potential outputs

def f(x):
    return x * x

# Calculate range (actual outputs)
range_f = {f(x) for x in domain}
print(f"Domain: {domain}")
print(f"Codomain: {codomain}")
print(f"Range: {range_f}")  # {0, 1, 4}

# Note: Range is a subset of codomain
print(f"Range ⊆ Codomain: {range_f.issubset(codomain)}")  # True

# Practical: Type annotations in Python show domain/codomain
from typing import List

def get_first(items: List[int]) -> int:
    """
    Domain: List[int] (all lists of integers)
    Codomain: int (all integers)
    Range: depends on actual inputs
    """
    return items[0] if items else 0
```

---

## Injective Functions (단사 함수)

### Concept (개념)

A function f: A → B is **injective** (단사함수, 일대일함수) if different inputs always produce different outputs.

**Definition:** f(a₁) = f(a₂) implies a₁ = a₂

**Equivalently:** a₁ ≠ a₂ implies f(a₁) ≠ f(a₂)

**Visual:** Each element in range is hit by at most one arrow.

### Example

```python
def is_injective(A, f):
    """Check if function f is injective on domain A"""
    outputs = set()
    for a in A:
        output = f(a)
        if output in outputs:
            return False  # Two inputs have same output
        outputs.add(output)
    return True

# Injective: f(x) = 2x
def double(x):
    return 2 * x

# Not injective: f(x) = x²
def square(x):
    return x * x

A = {-2, -1, 0, 1, 2}

print(f"double is injective: {is_injective(A, double)}")  # True
print(f"square is injective: {is_injective(A, square)}")  # False
# Because square(-1) = square(1) = 1

# Practical: Unique ID generation must be injective
class IDGenerator:
    def __init__(self):
        self.counter = 0

    def generate_id(self, user):
        """Injective: each user gets a unique ID"""
        self.counter += 1
        return f"USER_{self.counter}"

    # NOT injective would be: return user.name[:2]
    # (Two users with same starting letters = collision)
```

---

## Surjective Functions (전사 함수)

### Concept (개념)

A function f: A → B is **surjective** (전사함수, 위로의 함수) if every element in the codomain is mapped to by at least one element in the domain.

**Definition:** For every b ∈ B, there exists a ∈ A such that f(a) = b

**Equivalently:** Range = Codomain

**Visual:** Every element in codomain is hit by at least one arrow.

### Example

```python
def is_surjective(A, B, f):
    """Check if f: A → B is surjective"""
    range_f = {f(a) for a in A}
    return range_f == B

# Example: f(x) = x mod 3
def mod_3(x):
    return x % 3

A = {0, 1, 2, 3, 4, 5}
B = {0, 1, 2}

print(f"mod_3 is surjective onto {B}: {is_surjective(A, B, mod_3)}")  # True
# Because 0→0, 1→1, 2→2, 3→0, 4→1, 5→2 covers all of {0,1,2}

# Not surjective if codomain is larger
B_larger = {0, 1, 2, 3}
print(f"mod_3 is surjective onto {B_larger}: {is_surjective(A, B_larger, mod_3)}")  # False

# Practical: Hash function should ideally be surjective over its range
def simple_hash(s, table_size):
    """Should produce all possible indices"""
    return sum(ord(c) for c in s) % table_size
```

---

## Bijective Functions (전단사 함수)

### Concept (개념)

A function f: A → B is **bijective** (전단사함수, 일대일 대응) if it is both injective AND surjective.

**Properties:**
- Each element in A maps to exactly one element in B
- Each element in B is mapped from exactly one element in A
- |A| = |B| for finite sets
- f has an inverse function f⁻¹: B → A

**Applications:**
- Encryption/Decryption (must be reversible)
- Data encoding
- Permutations

### Example

```python
def is_bijective(A, B, f):
    """Check if f: A → B is bijective"""
    # Check injective
    outputs = set()
    for a in A:
        out = f(a)
        if out in outputs:
            return False
        outputs.add(out)

    # Check surjective
    return outputs == B

# Bijective example: Caesar cipher
def caesar_encrypt(char, shift=3):
    if 'a' <= char <= 'z':
        return chr((ord(char) - ord('a') + shift) % 26 + ord('a'))
    return char

def caesar_decrypt(char, shift=3):
    return caesar_encrypt(char, -shift)

alphabet = set('abcdefghijklmnopqrstuvwxyz')
encrypted = {caesar_encrypt(c) for c in alphabet}

print(f"Bijective: {encrypted == alphabet}")  # True

# The inverse exists:
original = 'hello'
encrypted_msg = ''.join(caesar_encrypt(c) for c in original)
decrypted_msg = ''.join(caesar_decrypt(c) for c in encrypted_msg)
print(f"'{original}' → '{encrypted_msg}' → '{decrypted_msg}'")
# 'hello' → 'khoor' → 'hello'

# Cardinality proof using bijection
# Two sets have same size iff there exists a bijection between them
```

---

## Function Composition (합성 함수)

### Concept (개념)

Given f: A → B and g: B → C, the **composition** (합성) g ∘ f: A → C is defined as:
(g ∘ f)(x) = g(f(x))

**Read:** "g after f" or "g composed with f"

**Properties:**
- Associative: (h ∘ g) ∘ f = h ∘ (g ∘ f)
- NOT commutative: g ∘ f ≠ f ∘ g (generally)
- Identity: f ∘ id = id ∘ f = f

### Example

```python
# Function composition
def f(x):
    return x + 1  # f: x → x + 1

def g(x):
    return x * 2  # g: x → 2x

def compose(g, f):
    """Return composition g ∘ f"""
    return lambda x: g(f(x))

g_after_f = compose(g, f)  # g(f(x)) = 2(x + 1)
f_after_g = compose(f, g)  # f(g(x)) = 2x + 1

x = 3
print(f"(g ∘ f)(3) = g(f(3)) = g(4) = {g_after_f(x)}")  # 8
print(f"(f ∘ g)(3) = f(g(3)) = f(6) = {f_after_g(x)}")  # 7

# Composition is NOT commutative!
print(f"g ∘ f ≠ f ∘ g: {g_after_f(x) != f_after_g(x)}")  # True

# Practical: Pipeline pattern
def clean_data(data):
    return data.strip()

def parse_int(data):
    return int(data)

def double(x):
    return x * 2

# Compose: double(parse_int(clean_data(input)))
pipeline = compose(compose(double, parse_int), clean_data)
print(pipeline("  42  "))  # 84
```

---

## Inverse Functions (역함수)

### Concept (개념)

If f: A → B is bijective, then its **inverse** (역함수) f⁻¹: B → A satisfies:
- f⁻¹(f(a)) = a for all a ∈ A
- f(f⁻¹(b)) = b for all b ∈ B

In other words: f⁻¹ ∘ f = idₐ and f ∘ f⁻¹ = idᵦ

**Existence:** f⁻¹ exists if and only if f is bijective.

### Example

```python
# Inverse function example
def f(x):
    return 2 * x + 1  # f: x → 2x + 1

def f_inverse(y):
    return (y - 1) / 2  # f⁻¹: y → (y-1)/2

# Verify: f⁻¹(f(x)) = x
x = 5
print(f"f({x}) = {f(x)}")                    # 11
print(f"f⁻¹(f({x})) = {f_inverse(f(x))}")    # 5.0 (back to x)

# Verify: f(f⁻¹(y)) = y
y = 11
print(f"f⁻¹({y}) = {f_inverse(y)}")          # 5.0
print(f"f(f⁻¹({y})) = {f(f_inverse(y))}")    # 11.0 (back to y)

# Practical: Encoding/Decoding
import base64

def encode(s):
    return base64.b64encode(s.encode()).decode()

def decode(s):
    return base64.b64decode(s.encode()).decode()

message = "Hello, World!"
encoded = encode(message)
decoded = decode(encoded)
print(f"Original: {message}")
print(f"Encoded: {encoded}")
print(f"Decoded: {decoded}")
# decode is the inverse of encode
```

---

## Programming vs Mathematical Functions (프로그래밍 함수 vs 수학 함수)

### Concept (개념)

Mathematical and programming functions differ in important ways:

| Aspect | Mathematical Function | Programming Function |
|--------|----------------------|---------------------|
| Side effects | None (pure) | Can have side effects |
| Same input | Always same output | May vary (state, I/O) |
| Total | Must handle all domain | May throw exceptions |
| Determinism | Always deterministic | Can be random |

**Pure functions** in programming are closest to mathematical functions.

### Example

```python
import random
from datetime import datetime

# PURE function (like mathematical function)
def pure_add(a, b):
    return a + b  # Same input → same output, no side effects

# IMPURE: Uses external state
counter = 0
def impure_count():
    global counter
    counter += 1
    return counter
# Each call returns different value!

# IMPURE: Side effect (I/O)
def impure_log(message):
    print(message)  # Side effect
    return message

# IMPURE: Non-deterministic
def impure_random():
    return random.randint(1, 100)  # Different each time

# IMPURE: Depends on external state
def impure_time():
    return datetime.now()  # Changes every call

# Benefits of pure functions:
# 1. Testable: same input = same output
# 2. Cacheable: can memoize results
# 3. Parallelizable: no shared state
# 4. Predictable: easier to reason about

from functools import lru_cache

@lru_cache(maxsize=100)
def fibonacci(n):
    """Pure function - can be memoized"""
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```

---

## Special Functions (특수 함수)

### Concept (개념)

Several special functions appear frequently:

| Function | Definition | Use Case |
|----------|------------|----------|
| **Identity** (항등함수) | id(x) = x | Placeholder, no-op |
| **Constant** (상수함수) | f(x) = c | Default values |
| **Floor** (바닥함수) | ⌊x⌋ = greatest integer ≤ x | Array indexing |
| **Ceiling** (천장함수) | ⌈x⌉ = smallest integer ≥ x | Pagination |

### Example

```python
import math

# Identity function
identity = lambda x: x
print(identity(42))  # 42

# Constant function
always_zero = lambda x: 0
print(always_zero("anything"))  # 0

# Floor and ceiling
x = 3.7
print(f"⌊{x}⌋ = {math.floor(x)}")  # 3
print(f"⌈{x}⌉ = {math.ceil(x)}")   # 4

# Practical: Pagination
def get_page_count(total_items, items_per_page):
    """Use ceiling to get number of pages needed"""
    return math.ceil(total_items / items_per_page)

print(f"100 items, 30 per page = {get_page_count(100, 30)} pages")  # 4

# Practical: Array index from continuous value
def get_bucket(value, min_val, max_val, num_buckets):
    """Use floor to assign to bucket"""
    normalized = (value - min_val) / (max_val - min_val)
    bucket = int(math.floor(normalized * num_buckets))
    return min(bucket, num_buckets - 1)  # Clamp to valid range
```

---

## Summary

| Concept | Korean | Definition |
|---------|--------|------------|
| Function | 함수 | f: A → B, each input → one output |
| Domain | 정의역 | Set of all inputs |
| Codomain | 공역 | Set of potential outputs |
| Range | 치역 | Set of actual outputs |
| Injective | 단사함수 | Different inputs → different outputs |
| Surjective | 전사함수 | Every codomain element is hit |
| Bijective | 전단사함수 | Injective + Surjective |
| Composition | 합성함수 | (g ∘ f)(x) = g(f(x)) |
| Inverse | 역함수 | f⁻¹: reverses f |
