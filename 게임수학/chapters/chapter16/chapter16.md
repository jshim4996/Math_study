# Chapter 16. Circle/Sphere Collision (원/구 충돌)

> The most common collision shape in games: fast, rotation-invariant, and easy to understand.

---

## Circle vs Circle Collision (2D) (원 vs 원 충돌)

### Concept (개념)

Two circles collide if the distance between their centers is less than the sum of their radii:

```
distance(center1, center2) < radius1 + radius2
```

**Optimization:** Compare squared values:
```
distanceSquared < (radius1 + radius2)²
```

**Benefits of circles:**
- Rotation doesn't affect the shape
- Very fast collision check
- Works well for round-ish objects

### Game Example
- Player vs enemies
- Ball physics
- Particle collisions
- Character hitboxes

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def distance_squared_to(self, other):
        dx = self.x - other.x
        dy = self.y - other.y
        return dx*dx + dy*dy

    def distance_to(self, other):
        return math.sqrt(self.distance_squared_to(other))

    def __sub__(self, other):
        return Vector2(self.x - other.x, self.y - other.y)

    def normalized(self):
        mag = math.sqrt(self.x*self.x + self.y*self.y)
        if mag == 0:
            return Vector2(0, 0)
        return Vector2(self.x / mag, self.y / mag)

    def __mul__(self, scalar):
        return Vector2(self.x * scalar, self.y * scalar)

    def __repr__(self):
        return f"({self.x:.1f}, {self.y:.1f})"


class Circle:
    def __init__(self, x, y, radius):
        self.center = Vector2(x, y)
        self.radius = radius

    def intersects(self, other):
        """Check if two circles overlap"""
        combined_radius = self.radius + other.radius
        distance_sq = self.center.distance_squared_to(other.center)
        return distance_sq < combined_radius * combined_radius

    def get_collision_info(self, other):
        """Get detailed collision information"""
        dist = self.center.distance_to(other.center)
        combined_radius = self.radius + other.radius

        if dist >= combined_radius:
            return None  # No collision

        # Calculate penetration depth and normal
        penetration = combined_radius - dist
        if dist > 0:
            normal = (other.center - self.center).normalized()
        else:
            normal = Vector2(1, 0)  # Default if centers overlap exactly

        return {
            'penetration': penetration,
            'normal': normal,
            'contact_point': Vector2(
                self.center.x + normal.x * self.radius,
                self.center.y + normal.y * self.radius
            )
        }


# Basic collision test
player = Circle(100, 100, 25)
enemy1 = Circle(140, 110, 20)   # Overlapping
enemy2 = Circle(200, 100, 20)   # Not overlapping

print(f"Player: center={player.center}, radius={player.radius}")
print(f"Enemy1: center={enemy1.center}, radius={enemy1.radius}")
print(f"Enemy2: center={enemy2.center}, radius={enemy2.radius}")

print(f"\nPlayer vs Enemy1: {player.intersects(enemy1)}")
print(f"Player vs Enemy2: {player.intersects(enemy2)}")

# Detailed collision info
info = player.get_collision_info(enemy1)
if info:
    print(f"\nCollision details:")
    print(f"  Penetration: {info['penetration']:.2f}")
    print(f"  Normal: {info['normal']}")
    print(f"  Contact point: {info['contact_point']}")


# Multiple collision check
enemies = [
    Circle(120, 90, 15),
    Circle(80, 130, 20),
    Circle(200, 200, 30),
    Circle(90, 100, 10),
]

print("\nChecking player against all enemies:")
for i, enemy in enumerate(enemies):
    if player.intersects(enemy):
        dist = player.center.distance_to(enemy.center)
        print(f"  Enemy {i}: COLLISION (distance={dist:.1f})")
    else:
        dist = player.center.distance_to(enemy.center)
        print(f"  Enemy {i}: clear (distance={dist:.1f})")
