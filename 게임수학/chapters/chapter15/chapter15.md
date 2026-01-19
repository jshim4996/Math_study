# Chapter 15. Collision Detection Basics (충돌 감지 기초)

> The foundation of game interactions: detecting when objects touch or overlap.

---

## What is Collision Detection (충돌 감지란 무엇인가)

### Concept (개념)

**Collision detection** determines if two objects are touching or overlapping. It answers:
- Are these two objects colliding? (boolean)
- Where is the collision? (contact point)
- How much are they overlapping? (penetration depth)

**Two phases:**
1. **Broad phase**: Quick rejection of obviously non-colliding pairs
2. **Narrow phase**: Precise collision test for potential collisions

**Common collision shapes:**
- Point
- Circle/Sphere
- Rectangle/Box (AABB, OBB)
- Polygon/Mesh

### Game Example
Every game needs collision: bullets hitting enemies, player collecting coins, character standing on ground.

### Code Example
```python
class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __sub__(self, other):
        return Vector2(self.x - other.x, self.y - other.y)

    def magnitude(self):
        return (self.x**2 + self.y**2) ** 0.5

    def __repr__(self):
        return f"({self.x}, {self.y})"


# Basic collision framework
class Collider:
    """Base class for all collision shapes"""
    def collides_with(self, other):
        raise NotImplementedError


class PointCollider(Collider):
    def __init__(self, x, y):
        self.position = Vector2(x, y)


class CircleCollider(Collider):
    def __init__(self, x, y, radius):
        self.position = Vector2(x, y)
        self.radius = radius


class RectCollider(Collider):
    def __init__(self, x, y, width, height):
        self.x = x  # Left edge
        self.y = y  # Bottom edge
        self.width = width
        self.height = height

    @property
    def right(self):
        return self.x + self.width

    @property
    def top(self):
        return self.y + self.height


# Example usage
player = CircleCollider(100, 100, 25)
enemy = CircleCollider(120, 110, 20)
wall = RectCollider(200, 0, 50, 300)

print(f"Player: center={player.position}, radius={player.radius}")
print(f"Enemy: center={enemy.position}, radius={enemy.radius}")
print(f"Wall: ({wall.x}, {wall.y}) to ({wall.right}, {wall.top})")
```

---

## Point vs Rectangle Collision (점과 사각형 충돌)

### Concept (개념)

A point (x, y) is inside a rectangle if:
```
left ≤ x ≤ right  AND  bottom ≤ y ≤ top
```

Or equivalently:
```
rect.x ≤ point.x ≤ rect.x + rect.width
rect.y ≤ point.y ≤ rect.y + rect.height
```

### Game Example
- Mouse click inside a button
- Checking if player is within a zone
- UI hit testing

### Code Example
```python
class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y


class Rect:
    def __init__(self, x, y, width, height):
        self.x = x
        self.y = y
        self.width = width
        self.height = height

    def contains_point(self, point):
        """Check if point is inside rectangle"""
        return (self.x <= point.x <= self.x + self.width and
                self.y <= point.y <= self.y + self.height)


# Button click detection
button = Rect(100, 200, 150, 50)  # Button at (100, 200), size 150x50

# Test clicks
clicks = [
    Vector2(175, 225),  # Center of button
    Vector2(100, 200),  # Corner (still inside)
    Vector2(50, 225),   # Left of button
    Vector2(175, 300),  # Above button
]

print(f"Button: ({button.x}, {button.y}) to ({button.x + button.width}, {button.y + button.height})")
print("\nClick tests:")
for click in clicks:
    inside = button.contains_point(click)
    result = "INSIDE" if inside else "outside"
    print(f"  Click at {click.x}, {click.y}: {result}")


# Zone detection
safe_zone = Rect(0, 0, 500, 500)
player_pos = Vector2(250, 250)
enemy_pos = Vector2(600, 300)

print(f"\nSafe zone check:")
print(f"  Player at {player_pos.x}, {player_pos.y}: {'SAFE' if safe_zone.contains_point(player_pos) else 'DANGER'}")
print(f"  Enemy at {enemy_pos.x}, {enemy_pos.y}: {'SAFE' if safe_zone.contains_point(enemy_pos) else 'DANGER'}")
```

---

## Point vs Circle Collision (점과 원 충돌)

### Concept (개념)

