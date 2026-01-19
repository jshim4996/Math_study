# Chapter 10. Inverse Trigonometric Functions (역삼각함수)

> Get angles from vectors: making enemies face the player, aiming systems, and look-at rotations.

---

## What is atan2 (atan2란 무엇인가)

### Concept (개념)

In Chapter 09, we converted angles to direction vectors using `cos` and `sin`. Now we need the reverse: **given a direction vector, what is the angle?**

The `atan2(y, x)` function solves this:

```
angle = atan2(y, x)
```

**Why atan2 instead of atan?**
- `atan(y/x)` only gives angles in [-90°, 90°] (two quadrants)
- `atan2(y, x)` gives angles in [-180°, 180°] (all four quadrants)
- `atan2` handles x=0 case (division by zero)

**Important:** Note the parameter order is `(y, x)`, not `(x, y)`!

### Game Example
An enemy needs to know what angle to rotate to face the player. Given the direction vector from enemy to player, `atan2` gives the facing angle.

### Code Example
```python
import math

def angle_from_vector(x, y):
    """Convert direction vector to angle in degrees"""
    radians = math.atan2(y, x)
    degrees = math.degrees(radians)
    return degrees

# Test with known directions
test_vectors = [
    (1, 0),    # Right
    (0, 1),    # Up
    (-1, 0),   # Left
    (0, -1),   # Down
    (1, 1),    # Upper-right (45°)
    (-1, 1),   # Upper-left (135°)
    (-1, -1),  # Lower-left (-135° or 225°)
    (1, -1),   # Lower-right (-45° or 315°)
]

print("Direction vector to angle:")
for x, y in test_vectors:
    angle = angle_from_vector(x, y)
    print(f"  ({x:2}, {y:2}) → {angle:7.1f}°")

# Output:
#   ( 1,  0) →     0.0°
#   ( 0,  1) →    90.0°
#   (-1,  0) →   180.0° (or -180.0°)
#   ( 0, -1) →   -90.0°
#   ( 1,  1) →    45.0°
#   (-1,  1) →   135.0°
#   (-1, -1) →  -135.0°
#   ( 1, -1) →   -45.0°
```

---

## Getting Angle from Vector (벡터에서 각도 구하기)

### Concept (개념)

To find the angle **from point A to point B**:

1. Calculate direction vector: `direction = B - A`
2. Apply atan2: `angle = atan2(direction.y, direction.x)`

This gives the angle in standard mathematical convention (0° = right, counter-clockwise positive).

### Game Example
A turret at position A needs to aim at target at position B. The turret calculates the direction vector and converts it to a rotation angle.

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __sub__(self, other):
        return Vector2(self.x - other.x, self.y - other.y)

    def angle(self):
        """Get angle of this vector in degrees"""
        return math.degrees(math.atan2(self.y, self.x))

    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2)

    def __repr__(self):
        return f"({self.x:.1f}, {self.y:.1f})"


def angle_from_to(from_pos, to_pos):
    """Calculate angle from one position to another"""
    direction = to_pos - from_pos
    return direction.angle()


# Turret aiming at different targets
turret_pos = Vector2(100, 100)

targets = [
    Vector2(200, 100),  # Right
    Vector2(100, 200),  # Up
    Vector2(0, 100),    # Left
    Vector2(100, 0),    # Down
    Vector2(200, 200),  # Upper-right
]

print(f"Turret at {turret_pos}")
print("\nAiming at targets:")
for target in targets:
    angle = angle_from_to(turret_pos, target)
    direction = target - turret_pos
    print(f"  Target {target}: direction={direction}, angle={angle:.1f}°")
