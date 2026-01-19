# Chapter 24. Game Probability Systems (게임 확률 시스템)

> Practical probability in games: weighted random, gacha systems, and player-friendly probability manipulation.

---

## Random vs Probability (랜덤과 확률의 차이)

### Concept (개념)

**Random** is the mechanism (how we generate unpredictable numbers).
**Probability** is the distribution (what numbers should appear and how often).

**Key insight:**
True randomness can feel "unfair" to players. Games often manipulate probability to feel better while appearing random.

**Problems with true random:**
- Streaks (5 misses in a row feels bad)
- Unlucky players (someone always gets worst luck)
- Perceived unfairness (even though it's statistically correct)

### Game Example
- 25% hit chance but missing 10 times in a row is possible (0.75^10 ≈ 5.6%)
- Players feel cheated even though it's mathematically correct

### Code Example
```python
import random

def true_random_hit(hit_chance=0.25):
    """True random: each roll is independent"""
    return random.random() < hit_chance


def demo_true_random_streaks():
    """Show that true random can have long miss streaks"""
    hit_chance = 0.25
    max_streak = 0
    current_streak = 0

    for _ in range(10000):
        if true_random_hit(hit_chance):
            max_streak = max(max_streak, current_streak)
            current_streak = 0
        else:
            current_streak += 1

    print(f"True random (25% hit, 10000 attempts):")
    print(f"  Longest miss streak: {max_streak}")
    print(f"  Expected worst streak in 10000: ~13 misses")
    # P(13 misses) = 0.75^13 ≈ 0.02, occurs ~200 times in 10000


demo_true_random_streaks()


# Comparing true vs perceived fairness
def player_perception_test():
    """Players often think true random is broken"""
    trials = 20
    true_results = []
    hit_chance = 0.5

    for _ in range(trials):
        true_results.append("HIT" if random.random() < hit_chance else "miss")

    print(f"\n50% hit chance, {trials} attacks:")
    print(f"  Results: {' '.join(true_results)}")
    print(f"  Hits: {true_results.count('HIT')}/{trials}")
    print("  (May look 'streaky' - that's normal for true random)")


player_perception_test()
```

---

## Weighted Random (가중치 랜덤)

### Concept (개념)

**Weighted random** assigns different probabilities to different outcomes.

**Implementation:**
1. Calculate total weight
2. Generate random number from 0 to total
3. Sum weights until exceeding random number

```python
items = [("Common", 70), ("Rare", 25), ("Epic", 5)]
# Common: 70%, Rare: 25%, Epic: 5%
```

### Game Example
- Loot tables
- Enemy spawn rates
- Music shuffle with preferences
- AI decision making

### Code Example
```python
import random

def weighted_random(items):
    """
    Select item based on weights.
    items: list of (item, weight) tuples
    """
    total = sum(weight for _, weight in items)
    r = random.uniform(0, total)

    cumulative = 0
    for item, weight in items:
        cumulative += weight
        if r <= cumulative:
            return item

    return items[-1][0]  # Fallback


# Loot table
loot_table = [
    ("Gold (10-50)", 50),
    ("Potion", 25),
    ("Weapon", 15),
    ("Rare Gem", 8),
    ("Legendary Item", 2),
]

# Simulate drops
drops = {}
for _ in range(10000):
    item = weighted_random(loot_table)
    drops[item] = drops.get(item, 0) + 1

print("Weighted loot drops (10000 samples):")
total_weight = sum(w for _, w in loot_table)
for item, weight in loot_table:
    expected_pct = weight / total_weight * 100
    actual = drops.get(item, 0)
    actual_pct = actual / 10000 * 100
    print(f"  {item:20}: expected {expected_pct:5.1f}%, got {actual_pct:5.1f}%")


# Dynamic weighted random (weights change based on game state)
class AdaptiveLoot:
    def __init__(self):
        self.base_weights = {
            "Health Potion": 30,
            "Mana Potion": 30,
            "Gold": 40,
        }

    def get_drop(self, player_health_pct, player_mana_pct):
        """Adjust drops based on player needs"""
        weights = self.base_weights.copy()

        # More health potions when low health
        if player_health_pct < 0.3:
            weights["Health Potion"] *= 2

        # More mana potions when low mana
        if player_mana_pct < 0.3:
            weights["Mana Potion"] *= 2

        items = list(weights.items())
        return weighted_random(items)


# Test adaptive loot
loot = AdaptiveLoot()
print("\nAdaptive loot (low health player):")
drops = {}
for _ in range(1000):
    item = loot.get_drop(player_health_pct=0.2, player_mana_pct=0.8)
    drops[item] = drops.get(item, 0) + 1

for item, count in sorted(drops.items()):
    print(f"  {item}: {count/10:.1f}%")
```

---

## Gacha/Drop Rate Design (가챠/드롭 확률 설계)

### Concept (개념)

**Gacha systems** need careful probability design:

**Common rates:**
- Common: 50-60%
- Uncommon: 25-30%
- Rare: 10-15%
- Epic: 3-5%
- Legendary: 0.5-1%

**Considerations:**
- Legal disclosure requirements
- Player satisfaction
- Monetization balance
- Duplicate handling

### Game Example
- Character gacha (Genshin Impact, FGO)
- Loot boxes
- Card packs
- Equipment upgrade systems

### Code Example
```python
import random

class GachaSystem:
    def __init__(self):
        self.rates = [
            ("3-Star (Common)", 0.50),
            ("4-Star (Rare)", 0.35),
            ("5-Star (Epic)", 0.12),
            ("6-Star (Legendary)", 0.03),
        ]

        # Pool of items per rarity
        self.pools = {
            "3-Star (Common)": ["Sword", "Shield", "Potion", "Scroll"],
            "4-Star (Rare)": ["Fire Sword", "Ice Shield", "Dragon Scale"],
            "5-Star (Epic)": ["Phoenix Blade", "Titan Armor"],
            "6-Star (Legendary)": ["Excalibur", "Aegis"],
        }

    def pull(self):
        """Single gacha pull"""
        r = random.random()
        cumulative = 0

        for rarity, rate in self.rates:
            cumulative += rate
            if r < cumulative:
                item = random.choice(self.pools[rarity])
                return rarity, item

        # Fallback
        rarity = self.rates[-1][0]
        return rarity, random.choice(self.pools[rarity])

    def multi_pull(self, count=10):
        """Multiple pulls with guaranteed rare"""
        results = []
        has_rare = False

        for i in range(count):
            rarity, item = self.pull()
            results.append((rarity, item))
            if "4-Star" in rarity or "5-Star" in rarity or "6-Star" in rarity:
                has_rare = True

        # Guarantee at least one 4-star in 10-pull
        if not has_rare:
            # Replace last common with guaranteed rare
            rarity = "4-Star (Rare)"
            item = random.choice(self.pools[rarity])
            results[-1] = (rarity, item)

        return results


# Test gacha
gacha = GachaSystem()

print("Single pulls (10):")
for i in range(10):
    rarity, item = gacha.pull()
    print(f"  Pull {i+1}: [{rarity[:6]}] {item}")

print("\n10-pull (guaranteed 4-star+):")
results = gacha.multi_pull(10)
for i, (rarity, item) in enumerate(results):
    star = rarity.split()[0]
    print(f"  {i+1}. [{star}] {item}")


# Statistics test
print("\nRate verification (10000 pulls):")
counts = {}
for _ in range(10000):
    rarity, _ = gacha.pull()
    counts[rarity] = counts.get(rarity, 0) + 1

for rarity, rate in gacha.rates:
    expected = rate * 100
    actual = counts.get(rarity, 0) / 100
    print(f"  {rarity}: expected {expected:.1f}%, got {actual:.1f}%")
```

---

## Pseudo Random (의사 난수)

### Concept (개념)

**Pseudo Random Distribution (PRD)** adjusts probability based on previous failures:

- Start with lower than stated chance
- Increase after each failure
- Reset after success

This guarantees the **average** matches stated rate while preventing long unlucky streaks.

**Example:** 25% stated rate with PRD
- 1st try: ~8% actual chance
- 2nd try: ~16%
- 3rd try: ~24%
- ...continues until hit

### Game Example
- Dota 2 uses PRD for critical hits
- Prevents both lucky streaks (unfair advantage) and unlucky streaks (frustration)

### Code Example
```python
import random

class PseudoRandom:
    """
    Pseudo Random Distribution - prevents long streaks
    """
    def __init__(self, nominal_rate):
        self.nominal_rate = nominal_rate
        self.c = self._calculate_c(nominal_rate)
        self.current_chance = self.c
        self.failures = 0

    def _calculate_c(self, p):
        """
        Calculate base chance C that gives average rate P.
        Simplified approximation.
        """
        # More accurate calculation would use iterative solving
        # This is a reasonable approximation
        if p >= 0.5:
            return p
        return p * 0.56  # Rough approximation

    def roll(self):
        """Roll with PRD"""
        if random.random() < self.current_chance:
            # Success - reset
            self.current_chance = self.c
            self.failures = 0
            return True
        else:
            # Failure - increase chance
            self.failures += 1
            self.current_chance += self.c
            return False


class TrueRandom:
    """Standard random for comparison"""
    def __init__(self, rate):
        self.rate = rate
        self.failures = 0

    def roll(self):
        if random.random() < self.rate:
            self.failures = 0
            return True
        else:
            self.failures += 1
            return False


def compare_distributions():
    rate = 0.25
    trials = 10000

    prd = PseudoRandom(rate)
    true = TrueRandom(rate)

    prd_hits = 0
    true_hits = 0
    prd_max_streak = 0
    true_max_streak = 0
    prd_streak = 0
    true_streak = 0

    for _ in range(trials):
        # PRD
        if prd.roll():
            prd_hits += 1
            prd_max_streak = max(prd_max_streak, prd_streak)
            prd_streak = 0
        else:
            prd_streak += 1

        # True random
        if true.roll():
            true_hits += 1
            true_max_streak = max(true_max_streak, true_streak)
            true_streak = 0
        else:
            true_streak += 1

    print(f"Comparison (25% rate, {trials} trials):")
    print(f"\n  True Random:")
    print(f"    Hit rate: {true_hits/trials*100:.1f}%")
    print(f"    Longest miss streak: {true_max_streak}")

    print(f"\n  Pseudo Random (PRD):")
    print(f"    Hit rate: {prd_hits/trials*100:.1f}%")
    print(f"    Longest miss streak: {prd_max_streak}")
    print(f"    (Note: PRD has similar average but shorter streaks)")


compare_distributions()
```

---

## Pity System (천장 시스템)

### Concept (개념)

**Pity system** guarantees a reward after a maximum number of failures.

**Types:**
1. **Hard pity:** Guaranteed at exact count (e.g., 90 pulls)
2. **Soft pity:** Increased rates before hard pity (e.g., rate up at 75+ pulls)
3. **Reset on success:** Counter resets when you get the reward
4. **Carry over:** Pity carries between banners

### Game Example
- Genshin Impact: 5-star guaranteed at 90 pulls, soft pity starts at 75
- Many gacha games use variations

### Code Example
```python
import random

class PityGacha:
    """Gacha with soft and hard pity"""

    def __init__(self):
        self.base_5star_rate = 0.006  # 0.6%
        self.soft_pity_start = 75
        self.hard_pity = 90
        self.soft_pity_increase = 0.06  # +6% per pull after soft pity

        self.pity_counter = 0

    def get_current_rate(self):
        """Calculate current 5-star rate based on pity"""
        if self.pity_counter < self.soft_pity_start:
            return self.base_5star_rate
        else:
            # Soft pity: rate increases each pull
            extra_pulls = self.pity_counter - self.soft_pity_start
            return self.base_5star_rate + (extra_pulls * self.soft_pity_increase)

    def pull(self):
        """Single pull with pity"""
        self.pity_counter += 1

        # Hard pity - guaranteed 5-star
        if self.pity_counter >= self.hard_pity:
            self.pity_counter = 0
            return "5-Star", True, "Hard Pity"

        # Check for 5-star with current rate
        rate = self.get_current_rate()
        if random.random() < rate:
            self.pity_counter = 0
            return "5-Star", True, f"Rate: {rate*100:.1f}%"

        # Not 5-star
        return "4-Star or below", False, f"Pity: {self.pity_counter}/{self.hard_pity}"

    def simulate_to_5star(self):
        """Pull until 5-star, return pull count"""
        pulls = 0
        while True:
            result, got_5star, _ = self.pull()
            pulls += 1
            if got_5star:
                return pulls


# Test pity system
gacha = PityGacha()

print("Pity System Demonstration:")
print(f"  Base 5-star rate: {gacha.base_5star_rate*100:.1f}%")
print(f"  Soft pity starts: pull {gacha.soft_pity_start}")
print(f"  Hard pity (guaranteed): pull {gacha.hard_pity}")

# Show rate increase during soft pity
print("\nRate progression during soft pity:")
gacha.pity_counter = 70
for i in range(70, 91):
    gacha.pity_counter = i
    rate = gacha.get_current_rate()
    bar = "█" * int(rate * 100)
    print(f"  Pull {i:2}: {rate*100:5.1f}% {bar}")

# Simulate many 5-star acquisitions
print("\nSimulation: Pulls needed for 5-star (1000 samples)")
gacha = PityGacha()
pull_counts = []

for _ in range(1000):
    pulls = gacha.simulate_to_5star()
    pull_counts.append(pulls)

avg = sum(pull_counts) / len(pull_counts)
minimum = min(pull_counts)
maximum = max(pull_counts)

print(f"  Average pulls: {avg:.1f}")
print(f"  Minimum: {minimum}")
print(f"  Maximum: {maximum} (should be ≤ 90)")

# Distribution
print("\nDistribution:")
ranges = [(1, 10), (11, 30), (31, 50), (51, 74), (75, 89), (90, 90)]
for low, high in ranges:
    count = sum(1 for p in pull_counts if low <= p <= high)
    pct = count / 10
    bar = "█" * int(pct)
    print(f"  Pulls {low:2}-{high:2}: {pct:5.1f}% {bar}")
```

---

## Summary (요약)

| System | Korean | Purpose |
|--------|--------|---------|
| Weighted Random | 가중치 랜덤 | Different probabilities per item |
| Pseudo Random | 의사 난수 | Reduce streaks, same average |
| Soft Pity | 소프트 천장 | Gradually increase rate |
| Hard Pity | 하드 천장 | Guarantee after max attempts |

**Design principles:**
- True random feels unfair
- Manipulation improves player experience
- Disclose rates (legally required in many regions)
- Balance monetization with fairness

---

## Practice Problems (연습 문제)

1. Create a weighted random with: Common (60%), Rare (30%), Legendary (10%).

2. Design a PRD system for 15% critical hit rate. What should the starting chance be?

3. A gacha has 1% legendary rate with hard pity at 100. What's the expected pulls for a legendary?

4. Implement a "bad luck protection" system that guarantees at least one success in every 10 attempts at a 20% rate.
