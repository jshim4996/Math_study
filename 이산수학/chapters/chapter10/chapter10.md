# Chapter 10. Counting (경우의 수)

> The foundation of combinatorics: systematic ways to count possibilities.

---

## Addition Rule (합의 법칙)

### Concept (개념)

When choosing from **mutually exclusive** options, add the counts:

```
If you can do task A in m ways OR task B in n ways (but not both),
total ways = m + n
```

**Key condition:** Options must not overlap (mutually exclusive).

### Example

```python
def addition_rule_demo():
    # Restaurant menu: main course options
    pasta_options = ["Carbonara", "Bolognese", "Pesto"]
    pizza_options = ["Margherita", "Pepperoni", "Hawaiian", "Veggie"]

    # You choose ONE main (pasta OR pizza)
    total_options = len(pasta_options) + len(pizza_options)

    print("Addition Rule Example:")
    print(f"  Pasta options: {len(pasta_options)}")
    print(f"  Pizza options: {len(pizza_options)}")
    print(f"  Total main course options: {total_options}")

    return total_options


addition_rule_demo()


# Programming application: Login methods
def count_login_options():
    email_providers = 3      # Gmail, Outlook, Yahoo
    social_logins = 4        # Google, Facebook, Twitter, GitHub
    phone_login = 1          # SMS verification

    # User chooses ONE method
    total = email_providers + social_logins + phone_login

    print("\nLogin options:")
    print(f"  Email: {email_providers} ways")
    print(f"  Social: {social_logins} ways")
    print(f"  Phone: {phone_login} way")
    print(f"  Total login methods: {total}")


count_login_options()


# Multiple categories
def count_with_overlap():
    """When sets overlap, use inclusion-exclusion"""
    students_in_math = 30
    students_in_cs = 25
    students_in_both = 10

    # Wrong: 30 + 25 = 55 (double counts students in both)
    # Correct: 30 + 25 - 10 = 45
    total_unique = students_in_math + students_in_cs - students_in_both

    print("\nOverlapping sets (inclusion-exclusion):")
    print(f"  Math: {students_in_math}, CS: {students_in_cs}, Both: {students_in_both}")
    print(f"  Unique students: {total_unique}")


count_with_overlap()
```

---

## Multiplication Rule (곱의 법칙)

### Concept (개념)

When doing **sequential** independent tasks, multiply the counts:

```
If task A can be done in m ways AND task B can be done in n ways,
total ways = m × n
```