```

---

## Look-at Rotation (타겟을 바라보는 회전)

### Concept (개념)

"Look-at" rotation makes an object face toward a target. The steps:

1. Calculate direction to target
2. Get target angle using atan2
3. Set object's rotation to that angle

For **smooth rotation**, interpolate from current angle to target angle instead of snapping instantly.

### Game Example
- Enemy AI that tracks the player
- Camera following a moving target
- Homing missiles
- Character portraits facing the speaker

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __sub__(self, other):
        return Vector2(self.x - other.x, self.y - other.y)

    def __add__(self, other):
        return Vector2(self.x + other.x, self.y + other.y)

    def __mul__(self, scalar):
        return Vector2(self.x * scalar, self.y * scalar)

    def angle(self):
        return math.degrees(math.atan2(self.y, self.x))

    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2)

    def normalized(self):
        mag = self.magnitude()
        if mag == 0:
            return Vector2(0, 0)
        return Vector2(self.x / mag, self.y / mag)

    @staticmethod
    def from_angle(degrees, magnitude=1):
        rad = math.radians(degrees)
        return Vector2(math.cos(rad) * magnitude, math.sin(rad) * magnitude)

    def __repr__(self):
        return f"({self.x:.1f}, {self.y:.1f})"


class Enemy:
    def __init__(self, x, y):
        self.position = Vector2(x, y)
        self.rotation = 0  # Current facing angle

    def look_at(self, target_pos):
        """Instantly face the target"""
        direction = target_pos - self.position
        self.rotation = direction.angle()

    def look_at_smooth(self, target_pos, turn_speed, dt):
        """Smoothly rotate toward target"""
        direction = target_pos - self.position
        target_angle = direction.angle()

        # Calculate angle difference
        angle_diff = target_angle - self.rotation

        # Normalize to [-180, 180]
        while angle_diff > 180:
            angle_diff -= 360
        while angle_diff < -180:
            angle_diff += 360

        # Rotate toward target (limited by turn speed)
        max_turn = turn_speed * dt
        if abs(angle_diff) <= max_turn:
            self.rotation = target_angle
        elif angle_diff > 0:
            self.rotation += max_turn
        else:
            self.rotation -= max_turn


# Instant look-at
enemy = Enemy(0, 0)
player_pos = Vector2(100, 50)

print("Instant look-at:")
print(f"  Before: enemy rotation = {enemy.rotation}°")
enemy.look_at(player_pos)
print(f"  After:  enemy rotation = {enemy.rotation:.1f}°")


# Smooth rotation simulation
enemy2 = Enemy(0, 0)
enemy2.rotation = 0  # Facing right
target = Vector2(-100, 100)  # Target is upper-left (135°)

print("\nSmooth rotation (turn speed = 90°/sec):")
dt = 0.5  # Time step
for i in range(6):
    target_angle = (target - enemy2.position).angle()
    print(f"  t={i*dt:.1f}s: rotation={enemy2.rotation:6.1f}°, target={target_angle:.1f}°")
    enemy2.look_at_smooth(target, turn_speed=90, dt=dt)
```

---

## Enemy Rotating Toward Player (적이 플레이어를 향해 회전)

### Concept (개념)

Combining look-at with movement creates tracking enemies:

1. Calculate angle to player
2. Rotate toward that angle (instant or smooth)
3. Move forward in facing direction
4. Repeat each frame

**Variations:**
- **Homing missile**: Always faces target, fast turn speed
- **Tank**: Slow turn speed, must lead the target
- **Turret**: Only rotates, doesn't move

### Game Example
A homing missile that chases the player with smooth turning.

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __sub__(self, other):
        return Vector2(self.x - other.x, self.y - other.y)

    def __add__(self, other):
        return Vector2(self.x + other.x, self.y + other.y)

    def __mul__(self, scalar):
        return Vector2(self.x * scalar, self.y * scalar)

    def angle(self):
        return math.degrees(math.atan2(self.y, self.x))

    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2)

    @staticmethod
    def from_angle(degrees, magnitude=1):
        rad = math.radians(degrees)
        return Vector2(math.cos(rad) * magnitude, math.sin(rad) * magnitude)

    def __repr__(self):
        return f"({self.x:.1f}, {self.y:.1f})"


def normalize_angle(angle):
    """Keep angle in [-180, 180]"""
    while angle > 180:
        angle -= 360
    while angle < -180:
        angle += 360
    return angle


class HomingMissile:
    def __init__(self, x, y, initial_angle):
        self.position = Vector2(x, y)
        self.rotation = initial_angle
        self.speed = 150        # pixels per second
        self.turn_speed = 180   # degrees per second

    def update(self, target_pos, dt):
        """Update missile: rotate toward target, then move forward"""
        # 1. Calculate angle to target
        to_target = target_pos - self.position
        target_angle = to_target.angle()

        # 2. Rotate toward target
        angle_diff = normalize_angle(target_angle - self.rotation)
        max_turn = self.turn_speed * dt

        if abs(angle_diff) <= max_turn:
            self.rotation = target_angle
        elif angle_diff > 0:
            self.rotation += max_turn
        else:
            self.rotation -= max_turn

        # 3. Move forward in facing direction
        velocity = Vector2.from_angle(self.rotation, self.speed)
        self.position = self.position + velocity * dt

    def distance_to(self, target_pos):
        return (target_pos - self.position).magnitude()