```

---

## Sphere vs Sphere Collision (3D) (구 vs 구 충돌)

### Concept (개념)

3D sphere collision is identical to 2D circle collision, just with an extra dimension:

```
distance(center1, center2) < radius1 + radius2
```

```
dx² + dy² + dz² < (r1 + r2)²
```

### Game Example
- 3D character collision
- Particle systems
- Physics simulations
- Broad-phase collision in 3D games

### Code Example
```python
import math

class Vector3:
    def __init__(self, x=0, y=0, z=0):
        self.x = x
        self.y = y
        self.z = z

    def distance_squared_to(self, other):
        dx = self.x - other.x
        dy = self.y - other.y
        dz = self.z - other.z
        return dx*dx + dy*dy + dz*dz

    def distance_to(self, other):
        return math.sqrt(self.distance_squared_to(other))

    def __sub__(self, other):
        return Vector3(self.x - other.x, self.y - other.y, self.z - other.z)

    def normalized(self):
        mag = math.sqrt(self.x**2 + self.y**2 + self.z**2)
        if mag == 0:
            return Vector3(0, 0, 0)
        return Vector3(self.x/mag, self.y/mag, self.z/mag)

    def __repr__(self):
        return f"({self.x:.1f}, {self.y:.1f}, {self.z:.1f})"


class Sphere:
    def __init__(self, x, y, z, radius):
        self.center = Vector3(x, y, z)
        self.radius = radius

    def intersects(self, other):
        """Check if two spheres overlap"""
        combined_radius = self.radius + other.radius
        dist_sq = self.center.distance_squared_to(other.center)
        return dist_sq < combined_radius * combined_radius

    def get_penetration(self, other):
        """Get penetration depth (negative if not colliding)"""
        dist = self.center.distance_to(other.center)
        combined_radius = self.radius + other.radius
        return combined_radius - dist


# 3D collision example
player = Sphere(0, 1, 0, 0.5)  # Player at (0, 1, 0), radius 0.5
obstacle1 = Sphere(0.8, 1, 0, 0.4)  # Close obstacle
obstacle2 = Sphere(3, 1, 2, 1.0)    # Far obstacle

print("3D Sphere Collision:")
print(f"Player: {player.center}, r={player.radius}")
print(f"Obstacle1: {obstacle1.center}, r={obstacle1.radius}")
print(f"Obstacle2: {obstacle2.center}, r={obstacle2.radius}")

print(f"\nPlayer vs Obstacle1: {player.intersects(obstacle1)}")
print(f"  Penetration: {player.get_penetration(obstacle1):.2f}")

print(f"\nPlayer vs Obstacle2: {player.intersects(obstacle2)}")
print(f"  Penetration: {player.get_penetration(obstacle2):.2f}")


# Batch collision check for game objects
class GameObject3D:
    def __init__(self, name, x, y, z, radius):
        self.name = name
        self.sphere = Sphere(x, y, z, radius)

    def collides_with(self, other):
        return self.sphere.intersects(other.sphere)


objects = [
    GameObject3D("Player", 0, 0, 0, 1),
    GameObject3D("Enemy1", 1.5, 0, 0, 0.8),
    GameObject3D("Enemy2", 5, 0, 0, 1),
    GameObject3D("Coin", 0, 1.2, 0, 0.3),
]

print("\n--- Collision Matrix ---")
for i, obj1 in enumerate(objects):
    for j, obj2 in enumerate(objects):
        if i < j:
            if obj1.collides_with(obj2):
                print(f"{obj1.name} ↔ {obj2.name}: COLLISION")