A point is inside a circle if the distance from point to circle center is less than the radius:

```
distance(point, center) ≤ radius
```

**Optimization:** Compare squared distances to avoid square root:
```
(point.x - center.x)² + (point.y - center.y)² ≤ radius²
```

### Game Example
- Checking if mouse is over a circular button
- Area of effect (AoE) abilities
- Explosion radius

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def distance_to(self, other):
        dx = self.x - other.x
        dy = self.y - other.y
        return math.sqrt(dx*dx + dy*dy)

    def distance_squared_to(self, other):
        dx = self.x - other.x
        dy = self.y - other.y
        return dx*dx + dy*dy


class Circle:
    def __init__(self, x, y, radius):
        self.center = Vector2(x, y)
        self.radius = radius

    def contains_point(self, point):
        """Check if point is inside circle"""
        return self.center.distance_to(point) <= self.radius

    def contains_point_fast(self, point):
        """Optimized: no square root"""
        return self.center.distance_squared_to(point) <= self.radius * self.radius


# Explosion damage check
explosion = Circle(200, 200, 100)  # Explosion at (200, 200), radius 100

characters = [
    ("Player", Vector2(250, 220)),    # Close to explosion
    ("Enemy1", Vector2(350, 200)),    # At edge
    ("Enemy2", Vector2(400, 400)),    # Far away
    ("NPC", Vector2(150, 150)),       # Inside
]

print(f"Explosion at ({explosion.center.x}, {explosion.center.y}), radius {explosion.radius}")
print("\nDamage check:")
for name, pos in characters:
    dist = explosion.center.distance_to(pos)
    hit = explosion.contains_point(pos)
    result = "HIT" if hit else "safe"
    print(f"  {name} at ({pos.x}, {pos.y}): distance={dist:.1f}, {result}")


# Performance comparison
import time

point = Vector2(150, 180)

# Timing test (simplified)
start = time.time()
for _ in range(100000):
    explosion.contains_point(point)
time_normal = time.time() - start

start = time.time()
for _ in range(100000):
    explosion.contains_point_fast(point)
time_fast = time.time() - start

print(f"\nPerformance (100k iterations):")
print(f"  With sqrt: {time_normal*1000:.2f}ms")
print(f"  Without sqrt: {time_fast*1000:.2f}ms")
```

---

## AABB: Axis-Aligned Bounding Box

### Concept (개념)

**AABB** (Axis-Aligned Bounding Box) is a rectangle whose edges are parallel to the coordinate axes.

**Properties:**
- Simple and fast collision detection
- Cannot rotate (always axis-aligned)
- Defined by min/max points or position + size

**Representation:**
```
Method 1: min point + max point
  min = (left, bottom)
  max = (right, top)

Method 2: position + size
  position = (left, bottom)
  size = (width, height)
```

### Game Example
Most 2D games use AABB for collision. Even 3D games use AABB for broad-phase collision.

### Code Example
```python
class AABB:
    def __init__(self, x, y, width, height):
        """Create AABB from position and size"""
        self.min_x = x
        self.min_y = y
        self.max_x = x + width
        self.max_y = y + height

    @classmethod
    def from_center(cls, cx, cy, half_width, half_height):
        """Create AABB from center point"""
        return cls(cx - half_width, cy - half_height,
                   half_width * 2, half_height * 2)

    @property
    def width(self):
        return self.max_x - self.min_x

    @property
    def height(self):
        return self.max_y - self.min_y

    @property
    def center(self):
        return ((self.min_x + self.max_x) / 2,
                (self.min_y + self.max_y) / 2)

    def contains_point(self, x, y):
        return (self.min_x <= x <= self.max_x and
                self.min_y <= y <= self.max_y)

    def __repr__(self):
        return f"AABB({self.min_x}, {self.min_y}) to ({self.max_x}, {self.max_y})"


# Create AABBs in different ways
box1 = AABB(0, 0, 100, 50)  # Position + size
box2 = AABB.from_center(200, 100, 30, 30)  # Center + half-size

print(f"Box1: {box1}")
print(f"  Center: {box1.center}")
print(f"  Size: {box1.width} x {box1.height}")

print(f"\nBox2: {box2}")
print(f"  Center: {box2.center}")
print(f"  Size: {box2.width} x {box2.height}")