# Simulation: missile chasing a moving player
missile = HomingMissile(0, 0, initial_angle=45)  # Starts facing upper-right
player = Vector2(200, 100)

print("Homing missile simulation:")
print(f"  Missile start: {missile.position}, facing {missile.rotation}°")
print(f"  Player at: {player}")
print()

dt = 0.1
for frame in range(20):
    time = frame * dt
    dist = missile.distance_to(player)

    if frame % 5 == 0:  # Print every 5th frame
        print(f"  t={time:.1f}s: pos={missile.position}, rot={missile.rotation:.1f}°, dist={dist:.1f}")

    if dist < 10:
        print(f"\n  HIT at t={time:.1f}s!")
        break

    missile.update(player, dt)


# Example with moving target
print("\n--- Chasing moving target ---")
missile2 = HomingMissile(0, 0, initial_angle=0)
player2 = Vector2(100, 0)
player_velocity = Vector2(0, 50)  # Player moving up

for frame in range(30):
    time = frame * dt

    # Move player
    player2 = player2 + player_velocity * dt

    dist = missile2.distance_to(player2)

    if frame % 5 == 0:
        print(f"  t={time:.1f}s: missile={missile2.position}, player={player2}, dist={dist:.1f}")

    if dist < 15:
        print(f"\n  HIT at t={time:.1f}s!")
        break

    missile2.update(player2, dt)
```

---

## Angle Wrapping and Delta (각도 래핑과 차이)

### Concept (개념)

Angles wrap around: 359° + 2° = 1° (not 361°). This creates problems when calculating angle differences.

**Problem:** Current angle = 350°, Target angle = 10°
- Naive: 10° - 350° = -340° (wrong! should be +20°)

**Solution:** Normalize the difference to [-180°, 180°]:

```python
def angle_difference(from_angle, to_angle):
    diff = to_angle - from_angle
    while diff > 180:
        diff -= 360
    while diff < -180:
        diff += 360
    return diff
```

### Code Example
```python
def normalize_angle(angle):
    """Normalize angle to [-180, 180]"""
    while angle > 180:
        angle -= 360
    while angle < -180:
        angle += 360
    return angle

def angle_difference(from_angle, to_angle):
    """Calculate shortest angular distance from one angle to another"""
    diff = to_angle - from_angle
    return normalize_angle(diff)

def lerp_angle(from_angle, to_angle, t):
    """Interpolate between angles, handling wrap-around"""
    diff = angle_difference(from_angle, to_angle)
    return normalize_angle(from_angle + diff * t)


# Test angle difference
test_cases = [
    (350, 10),   # Should be +20 (turn right)
    (10, 350),   # Should be -20 (turn left)
    (90, 270),   # Should be -180 or +180
    (0, 180),    # Should be +180 or -180
    (45, 90),    # Should be +45
]

print("Angle differences:")
for from_a, to_a in test_cases:
    diff = angle_difference(from_a, to_a)
    print(f"  {from_a}° → {to_a}°: diff = {diff:+.0f}°")


# Smooth angle interpolation
print("\nLerping from 350° to 10°:")
for t in [0, 0.25, 0.5, 0.75, 1.0]:
    result = lerp_angle(350, 10, t)
    print(f"  t={t}: {result:.1f}°")
```

---

## Summary (요약)

| Operation | Korean | Formula/Function |
|-----------|--------|------------------|
| Vector to Angle | 벡터→각도 | `atan2(y, x)` |
| Angle to A→B | A에서 B 방향 | `atan2(B.y - A.y, B.x - A.x)` |
| Look-at | 바라보기 | Set rotation = angle to target |
| Angle Difference | 각도 차이 | Normalize to [-180, 180] |
| Smooth Rotation | 부드러운 회전 | Clamp rotation by turn_speed * dt |

---

## Practice Problems (연습 문제)

1. An enemy at (50, 50) wants to face a player at (150, 100). What angle should the enemy rotate to?

2. A turret is currently facing 30°. The target is at angle 300°. What's the shortest rotation direction and amount?

3. Implement a "lead targeting" system: instead of aiming at where the target is, aim at where it will be based on its velocity.

4. Create a security camera that smoothly pans back and forth between two angles (e.g., -45° to 45°).
