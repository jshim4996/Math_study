# Chapter 06. Dot Product (내적)

> The dot product is your Swiss Army knife for angles, visibility checks, and lighting calculations.

---

## What is the Dot Product? (내적이란?)

### Concept (개념)
The **dot product (내적)**, also called scalar product or inner product, takes two vectors and returns a single number (scalar).

**Formula (공식)**:
```
A · B = Ax * Bx + Ay * By
```

For 3D:
```
A · B = Ax * Bx + Ay * By + Az * Bz
```

The result tells us about the **relationship** between the two vectors:
- Positive: Vectors point in similar directions (angle < 90°)
- Zero: Vectors are perpendicular (angle = 90°)
- Negative: Vectors point in opposite directions (angle > 90°)

### Game Example
Checking if an enemy is in front of or behind the player:
- If dot product of player's forward direction and direction to enemy is positive: Enemy is in front
- If negative: Enemy is behind

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def dot(self, other):
        """Calculate dot product"""
        return self.x * other.x + self.y * other.y

    def __repr__(self):
        return f"({self.x}, {self.y})"


# Example vectors
a = Vector2(3, 4)
b = Vector2(2, 1)

dot_result = a.dot(b)
print(f"{a} · {b} = {dot_result}")  # 3*2 + 4*1 = 10

# Check front/back
player_forward = Vector2(1, 0)  # Facing right
direction_to_enemy = Vector2(0.5, 0.5)  # Enemy is upper-right

result = player_forward.dot(direction_to_enemy)
if result > 0:
    print("Enemy is in front!")
else:
    print("Enemy is behind!")
```

---

## The Geometric Formula (기하학적 공식)

### Concept (개념)
The dot product can also be calculated using magnitudes and the angle between vectors:

```
A · B = |A| * |B| * cos(θ)
```

Where θ (theta) is the angle between the two vectors.

This formula is incredibly useful because:
- If we know the vectors, we can find the angle
- cos(0°) = 1, so parallel vectors give maximum dot product
- cos(90°) = 0, so perpendicular vectors give 0
- cos(180°) = -1, so opposite vectors give negative product

### Game Example
For unit vectors (magnitude = 1):
```
A · B = cos(θ)
```

This directly gives us the cosine of the angle, which is very useful for angle-based checks!

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def dot(self, other):
        return self.x * other.x + self.y * other.y

    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2)

    def normalized(self):
        mag = self.magnitude()
        if mag == 0:
            return Vector2(0, 0)
        return Vector2(self.x / mag, self.y / mag)


# Verify both formulas give same result
a = Vector2(3, 4)
b = Vector2(4, 3)

# Component formula
dot1 = a.dot(b)
print(f"Component formula: {dot1}")  # 3*4 + 4*3 = 24

# Geometric formula (we need to find angle first)
mag_a = a.magnitude()  # 5
mag_b = b.magnitude()  # 5

# From dot = |A||B|cos(θ), we get cos(θ) = dot / (|A||B|)
cos_theta = dot1 / (mag_a * mag_b)
print(f"cos(θ) = {cos_theta}")  # 0.96

# Verify: |A| * |B| * cos(θ) should equal dot1
dot2 = mag_a * mag_b * cos_theta
print(f"Geometric formula: {dot2}")  # 24
```

---

## Finding Angles Between Vectors (벡터 사이 각도 구하기)

### Concept (개념)
Rearranging the geometric formula:

```
cos(θ) = (A · B) / (|A| * |B|)
θ = arccos((A · B) / (|A| * |B|))
```

For unit vectors:
```
θ = arccos(A · B)
```

### Game Example
Finding the angle between the player's facing direction and the direction to an enemy - useful for determining if the enemy is in the player's field of view.

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def dot(self, other):
        return self.x * other.x + self.y * other.y

    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2)

    def normalized(self):
        mag = self.magnitude()
        if mag == 0:
            return Vector2(0, 0)
        return Vector2(self.x / mag, self.y / mag)

    def angle_to(self, other):
        """Calculate angle between this vector and another (in degrees)"""
        # Normalize both vectors
        a = self.normalized()
        b = other.normalized()

        # Calculate dot product
        dot = a.dot(b)

        # Clamp to [-1, 1] to handle floating point errors
        dot = max(-1, min(1, dot))

        # Get angle in radians, then convert to degrees
        angle_rad = math.acos(dot)
        return math.degrees(angle_rad)


