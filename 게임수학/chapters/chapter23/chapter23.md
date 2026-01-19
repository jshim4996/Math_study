# Chapter 23. Probability Basics (확률 기초)

> The mathematics of chance: understanding probability for game systems.

---

## What is Probability (확률이란 무엇인가)

### Concept (개념)

**Probability** measures how likely an event is to occur, expressed as a number between 0 and 1:

- **0**: Impossible (never happens)
- **1**: Certain (always happens)
- **0.5**: 50% chance (equally likely to happen or not)

**Basic formula:**
```
P(event) = favorable outcomes / total outcomes
```

**Key properties:**
- P(A) ranges from 0 to 1
- P(not A) = 1 - P(A)
- P(A or B) = P(A) + P(B) - P(A and B)

### Game Example
- Critical hit chance
- Loot drop rates
- Enemy spawn probability
- Card drawing

### Code Example
```python
import random

# Basic probability demonstration
def roll_die():
    """Roll a 6-sided die"""
    return random.randint(1, 6)

def probability_of_six():
    """Theoretical: 1/6 ≈ 0.167"""
    return 1 / 6

# Simulate many rolls to verify
def simulate_probability(trials=10000):
    sixes = 0
    for _ in range(trials):
        if roll_die() == 6:
            sixes += 1
    return sixes / trials

print("Probability of rolling a 6:")
print(f"  Theoretical: {probability_of_six():.4f} ({probability_of_six()*100:.2f}%)")
print(f"  Simulated (10k rolls): {simulate_probability():.4f}")


# Probability in games
class CriticalHit:
    def __init__(self, crit_chance=0.2):  # 20% crit rate
        self.crit_chance = crit_chance

    def is_critical(self):
        return random.random() < self.crit_chance

    def attack(self, base_damage):
        if self.is_critical():
            return base_damage * 2, True
        return base_damage, False


# Test critical hits
player = CriticalHit(crit_chance=0.2)

crits = 0
total_attacks = 1000

for _ in range(total_attacks):
    damage, was_crit = player.attack(100)
    if was_crit:
        crits += 1

print(f"\nCritical hit test ({total_attacks} attacks, 20% rate):")
print(f"  Expected crits: {total_attacks * 0.2:.0f}")
print(f"  Actual crits: {crits}")
print(f"  Observed rate: {crits/total_attacks*100:.1f}%")
```

---

## Basic Probability Calculation (확률 계산 기초)

### Concept (개념)

**Coin flip:** P(heads) = 1/2 = 0.5
**Die roll:** P(any specific number) = 1/6 ≈ 0.167
**Card draw (standard deck):** P(ace) = 4/52 ≈ 0.077

**Complementary probability:**
```
P(not A) = 1 - P(A)
```

Example: P(NOT getting a 6) = 1 - 1/6 = 5/6

### Game Example
- P(rare drop) = 0.05 means P(no rare drop) = 0.95
- P(miss) = 1 - P(hit)

### Code Example
```python
import random

# Loot drop system
class LootTable:
    def __init__(self):
        # Rarity: (probability, name)
        self.items = [
            (0.50, "Common"),
            (0.30, "Uncommon"),
            (0.15, "Rare"),
            (0.04, "Epic"),
            (0.01, "Legendary"),
        ]

    def drop(self):
        """Roll for loot"""
        roll = random.random()
        cumulative = 0

        for prob, name in self.items:
            cumulative += prob
            if roll < cumulative:
                return name

        return self.items[-1][1]  # Fallback


loot = LootTable()

# Simulate many drops
results = {}
trials = 10000

for _ in range(trials):
    item = loot.drop()
    results[item] = results.get(item, 0) + 1

print("Loot drop simulation (10,000 drops):")
print("  Item        Expected   Actual")
for prob, name in loot.items:
    expected = prob * trials
    actual = results.get(name, 0)
    print(f"  {name:10}  {expected:6.0f}     {actual:5}")


# Probability of NOT getting legendary in N tries
def prob_no_legendary(tries, legend_rate=0.01):
    """P(no legendary in N tries) = (1 - rate)^N"""
    return (1 - legend_rate) ** tries

def prob_at_least_one_legendary(tries, legend_rate=0.01):
    """P(at least one) = 1 - P(none)"""
    return 1 - prob_no_legendary(tries, legend_rate)


print("\nProbability of getting at least one Legendary:")
for tries in [1, 10, 50, 100, 200, 500]:
    prob = prob_at_least_one_legendary(tries, 0.01)
    print(f"  After {tries:3} tries: {prob*100:.1f}%")
```

