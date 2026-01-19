# Chapter 02. Basic Operations (기초 연산)

> Calculating distances and midpoints - essential for detecting enemies, triggering events, and measuring game space.

---

## Distance and the Pythagorean Theorem (거리와 피타고라스 정리)

### Concept (개념)
The **Pythagorean theorem (피타고라스 정리)** states that in a right triangle:

```
a² + b² = c²
```

Where:
- a and b are the two shorter sides (legs)
- c is the longest side (hypotenuse, 빗변)

This theorem is the foundation for calculating distances in games.

### Game Example
When a character moves diagonally, they don't just move in X or Y - they move along the hypotenuse. If a character moves 3 units right and 4 units up, the actual distance traveled is:
- √(3² + 4²) = √(9 + 16) = √25 = 5 units

### Code Example
```python
import math

def diagonal_distance(dx, dy):
    """Calculate diagonal distance using Pythagorean theorem"""
    return math.sqrt(dx * dx + dy * dy)

# Character moves 3 right, 4 up
movement = diagonal_distance(3, 4)
print(f"Actual distance moved: {movement}")  # 5.0
```

---

## Distance Between Two Points (두 점 사이의 거리)

### Concept (개념)
To find the distance between two points (x1, y1) and (x2, y2):

1. Find the horizontal difference: dx = x2 - x1
2. Find the vertical difference: dy = y2 - y1
3. Apply Pythagorean theorem: distance = √(dx² + dy²)

**Formula (공식)**:
```
distance = √((x2 - x1)² + (y2 - y1)²)
```

### Game Example
Checking if the player is close enough to interact with an NPC:
- Player at (100, 150)
- NPC at (130, 190)
- Distance = √((130-100)² + (190-150)²) = √(900 + 1600) = √2500 = 50

If interaction range is 60, the player can interact!

### Code Example
```python
import math

def distance_between_points(x1, y1, x2, y2):
    """Calculate distance between two 2D points"""
    dx = x2 - x1
    dy = y2 - y1
    return math.sqrt(dx * dx + dy * dy)

# Alternative using Vector2 class
class Vector2:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def distance_to(self, other):
        dx = other.x - self.x
        dy = other.y - self.y
        return math.sqrt(dx * dx + dy * dy)

# Example: Player and NPC
player = Vector2(100, 150)
npc = Vector2(130, 190)

dist = player.distance_to(npc)
print(f"Distance to NPC: {dist}")  # 50.0

# Check interaction
INTERACTION_RANGE = 60
if dist <= INTERACTION_RANGE:
    print("Press E to interact")
```

---

## Distance in 3D (3D 거리)

### Concept (개념)
For 3D, we extend the formula to include Z:

```
distance = √((x2 - x1)² + (y2 - y1)² + (z2 - z1)²)
```

### Game Example
In a 3D shooter, checking if an enemy is within weapon range requires 3D distance calculation.

### Code Example
```python
import math

def distance_3d(x1, y1, z1, x2, y2, z2):
    """Calculate distance between two 3D points"""
    dx = x2 - x1
    dy = y2 - y1
    dz = z2 - z1
    return math.sqrt(dx*dx + dy*dy + dz*dz)

class Vector3:
    def __init__(self, x, y, z):
        self.x = x
        self.y = y
        self.z = z

    def distance_to(self, other):
        dx = other.x - self.x
        dy = other.y - self.y
        dz = other.z - self.z
        return math.sqrt(dx*dx + dy*dy + dz*dz)

# Player and enemy in 3D space
player = Vector3(10, 5, 20)
enemy = Vector3(15, 8, 25)

dist = player.distance_to(enemy)
print(f"Distance to enemy: {dist:.2f}")  # ~7.07
```

---

## Squared Distance Optimization (제곱 거리 최적화)

### Concept (개념)
The square root operation (√) is computationally expensive. If you're only comparing distances (not needing the actual value), you can skip the square root and compare squared distances instead.

```
distance² = dx² + dy²
```

If distance_a < distance_b, then distance_a² < distance_b² (for positive values)

### Game Example
When finding the nearest enemy among many, comparing squared distances is much faster.

### Code Example
```python
def distance_squared(x1, y1, x2, y2):
    """Calculate squared distance (no sqrt - faster!)"""
    dx = x2 - x1
    dy = y2 - y1
    return dx * dx + dy * dy

# Check if player is within range of 50
player_pos = (100, 100)
enemy_pos = (130, 140)

RANGE = 50
RANGE_SQUARED = RANGE * RANGE  # 2500

dist_sq = distance_squared(*player_pos, *enemy_pos)
# dist_sq = 30² + 40² = 900 + 1600 = 2500

if dist_sq <= RANGE_SQUARED:
    print("Enemy in range!")
else:
    print("Enemy out of range")

# Finding nearest enemy efficiently
import math

def find_nearest_enemy(player_pos, enemies):
    """Find nearest enemy using squared distance for efficiency"""
    nearest = None
    nearest_dist_sq = float('inf')

    for enemy in enemies:
        dist_sq = distance_squared(
            player_pos[0], player_pos[1],
            enemy['x'], enemy['y']
        )
        if dist_sq < nearest_dist_sq:
            nearest_dist_sq = dist_sq
            nearest = enemy

    # Only calculate actual distance for the result
    actual_distance = math.sqrt(nearest_dist_sq) if nearest else 0
    return nearest, actual_distance

enemies = [
    {'name': 'Goblin', 'x': 150, 'y': 150},
    {'name': 'Orc', 'x': 110, 'y': 105},
    {'name': 'Dragon', 'x': 500, 'y': 500}
]

nearest, dist = find_nearest_enemy((100, 100), enemies)
print(f"Nearest enemy: {nearest['name']} at distance {dist:.2f}")
```