# Point containment
test_points = [(50, 25), (200, 100), (0, 0), (150, 75)]
print("\nPoint containment tests:")
for px, py in test_points:
    in_box1 = box1.contains_point(px, py)
    in_box2 = box2.contains_point(px, py)
    print(f"  Point ({px}, {py}): box1={in_box1}, box2={in_box2}")
```

---

## AABB vs AABB Collision

### Concept (개념)

Two AABBs collide if they overlap on **both** axes:

```
NOT (A.max_x < B.min_x OR   // A is left of B
     A.min_x > B.max_x OR   // A is right of B
     A.max_y < B.min_y OR   // A is below B
     A.min_y > B.max_y)     // A is above B
```

Or equivalently (collision = overlap on both axes):
```
A.max_x >= B.min_x AND
A.min_x <= B.max_x AND
A.max_y >= B.min_y AND
A.min_y <= B.max_y
```

### Game Example
Platform collision, enemy detection, item pickup ranges.

### Code Example
```python
class AABB:
    def __init__(self, x, y, width, height):
        self.min_x = x
        self.min_y = y
        self.max_x = x + width
        self.max_y = y + height

    def intersects(self, other):
        """Check if two AABBs overlap"""
        return (self.max_x >= other.min_x and
                self.min_x <= other.max_x and
                self.max_y >= other.min_y and
                self.min_y <= other.max_y)

    def get_overlap(self, other):
        """Calculate overlap amount on each axis (for collision response)"""
        if not self.intersects(other):
            return None

        overlap_x = min(self.max_x, other.max_x) - max(self.min_x, other.min_x)
        overlap_y = min(self.max_y, other.max_y) - max(self.min_y, other.min_y)
        return (overlap_x, overlap_y)

    def __repr__(self):
        return f"AABB({self.min_x}, {self.min_y}, {self.max_x - self.min_x}, {self.max_y - self.min_y})"


# Collision tests
player = AABB(100, 100, 50, 80)  # Player box
ground = AABB(0, 0, 800, 50)     # Ground platform
enemy = AABB(130, 100, 40, 60)   # Overlapping enemy
coin = AABB(300, 150, 20, 20)    # Coin pickup

print(f"Player: {player}")
print(f"Ground: {ground}")
print(f"Enemy: {enemy}")
print(f"Coin: {coin}")

print("\nCollision checks:")
print(f"  Player vs Ground: {player.intersects(ground)}")
print(f"  Player vs Enemy: {player.intersects(enemy)}")
print(f"  Player vs Coin: {player.intersects(coin)}")

# Get overlap for collision response
overlap = player.get_overlap(enemy)
if overlap:
    print(f"\nPlayer-Enemy overlap: x={overlap[0]:.1f}, y={overlap[1]:.1f}")
    # Push out along smallest overlap
    if overlap[0] < overlap[1]:
        print("  → Push horizontally")
    else:
        print("  → Push vertically")


# Batch collision checking
def check_all_collisions(boxes):
    """Check all pairs for collision"""
    collisions = []
    for i in range(len(boxes)):
        for j in range(i + 1, len(boxes)):
            if boxes[i][1].intersects(boxes[j][1]):
                collisions.append((boxes[i][0], boxes[j][0]))
    return collisions

all_objects = [
    ("Player", player),
    ("Ground", ground),
    ("Enemy", enemy),
    ("Coin", coin),
]

print("\nAll collisions:")
for a, b in check_all_collisions(all_objects):
    print(f"  {a} ↔ {b}")
```

---

## Summary (요약)

| Test | Korean | Condition |
|------|--------|-----------|
| Point in Rect | 점-사각형 | x in [left, right] AND y in [bottom, top] |
| Point in Circle | 점-원 | distance ≤ radius |
| AABB vs AABB | AABB-AABB | Overlap on BOTH axes |

**Tips:**
- Use squared distance to avoid sqrt for circle tests
- AABB is fast but can't rotate
- Check collision before detailed physics

---

## Practice Problems (연습 문제)

1. Point (75, 40) and rectangle at (50, 30) with size (50, 30). Is the point inside?

2. Point (10, 10) and circle at (15, 15) with radius 8. Is the point inside?

3. AABB A: (0, 0) to (100, 100). AABB B: (80, 50) to (150, 120). Do they collide?

4. Implement a function that returns the closest point on an AABB to a given point.