---

## Independent vs Dependent Events (독립 사건, 종속 사건)

### Concept (개념)

**Independent events:** One event doesn't affect the other.
```
P(A and B) = P(A) × P(B)
```

Example: Coin flips are independent. Previous flip doesn't affect next flip.

**Dependent events:** One event affects the probability of another.
```
P(A and B) = P(A) × P(B|A)
```
where P(B|A) is "probability of B given A happened"

Example: Drawing cards without replacement.

**Gambler's Fallacy:** Believing past independent events affect future probability. (They don't!)

### Game Example
- Independent: Each enemy kill is separate loot roll
- Dependent: Drawing cards from a deck (card removed after draw)

### Code Example
```python
import random

# Independent events: Dice rolls
def demo_independent():
    """Rolling two dice - each roll is independent"""
    # P(both sixes) = 1/6 × 1/6 = 1/36 ≈ 0.028
    trials = 10000
    both_sixes = 0

    for _ in range(trials):
        die1 = random.randint(1, 6)
        die2 = random.randint(1, 6)
        if die1 == 6 and die2 == 6:
            both_sixes += 1

    print("Independent events (two dice):")
    print(f"  P(both sixes) theoretical: {1/36:.4f}")
    print(f"  P(both sixes) simulated: {both_sixes/trials:.4f}")


# Dependent events: Card drawing
def demo_dependent():
    """Drawing cards without replacement"""
    trials = 10000
    two_aces = 0

    for _ in range(trials):
        # Fresh deck with 4 aces in 52 cards
        aces_left = 4
        cards_left = 52

        # Draw first card
        first_ace = random.random() < (aces_left / cards_left)
        if first_ace:
            aces_left -= 1
        cards_left -= 1

        # Draw second card (probability changed!)
        second_ace = random.random() < (aces_left / cards_left)

        if first_ace and second_ace:
            two_aces += 1

    # P(two aces) = 4/52 × 3/51 ≈ 0.0045
    theoretical = (4/52) * (3/51)

    print("\nDependent events (card drawing):")
    print(f"  P(two aces) theoretical: {theoretical:.4f}")
    print(f"  P(two aces) simulated: {two_aces/trials:.4f}")


demo_independent()
demo_dependent()


# Card deck for games
class Deck:
    def __init__(self):
        self.reset()

    def reset(self):
        """Shuffle new deck"""
        self.cards = list(range(52))  # 0-51
        random.shuffle(self.cards)

    def draw(self):
        """Draw a card (removes from deck)"""
        if len(self.cards) == 0:
            return None
        return self.cards.pop()

    def cards_remaining(self):
        return len(self.cards)

    def get_card_name(self, card_num):
        suits = ["Hearts", "Diamonds", "Clubs", "Spades"]
        ranks = ["A", "2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K"]
        suit = suits[card_num // 13]
        rank = ranks[card_num % 13]
        return f"{rank} of {suit}"


# Card game example
deck = Deck()
print("\nCard drawing (dependent):")
print(f"  Initial deck size: {deck.cards_remaining()}")

for i in range(5):
    card = deck.draw()
    name = deck.get_card_name(card)
    print(f"  Draw {i+1}: {name}, remaining: {deck.cards_remaining()}")
```

---

## Expected Value (기대값)

### Concept (개념)

**Expected value** is the average outcome if you repeated something many times.