---

## Midpoint (중점)

### Concept (개념)
The **midpoint (중점)** is the point exactly halfway between two points.

**Formula (공식)**:
```
midpoint_x = (x1 + x2) / 2
midpoint_y = (y1 + y2) / 2
```

### Game Example
Uses include:
- Finding the center of a line segment
- Camera positioning between two characters
- Spawning effects between two points

### Code Example
```python
def midpoint(x1, y1, x2, y2):
    """Calculate the midpoint between two points"""
    return ((x1 + x2) / 2, (y1 + y2) / 2)

class Vector2:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    @staticmethod
    def midpoint(a, b):
        """Find midpoint between two Vector2s"""
        return Vector2((a.x + b.x) / 2, (a.y + b.y) / 2)

    def __repr__(self):
        return f"({self.x}, {self.y})"

# Two players in co-op game
player1 = Vector2(100, 200)
player2 = Vector2(300, 400)

# Camera should focus on midpoint
camera_target = Vector2.midpoint(player1, player2)
print(f"Camera position: {camera_target}")  # (200.0, 300.0)
```

---

## Game Applications (게임 활용)

### Enemy Detection Range (적 감지 범위)

```python
import math

class Enemy:
    def __init__(self, x, y, detection_range=100):
        self.x = x
        self.y = y
        self.detection_range = detection_range
        self.state = "patrol"

    def update(self, player_x, player_y):
        # Calculate distance to player
        dist_sq = (player_x - self.x)**2 + (player_y - self.y)**2
        range_sq = self.detection_range ** 2

        if dist_sq <= range_sq:
            self.state = "chase"
            print(f"Enemy detected player! Distance: {math.sqrt(dist_sq):.1f}")
        else:
            self.state = "patrol"

# Create enemy with detection range of 100
enemy = Enemy(200, 200, detection_range=100)

# Player positions
enemy.update(150, 150)  # Distance ~70.7 - Detected!
enemy.update(50, 50)    # Distance ~212 - Not detected
```

### Collision Detection (충돌 감지)

```python
def circles_collide(x1, y1, r1, x2, y2, r2):
    """Check if two circles collide using distance"""
    dist_sq = (x2 - x1)**2 + (y2 - y1)**2
    radii_sum = r1 + r2
    radii_sum_sq = radii_sum ** 2

    return dist_sq <= radii_sum_sq

# Player circle and coin circle
player = {'x': 100, 'y': 100, 'radius': 20}
coin = {'x': 115, 'y': 110, 'radius': 10}

if circles_collide(
    player['x'], player['y'], player['radius'],
    coin['x'], coin['y'], coin['radius']
):
    print("Coin collected!")
```

### Area of Effect (범위 공격)

```python
def get_enemies_in_range(center_x, center_y, radius, enemies):
    """Find all enemies within a circular area"""
    radius_sq = radius ** 2
    affected = []

    for enemy in enemies:
        dist_sq = (enemy['x'] - center_x)**2 + (enemy['y'] - center_y)**2
        if dist_sq <= radius_sq:
            affected.append(enemy)

    return affected

# Explosion at position (200, 200) with radius 80
explosion_x, explosion_y = 200, 200
explosion_radius = 80

enemies = [
    {'name': 'Goblin A', 'x': 220, 'y': 220},  # Distance ~28
    {'name': 'Goblin B', 'x': 300, 'y': 200},  # Distance 100
    {'name': 'Goblin C', 'x': 180, 'y': 250},  # Distance ~54
]

hit_enemies = get_enemies_in_range(explosion_x, explosion_y, explosion_radius, enemies)
print(f"Enemies hit by explosion: {[e['name'] for e in hit_enemies]}")
# Output: ['Goblin A', 'Goblin C']
```

---

## Summary (요약)

| Operation | Korean | Formula |
|-----------|--------|---------|
| Pythagorean Theorem | 피타고라스 정리 | a² + b² = c² |
| 2D Distance | 2D 거리 | √((x2-x1)² + (y2-y1)²) |
| 3D Distance | 3D 거리 | √((x2-x1)² + (y2-y1)² + (z2-z1)²) |
| Squared Distance | 제곱 거리 | (x2-x1)² + (y2-y1)² |
| Midpoint | 중점 | ((x1+x2)/2, (y1+y2)/2) |

---

## Practice Problems (연습 문제)

1. Calculate the distance between player at (50, 80) and enemy at (110, 160).
2. Find the midpoint between spawn point A (0, 0) and spawn point B (400, 300).
3. Check if a player at (100, 100) with radius 25 collides with an enemy at (140, 130) with radius 20.
4. An enemy has detection range 75. Player is at (200, 200), enemy at (260, 230). Is player detected?