```

---

## Circle vs AABB Collision (원 vs AABB 충돌)

### Concept (개념)

To check circle vs AABB:
1. Find the **closest point** on the AABB to the circle's center
2. Check if that point is within the circle's radius

**Finding closest point:**
```python
closest_x = clamp(circle.x, aabb.min_x, aabb.max_x)
closest_y = clamp(circle.y, aabb.min_y, aabb.max_y)
```

Then: `distance(closest, circle.center) ≤ circle.radius`

### Game Example
- Round player vs rectangular obstacles
- Ball vs paddle in breakout
- Circular hitbox vs rectangular trigger zones

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def distance_squared_to(self, other):
        dx = self.x - other.x
        dy = self.y - other.y
        return dx*dx + dy*dy


class Circle:
    def __init__(self, x, y, radius):
        self.center = Vector2(x, y)
        self.radius = radius


class AABB:
    def __init__(self, x, y, width, height):
        self.min_x = x
        self.min_y = y
        self.max_x = x + width
        self.max_y = y + height

    @property
    def center(self):
        return Vector2(
            (self.min_x + self.max_x) / 2,
            (self.min_y + self.max_y) / 2
        )


def clamp(value, min_val, max_val):
    """Clamp value to range [min_val, max_val]"""
    return max(min_val, min(value, max_val))


def circle_vs_aabb(circle, aabb):
    """Check if circle intersects AABB"""
    # Find closest point on AABB to circle center
    closest_x = clamp(circle.center.x, aabb.min_x, aabb.max_x)
    closest_y = clamp(circle.center.y, aabb.min_y, aabb.max_y)

    closest = Vector2(closest_x, closest_y)

    # Check if closest point is within circle
    dist_sq = circle.center.distance_squared_to(closest)
    return dist_sq < circle.radius * circle.radius


def circle_aabb_collision_info(circle, aabb):
    """Get detailed collision info for circle vs AABB"""
    closest_x = clamp(circle.center.x, aabb.min_x, aabb.max_x)
    closest_y = clamp(circle.center.y, aabb.min_y, aabb.max_y)

    dx = circle.center.x - closest_x
    dy = circle.center.y - closest_y
    dist_sq = dx*dx + dy*dy

    if dist_sq >= circle.radius * circle.radius:
        return None  # No collision

    dist = math.sqrt(dist_sq)

    # Calculate normal (from AABB toward circle)
    if dist > 0:
        normal = Vector2(dx / dist, dy / dist)
    else:
        # Circle center is inside AABB
        # Find closest edge
        distances = [
            (circle.center.x - aabb.min_x, Vector2(-1, 0)),  # Left
            (aabb.max_x - circle.center.x, Vector2(1, 0)),   # Right
            (circle.center.y - aabb.min_y, Vector2(0, -1)),  # Bottom
            (aabb.max_y - circle.center.y, Vector2(0, 1)),   # Top
        ]
        _, normal = min(distances, key=lambda x: x[0])

    penetration = circle.radius - dist

    return {
        'closest_point': Vector2(closest_x, closest_y),
        'penetration': penetration,
        'normal': normal
    }


# Ball vs paddle example (Breakout game)
ball = Circle(150, 95, 10)
paddle = AABB(100, 80, 100, 15)

print(f"Ball: center=({ball.center.x}, {ball.center.y}), radius={ball.radius}")
print(f"Paddle: ({paddle.min_x}, {paddle.min_y}) to ({paddle.max_x}, {paddle.max_y})")

collision = circle_vs_aabb(ball, paddle)
print(f"\nCollision: {collision}")

if collision:
    info = circle_aabb_collision_info(ball, paddle)
    print(f"  Closest point: ({info['closest_point'].x}, {info['closest_point'].y})")
    print(f"  Penetration: {info['penetration']:.2f}")
    print(f"  Normal: ({info['normal'].x}, {info['normal'].y})")


# Multiple obstacle test
player = Circle(100, 100, 20)
obstacles = [
    AABB(50, 50, 30, 30),    # Should collide
    AABB(150, 150, 40, 40),  # Should not collide
    AABB(90, 110, 15, 15),   # Should collide
]

print("\n--- Player vs Obstacles ---")
for i, obs in enumerate(obstacles):
    if circle_vs_aabb(player, obs):
        print(f"Obstacle {i}: COLLISION")
    else:
        print(f"Obstacle {i}: clear")
```

---

## Collision Response (충돌 반응)

### Concept (개념)

After detecting collision, we need to **respond**:
1. **Separation**: Push objects apart to remove overlap
2. **Velocity change**: Reflect, stop, or bounce