# Calculate angle between two directions
player_facing = Vector2(1, 0)  # Facing right
enemy_direction = Vector2(1, 1)  # Enemy is upper-right

angle = player_facing.angle_to(enemy_direction)
print(f"Angle to enemy: {angle:.1f}°")  # 45°

# Another example
direction1 = Vector2(1, 0)  # Right
direction2 = Vector2(0, 1)  # Up
angle = direction1.angle_to(direction2)
print(f"Angle between right and up: {angle:.1f}°")  # 90°
```

---

## Field of View Check (시야각 확인)

### Concept (개념)
To check if a target is within a character's field of view:

1. Get direction to target: `dir_to_target = (target - position).normalized()`
2. Calculate dot product with forward direction
3. Compare with cosine of half the FOV angle

If `forward · dir_to_target > cos(fov/2)`, target is visible.

### Game Example
Stealth games: Guards can only see within their cone of vision. Players outside this cone are safe.

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __sub__(self, other):
        return Vector2(self.x - other.x, self.y - other.y)

    def dot(self, other):
        return self.x * other.x + self.y * other.y

    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2)

    def normalized(self):
        mag = self.magnitude()
        if mag == 0:
            return Vector2(0, 0)
        return Vector2(self.x / mag, self.y / mag)


class Guard:
    def __init__(self, x, y, facing_x, facing_y):
        self.position = Vector2(x, y)
        self.forward = Vector2(facing_x, facing_y).normalized()
        self.fov = 90  # Field of view in degrees
        self.view_distance = 200

    def can_see(self, target_pos):
        """Check if target is within field of view"""
        # Get direction to target
        to_target = target_pos - self.position
        distance = to_target.magnitude()

        # Check distance first
        if distance > self.view_distance:
            return False

        # Normalize direction
        to_target = to_target.normalized()

        # Calculate dot product
        dot = self.forward.dot(to_target)

        # Compare with cosine of half FOV
        fov_threshold = math.cos(math.radians(self.fov / 2))

        return dot > fov_threshold


# Create guard facing right
guard = Guard(100, 100, 1, 0)
guard.fov = 90  # 90 degree cone

# Test different player positions
test_positions = [
    (Vector2(200, 100), "directly in front"),
    (Vector2(200, 150), "slightly above"),
    (Vector2(100, 200), "to the side"),
    (Vector2(0, 100), "behind"),
]

print(f"Guard at (100, 100) facing right with {guard.fov}° FOV:")
for pos, desc in test_positions:
    visible = guard.can_see(pos)
    status = "VISIBLE" if visible else "HIDDEN"
    print(f"  Player {desc} at ({pos.x}, {pos.y}): {status}")
```

---

## Basic Lighting with Dot Product (내적을 이용한 기초 조명)

### Concept (개념)
In 3D graphics, lighting is calculated using the dot product of:
- **Surface normal (표면 법선)**: Direction the surface is facing
- **Light direction (광원 방향)**: Direction toward the light source

**Lambert's Law**:
```
brightness = max(0, normal · light_direction)
```

When the surface faces the light directly: dot = 1, full brightness
When the surface is perpendicular to light: dot = 0, no direct light
When the surface faces away: dot < 0, clamped to 0

### Game Example
Simple 2D lighting for a top-down game where surfaces (walls) have different brightnesses based on their orientation relative to a light source.

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def dot(self, other):
        return self.x * other.x + self.y * other.y

    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2)

    def normalized(self):
        mag = self.magnitude()
        if mag == 0:
            return Vector2(0, 0)
        return Vector2(self.x / mag, self.y / mag)

    def __neg__(self):
        return Vector2(-self.x, -self.y)