```
E[X] = Σ (value × probability)
```

Example: Die roll expected value = (1+2+3+4+5+6)/6 = 3.5

**In games:**
- Expected damage per attack
- Average gold per enemy
- Long-term value of a random reward

### Game Example
- Is a 10% chance at 1000 gold better than guaranteed 80 gold?
  - Expected: 0.1 × 1000 = 100 gold (better!)
- Damage: 100 base, 20% crit for 2x
  - Expected: 0.8 × 100 + 0.2 × 200 = 120

### Code Example
```python
import random

def expected_value(outcomes):
    """
    Calculate expected value.
    outcomes: list of (probability, value) tuples
    """
    return sum(prob * value for prob, value in outcomes)


# Die roll expected value
die_outcomes = [(1/6, i) for i in range(1, 7)]
print(f"Die roll expected value: {expected_value(die_outcomes):.2f}")


# Game damage calculation
def calculate_expected_damage(base_damage, crit_chance, crit_multiplier):
    outcomes = [
        (1 - crit_chance, base_damage),              # Normal hit
        (crit_chance, base_damage * crit_multiplier)  # Critical hit
    ]
    return expected_value(outcomes)


base = 100
crit_chance = 0.2
crit_mult = 2.0

exp_damage = calculate_expected_damage(base, crit_chance, crit_mult)
print(f"\nExpected damage (base=100, 20% crit, 2x multiplier):")
print(f"  Expected: {exp_damage}")

# Verify with simulation
total_damage = 0
attacks = 10000
for _ in range(attacks):
    if random.random() < crit_chance:
        total_damage += base * crit_mult
    else:
        total_damage += base

print(f"  Simulated average: {total_damage / attacks:.1f}")


# Loot box expected value
def loot_box_expected_value():
    # Loot box costs 100 gold
    cost = 100

    # Contents with probabilities and values
    contents = [
        (0.60, 50),    # 60% chance: 50 gold worth
        (0.25, 100),   # 25% chance: 100 gold worth
        (0.10, 200),   # 10% chance: 200 gold worth
        (0.04, 500),   # 4% chance: 500 gold worth
        (0.01, 2000),  # 1% chance: 2000 gold worth
    ]

    expected = expected_value(contents)
    return expected, expected - cost


exp_value, net = loot_box_expected_value()
print(f"\nLoot box analysis:")
print(f"  Cost: 100 gold")
print(f"  Expected value: {exp_value:.1f} gold")
print(f"  Net expected: {net:.1f} gold ({'profit' if net > 0 else 'loss'})")


# Comparing two weapons
def compare_weapons():
    # Weapon A: 100 damage, no variance
    weapon_a = 100

    # Weapon B: 50-150 damage random
    weapon_b_outcomes = [(0.2, 50), (0.2, 75), (0.2, 100), (0.2, 125), (0.2, 150)]
    weapon_b = expected_value(weapon_b_outcomes)

    print(f"\nWeapon comparison:")
    print(f"  Weapon A (consistent): {weapon_a} damage")
    print(f"  Weapon B (random 50-150): {weapon_b} expected damage")
    print(f"  Same expected value, different variance!")


compare_weapons()
```

---

## Summary (요약)

| Concept | Korean | Formula |
|---------|--------|---------|
| Probability | 확률 | P = favorable / total |
| Complement | 여사건 | P(not A) = 1 - P(A) |
| Independent | 독립 | P(A and B) = P(A) × P(B) |
| Dependent | 종속 | P(A and B) = P(A) × P(B\|A) |
| Expected Value | 기대값 | E[X] = Σ(value × prob) |

---

## Practice Problems (연습 문제)

1. What's the probability of rolling at least one 6 in three dice rolls?

2. A loot drop has 5% legendary rate. What's the probability of getting no legendary in 50 drops?

3. Calculate expected damage: 80 base, 25% crit chance, 1.5x crit multiplier.

4. Is a 1% chance at 10,000 gold better than guaranteed 80 gold?