**Simple separation:**
```python
# Push circle away from other circle
push_direction = (this.center - other.center).normalized()
push_amount = penetration_depth / 2  # Split between both objects
this.center += push_direction * push_amount
other.center -= push_direction * push_amount
```

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __add__(self, other):
        return Vector2(self.x + other.x, self.y + other.y)

    def __sub__(self, other):
        return Vector2(self.x - other.x, self.y - other.y)

    def __mul__(self, scalar):
        return Vector2(self.x * scalar, self.y * scalar)

    def magnitude(self):
        return math.sqrt(self.x*self.x + self.y*self.y)

    def normalized(self):
        mag = self.magnitude()
        if mag == 0:
            return Vector2(0, 0)
        return Vector2(self.x / mag, self.y / mag)

    def __repr__(self):
        return f"({self.x:.2f}, {self.y:.2f})"


class Ball:
    def __init__(self, x, y, radius, vx=0, vy=0):
        self.position = Vector2(x, y)
        self.velocity = Vector2(vx, vy)
        self.radius = radius

    def update(self, dt):
        self.position = self.position + self.velocity * dt


def resolve_circle_collision(ball1, ball2):
    """Separate two colliding circles"""
    diff = ball1.position - ball2.position
    dist = diff.magnitude()
    combined_radius = ball1.radius + ball2.radius

    if dist >= combined_radius or dist == 0:
        return False  # No collision

    # Calculate penetration
    penetration = combined_radius - dist

    # Separation direction
    normal = diff.normalized()

    # Push apart (50/50 split)
    separation = normal * (penetration / 2 + 0.01)  # Small buffer
    ball1.position = ball1.position + separation
    ball2.position = ball2.position - separation

    return True


def resolve_with_bounce(ball1, ball2, restitution=1.0):
    """Resolve collision with velocity change (elastic bounce)"""
    diff = ball1.position - ball2.position
    dist = diff.magnitude()

    if dist == 0:
        return

    normal = diff.normalized()

    # Relative velocity
    rel_vel = ball1.velocity - ball2.velocity

    # Velocity along collision normal
    vel_along_normal = rel_vel.x * normal.x + rel_vel.y * normal.y

    # Only resolve if objects are moving toward each other
    if vel_along_normal > 0:
        return

    # Simple elastic collision (equal mass)
    ball1.velocity = ball1.velocity - normal * vel_along_normal
    ball2.velocity = ball2.velocity + normal * vel_along_normal


# Simulation
ball1 = Ball(100, 100, 20, 50, 0)   # Moving right
ball2 = Ball(130, 100, 20, -30, 0)  # Moving left

print("Before collision:")
print(f"  Ball1: pos={ball1.position}, vel={ball1.velocity}")
print(f"  Ball2: pos={ball2.position}, vel={ball2.velocity}")

# Check and resolve
if resolve_circle_collision(ball1, ball2):
    print("\nCollision detected! Positions separated.")

resolve_with_bounce(ball1, ball2)
print("\nAfter collision response:")
print(f"  Ball1: pos={ball1.position}, vel={ball1.velocity}")
print(f"  Ball2: pos={ball2.position}, vel={ball2.velocity}")
```

---

## Summary (요약)

| Collision Type | Korean | Test |
|---------------|--------|------|
| Circle vs Circle | 원-원 | distance < r1 + r2 |
| Sphere vs Sphere | 구-구 | Same, in 3D |
| Circle vs AABB | 원-사각형 | closest point distance < r |

**Optimization tips:**
- Compare squared distances (avoid sqrt)
- Use broad-phase first (AABB) before circle tests
- Early exit when possible

---

## Practice Problems (연습 문제)

1. Two circles: A at (0, 0) r=5, B at (8, 0) r=4. Do they collide? What's the penetration?

2. Circle at (10, 10) r=5 and AABB from (0, 0) to (8, 8). Do they collide?

3. Implement a function that finds all colliding pairs in a list of circles.

4. A ball at (100, 100) with velocity (50, 0) hits a stationary ball at (150, 100). What are their velocities after an elastic collision?