class Surface:
    def __init__(self, name, normal_x, normal_y):
        self.name = name
        self.normal = Vector2(normal_x, normal_y).normalized()

    def calculate_brightness(self, light_direction):
        """Calculate surface brightness based on light direction"""
        # Light direction should point FROM surface TO light
        # Normal points outward from surface
        dot = self.normal.dot(light_direction)

        # Clamp to [0, 1] - surfaces facing away get no light
        brightness = max(0, dot)
        return brightness


# Light coming from upper-right (normalized)
light_dir = Vector2(1, 1).normalized()

# Different wall orientations
surfaces = [
    Surface("North Wall", 0, -1),   # Normal points down (south)
    Surface("South Wall", 0, 1),    # Normal points up (north)
    Surface("East Wall", 1, 0),     # Normal points right (east)
    Surface("West Wall", -1, 0),    # Normal points left (west)
]

print(f"Light direction: ({light_dir.x:.2f}, {light_dir.y:.2f})")
print("\nSurface brightness:")
for surface in surfaces:
    brightness = surface.calculate_brightness(light_dir)
    bar = "█" * int(brightness * 10)
    print(f"  {surface.name}: {brightness:.2f} {bar}")
```

---

## Projection Using Dot Product (내적을 이용한 투영)

### Concept (개념)
The dot product can project one vector onto another:

**Scalar projection** of A onto B:
```
scalar = A · B_normalized = |A| * cos(θ)
```

**Vector projection** of A onto B:
```
vector = (A · B_normalized) * B_normalized
```

### Game Example
Finding how much a character's velocity is in a specific direction (e.g., how fast they're moving forward vs sideways).

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def dot(self, other):
        return self.x * other.x + self.y * other.y

    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2)

    def normalized(self):
        mag = self.magnitude()
        if mag == 0:
            return Vector2(0, 0)
        return Vector2(self.x / mag, self.y / mag)

    def __mul__(self, scalar):
        return Vector2(self.x * scalar, self.y * scalar)

    def project_onto(self, other):
        """Project this vector onto another vector"""
        other_normalized = other.normalized()
        scalar = self.dot(other_normalized)
        return other_normalized * scalar

    def __repr__(self):
        return f"({self.x:.2f}, {self.y:.2f})"


# Car velocity
velocity = Vector2(10, 5)

# Car's forward direction
forward = Vector2(1, 0)

# How much velocity is in forward direction?
forward_velocity = velocity.project_onto(forward)
forward_speed = forward_velocity.magnitude()

print(f"Total velocity: {velocity}")
print(f"Forward component: {forward_velocity}")
print(f"Forward speed: {forward_speed}")

# Side direction
right = Vector2(0, 1)
side_velocity = velocity.project_onto(right)
side_speed = side_velocity.magnitude()

print(f"Side component: {side_velocity}")
print(f"Side speed (drift): {side_speed}")
```

---

## Summary (요약)

| Use Case | Korean | Formula/Method |
|----------|--------|----------------|
| Calculate Dot Product | 내적 계산 | A·B = Ax*Bx + Ay*By |
| Find Angle | 각도 구하기 | θ = arccos(A·B / (\|A\|\|B\|)) |
| Front/Back Check | 전방/후방 확인 | dot > 0 means in front |
| FOV Check | 시야각 확인 | dot > cos(fov/2) |
| Basic Lighting | 기초 조명 | brightness = max(0, N·L) |
| Projection | 투영 | (A·B_norm) * B_norm |

---

## Quick Reference: Dot Product Values

| Angle | cos(θ) | Meaning |
|-------|--------|---------|
| 0° | 1.0 | Same direction |
| 45° | 0.707 | Slightly different |
| 60° | 0.5 | Moderate angle |
| 90° | 0.0 | Perpendicular |
| 120° | -0.5 | Mostly opposite |
| 180° | -1.0 | Opposite direction |

---

## Practice Problems (연습 문제)

1. Calculate the dot product of (3, 4) and (4, -3). What does the result tell you about their relationship?
2. A guard faces direction (1, 0) with 120° FOV. Is a player at direction (0.6, 0.8) visible?
3. A surface has normal (0, 1). Light comes from direction (0.707, 0.707). What's the brightness?
4. Project velocity (8, 6) onto forward direction (1, 0). What's the forward speed?