**Key condition:** Choices are independent (one doesn't affect another).

### Example

```python
def multiplication_rule_demo():
    # Outfit combinations
    shirts = ["White", "Blue", "Black"]
    pants = ["Jeans", "Khakis"]
    shoes = ["Sneakers", "Boots"]

    # You choose one of each (shirt AND pants AND shoes)
    total_outfits = len(shirts) * len(pants) * len(shoes)

    print("Multiplication Rule Example:")
    print(f"  Shirts: {len(shirts)}")
    print(f"  Pants: {len(pants)}")
    print(f"  Shoes: {len(shoes)}")
    print(f"  Total outfits: {total_outfits}")

    # List all combinations
    print("\nAll outfits:")
    count = 0
    for s in shirts:
        for p in pants:
            for sh in shoes:
                count += 1
                print(f"  {count}: {s}, {p}, {sh}")


multiplication_rule_demo()


# Password combinations
def count_passwords():
    # Password format: 2 letters + 3 digits
    letters = 26  # a-z
    digits = 10   # 0-9

    # Letter1 AND Letter2 AND Digit1 AND Digit2 AND Digit3
    total = letters * letters * digits * digits * digits

    print("\nPassword combinations (2 letters + 3 digits):")
    print(f"  Positions: letter × letter × digit × digit × digit")
    print(f"  Total: {letters} × {letters} × {digits} × {digits} × {digits} = {total:,}")


count_passwords()


# Menu combinations
def restaurant_menu():
    appetizers = 5
    mains = 8
    desserts = 4
    drinks = 6

    # Full meal: one of each
    full_meal = appetizers * mains * desserts * drinks

    # Just main and drink
    simple_meal = mains * drinks

    print("\nRestaurant menu combinations:")
    print(f"  Full meal (app + main + dessert + drink): {full_meal}")
    print(f"  Simple meal (main + drink): {simple_meal}")


restaurant_menu()
```

---

## Counting Principles (경우의 수 세기)

### Concept (개념)

**Combining rules:**
- Use **addition** for OR situations
- Use **multiplication** for AND situations

**Decision trees** help visualize:
```
           Root
          /    \
      Choice1  Choice2
        /\        /\
       C3 C4    C5 C6
```

Count = number of leaf nodes

### Example

```python
def decision_tree_counting():
    """
    Travel from A to B to C
    A to B: 3 routes (bus, train, car)
    B to C: 2 routes (bus, plane)

    Total routes = 3 × 2 = 6
    """
    a_to_b = ["Bus1", "Train", "Car"]
    b_to_c = ["Bus2", "Plane"]

    print("Travel routes A → B → C:")
    print(f"  A→B options: {len(a_to_b)}")
    print(f"  B→C options: {len(b_to_c)}")

    count = 0
    for route1 in a_to_b:
        for route2 in b_to_c:
            count += 1
            print(f"  Route {count}: {route1} → {route2}")

    print(f"  Total: {count}")


decision_tree_counting()


# Complex example: License plates
def license_plate_counting():
    """
    Format: 3 letters followed by 4 digits
    Letters: A-Z (26)
    Digits: 0-9 (10)
    """
    letters = 26
    digits = 10

    # All different (no repeats)
    different = 26 * 25 * 24 * 10 * 9 * 8 * 7

    # Repeats allowed
    with_repeats = 26 * 26 * 26 * 10 * 10 * 10 * 10

    print("\nLicense plates (3 letters + 4 digits):")
    print(f"  With repeats allowed: {with_repeats:,}")
    print(f"  All different: {different:,}")


license_plate_counting()


# Conditional counting
def conditional_count():
    """
    Form a 3-digit number from {1,2,3,4,5}
    a) All digits different
    b) All digits same
    c) First digit is 1
    """
    digits = [1, 2, 3, 4, 5]

    # a) All different: 5 × 4 × 3
    all_diff = 5 * 4 * 3

    # b) All same: 5 (111, 222, 333, 444, 555)
    all_same = 5

    # c) First is 1: 1 × 5 × 5 (others can repeat)
    first_is_1 = 1 * 5 * 5

    # c-alt) First is 1, all different: 1 × 4 × 3
    first_is_1_diff = 1 * 4 * 3

    print("\n3-digit numbers from {1,2,3,4,5}:")
    print(f"  All different: {all_diff}")
    print(f"  All same: {all_same}")
    print(f"  First digit is 1 (repeats ok): {first_is_1}")
    print(f"  First digit is 1 (all different): {first_is_1_diff}")


conditional_count()
```

---

## Practice Applications

### Example

```python
# Application 1: Committee selection
def committee_problems():
    """
    10 people, select committee of 3
    How many ways if:
    a) Anyone can be selected
    b) Person A must be on committee
    c) Person A must NOT be on committee
    """
    from math import comb

    n = 10

    # a) Choose 3 from 10
    total = comb(10, 3)

    # b) A is fixed, choose 2 more from remaining 9
    with_a = comb(9, 2)

    # c) Choose 3 from 9 (excluding A)
    without_a = comb(9, 3)

    # Verify: with_a + without_a = total
    print("Committee selection (10 people, choose 3):")
    print(f"  Total ways: {total}")
    print(f"  Must include A: {with_a}")
    print(f"  Must exclude A: {without_a}")
    print(f"  Check: {with_a} + {without_a} = {with_a + without_a}")


committee_problems()


# Application 2: Bit strings
def bit_string_counting():
    """
    8-bit strings
    a) Total
    b) Starting with 1
    c) Starting with 1 and ending with 0
    d) Having exactly three 1s
    """
    from math import comb

    n = 8

    # a) Each bit: 2 choices, 8 positions
    total = 2 ** n

    # b) First bit fixed, 7 remaining
    starts_with_1 = 2 ** 7

    # c) First and last fixed, 6 remaining
    starts_1_ends_0 = 2 ** 6

    # d) Choose 3 positions for 1s
    exactly_three_1s = comb(8, 3)

    print("\n8-bit strings:")
    print(f"  Total: {total}")
    print(f"  Starting with 1: {starts_with_1}")
    print(f"  Starting with 1, ending with 0: {starts_1_ends_0}")
    print(f"  Exactly three 1s: {exactly_three_1s}")


bit_string_counting()


# Application 3: Functions between sets
def function_counting():
    """
    Functions from set A (3 elements) to set B (4 elements)
    a) Total functions
    b) Injective (one-to-one)
    """
    a_size = 3
    b_size = 4

    # a) Each element in A can map to any in B
    total = b_size ** a_size

    # b) Injective: first has 4 choices, second has 3, third has 2
    injective = 4 * 3 * 2

    print(f"\nFunctions from {{a,b,c}} to {{1,2,3,4}}:")
    print(f"  Total functions: {total}")
    print(f"  Injective functions: {injective}")


function_counting()
```

---

## Summary

| Rule | Korean | When to Use |
|------|--------|-------------|
| Addition | 합의 법칙 | Choose ONE from disjoint sets |
| Multiplication | 곱의 법칙 | Make sequential independent choices |
| Combined | 복합 | Break into sub-problems |

**Key questions:**
- OR (mutually exclusive)? → Add
- AND (sequential)? → Multiply
- Repeats allowed? → Affects count significantly

---

## Practice Problems

1. A password must be 6-8 characters, using letters and digits. How many passwords?

2. How many 4-digit even numbers exist (no leading zero)?

3. A committee of 4 from 6 men and 5 women must have at least 2 women. How many ways?

4. How many ways to arrange letters in "MISSISSIPPI"?
