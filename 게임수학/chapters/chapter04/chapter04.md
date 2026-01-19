# Chapter 04. Vector Operations (벡터 연산)

> Master the essential vector operations that power movement, combat, and physics in games.

---

## Vector Addition (벡터 덧셈)

### Concept (개념)
**Vector addition (벡터 덧셈)** combines two vectors by adding their components:

```
A + B = (Ax + Bx, Ay + By)
```

Geometrically, place the tail of vector B at the head of vector A. The result goes from A's tail to B's head.

### Game Example
Adding velocity to position creates movement:
- Position: (100, 200)
- Velocity: (5, -3) per frame
- New Position: (100 + 5, 200 + (-3)) = (105, 197)

Combining forces:
- Wind force: (2, 0)
- Gravity: (0, -10)
- Total force: (2, -10)

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __add__(self, other):
        """Vector addition: self + other"""
        return Vector2(self.x + other.x, self.y + other.y)

    def __repr__(self):
        return f"({self.x}, {self.y})"

# Movement example
position = Vector2(100, 200)
velocity = Vector2(5, -3)

# Update position
new_position = position + velocity
print(f"Old: {position} + Velocity: {velocity} = New: {new_position}")

# Combining forces
gravity = Vector2(0, -10)
wind = Vector2(2, 0)
jump = Vector2(0, 15)

total_force = gravity + wind + jump
print(f"Total force: {total_force}")  # (2, 5)
```

---

## Vector Subtraction (벡터 뺄셈)

### Concept (개념)
**Vector subtraction (벡터 뺄셈)** finds the difference between vectors:

```
A - B = (Ax - Bx, Ay - By)
```

The result of B - A gives the vector FROM point A TO point B.

### Game Example
Finding direction from player to enemy:
- Player at: (100, 100)
- Enemy at: (150, 200)
- Direction to enemy: (150 - 100, 200 - 100) = (50, 100)

This is crucial for:
- AI targeting
- Projectile aiming
- Camera follow systems

### Code Example
```python
class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __sub__(self, other):
        """Vector subtraction: self - other"""
        return Vector2(self.x - other.x, self.y - other.y)

    def __repr__(self):
        return f"({self.x}, {self.y})"

# Finding direction from player to enemy
player_pos = Vector2(100, 100)
enemy_pos = Vector2(150, 200)

# Direction vector FROM player TO enemy
direction_to_enemy = enemy_pos - player_pos
print(f"Direction to enemy: {direction_to_enemy}")  # (50, 100)

# Direction vector FROM enemy TO player (opposite)
direction_to_player = player_pos - enemy_pos
print(f"Direction to player: {direction_to_player}")  # (-50, -100)
```

---

## Scalar Multiplication (스칼라 곱셈)

### Concept (개념)
**Scalar multiplication (스칼라 곱셈)** multiplies each component by a number:

```
k * A = (k * Ax, k * Ay)
```

- If k > 1: Vector gets longer (scales up)
- If 0 < k < 1: Vector gets shorter (scales down)
- If k < 0: Vector reverses direction

### Game Example
Adjusting speed:
- Base velocity: (3, 4) with magnitude 5
- Speed boost (2x): (6, 8) with magnitude 10
- Slow effect (0.5x): (1.5, 2) with magnitude 2.5
- Reverse: (-3, -4) with magnitude 5, opposite direction

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __mul__(self, scalar):
        """Scalar multiplication: self * scalar"""
        return Vector2(self.x * scalar, self.y * scalar)

    def __rmul__(self, scalar):
        """Allow scalar * vector"""
        return self.__mul__(scalar)

    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2)

    def __repr__(self):
        return f"({self.x}, {self.y})"

# Speed modifications
base_velocity = Vector2(3, 4)
print(f"Base velocity: {base_velocity}, speed: {base_velocity.magnitude()}")

# Speed boost
boosted = base_velocity * 2
print(f"Boosted (2x): {boosted}, speed: {boosted.magnitude()}")

# Slow debuff
slowed = base_velocity * 0.5
print(f"Slowed (0.5x): {slowed}, speed: {slowed.magnitude()}")

# Reverse (knockback)
reversed_vel = base_velocity * -1
print(f"Reversed: {reversed_vel}")

# Using delta time for frame-rate independence
velocity = Vector2(100, 50)  # units per second
dt = 0.016  # 60 FPS = ~0.016 seconds per frame
frame_movement = velocity * dt
print(f"Movement this frame: {frame_movement}")
```

---

## Vector Magnitude (벡터 크기)

### Concept (개념)
**Magnitude (크기)**, also called length or norm, is the "size" of a vector:

```
|A| = √(Ax² + Ay²)
```

For 3D:
```
|A| = √(Ax² + Ay² + Az²)
```

### Game Example
- Finding actual speed from velocity vector
- Checking if a direction vector is unit length
- Determining the strength of a force

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def magnitude(self):
        """Calculate the length of this vector"""
        return math.sqrt(self.x**2 + self.y**2)

    def magnitude_squared(self):
        """Squared magnitude (faster, for comparisons)"""
        return self.x**2 + self.y**2

    def __repr__(self):
        return f"({self.x}, {self.y})"

# Calculate speed from velocity
velocity = Vector2(3, 4)
speed = velocity.magnitude()
print(f"Velocity {velocity} has speed {speed}")  # 5.0

# Compare which enemy is closer (using squared magnitude for efficiency)
player = Vector2(0, 0)
enemy1_offset = Vector2(3, 4)   # Distance 5
enemy2_offset = Vector2(6, 2)   # Distance ~6.32

if enemy1_offset.magnitude_squared() < enemy2_offset.magnitude_squared():
    print("Enemy 1 is closer")
else:
    print("Enemy 2 is closer")
```

---

## Vector Normalization (벡터 정규화)

### Concept (개념)
**Normalization (정규화)** converts a vector to a **unit vector (단위 벡터)** - a vector with magnitude 1 that keeps the same direction.

```
normalized = A / |A| = (Ax/|A|, Ay/|A|)
```

Why normalize?
- Separate direction from magnitude
- Consistent movement speed regardless of input direction
- Required for many calculations (dot product angles, etc.)

### Game Example
When using keyboard input (WASD):
- Pressing just D: direction = (1, 0), magnitude = 1
- Pressing W+D: direction = (1, 1), magnitude = √2 ≈ 1.414

Without normalization, diagonal movement is ~41% faster! Normalize to fix this.

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2)

    def normalized(self):
        """Return unit vector in same direction"""
        mag = self.magnitude()
        if mag == 0:
            return Vector2(0, 0)  # Can't normalize zero vector
        return Vector2(self.x / mag, self.y / mag)

    def __mul__(self, scalar):
        return Vector2(self.x * scalar, self.y * scalar)

    def __repr__(self):
        return f"({self.x:.3f}, {self.y:.3f})"


# Problem: Diagonal movement is faster
input_right = Vector2(1, 0)
input_diagonal = Vector2(1, 1)

print(f"Right input magnitude: {input_right.magnitude()}")      # 1.0
print(f"Diagonal input magnitude: {input_diagonal.magnitude()}")  # 1.414...

# Solution: Normalize then multiply by speed
MOVE_SPEED = 5

# Right movement (already unit length)
move_right = input_right.normalized() * MOVE_SPEED
print(f"Right movement: {move_right}, speed: {move_right.magnitude()}")

# Diagonal movement (normalized to unit length)
move_diagonal = input_diagonal.normalized() * MOVE_SPEED
print(f"Diagonal movement: {move_diagonal}, speed: {move_diagonal.magnitude()}")

# Both have speed 5 now!
```

---

## Complete Vector2 Class

### Code Example
```python
import math

class Vector2:
    """Complete 2D Vector class with all basic operations"""

    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    # Addition
    def __add__(self, other):
        return Vector2(self.x + other.x, self.y + other.y)

    # Subtraction
    def __sub__(self, other):
        return Vector2(self.x - other.x, self.y - other.y)

    # Scalar multiplication
    def __mul__(self, scalar):
        return Vector2(self.x * scalar, self.y * scalar)

    def __rmul__(self, scalar):
        return self.__mul__(scalar)

    # Scalar division
    def __truediv__(self, scalar):
        return Vector2(self.x / scalar, self.y / scalar)

    # Negation
    def __neg__(self):
        return Vector2(-self.x, -self.y)

    # Equality
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

    # Magnitude
    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2)

    def magnitude_squared(self):
        return self.x**2 + self.y**2

    # Normalization
    def normalized(self):
        mag = self.magnitude()
        if mag == 0:
            return Vector2(0, 0)
        return Vector2(self.x / mag, self.y / mag)

    # Distance to another vector
    def distance_to(self, other):
        return (other - self).magnitude()

    # String representation
    def __repr__(self):
        return f"Vector2({self.x:.2f}, {self.y:.2f})"

    # Common vectors
    @staticmethod
    def zero():
        return Vector2(0, 0)

    @staticmethod
    def one():
        return Vector2(1, 1)

    @staticmethod
    def up():
        return Vector2(0, 1)

    @staticmethod
    def down():
        return Vector2(0, -1)

    @staticmethod
    def left():
        return Vector2(-1, 0)

    @staticmethod
    def right():
        return Vector2(1, 0)


# Example usage
player_pos = Vector2(100, 100)
enemy_pos = Vector2(200, 150)

# Direction from player to enemy
direction = (enemy_pos - player_pos).normalized()
print(f"Direction to enemy: {direction}")

# Move player toward enemy
SPEED = 5
player_pos = player_pos + direction * SPEED
print(f"New player position: {player_pos}")
```

---

## Practical Game Example: Movement System

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __add__(self, other):
        return Vector2(self.x + other.x, self.y + other.y)

    def __mul__(self, scalar):
        return Vector2(self.x * scalar, self.y * scalar)

    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2)

    def normalized(self):
        mag = self.magnitude()
        if mag == 0:
            return Vector2(0, 0)
        return Vector2(self.x / mag, self.y / mag)


class Player:
    def __init__(self):
        self.position = Vector2(400, 300)
        self.speed = 200  # pixels per second
        self.velocity = Vector2.zero()

    def handle_input(self, keys):
        """Process keyboard input"""
        input_dir = Vector2(0, 0)

        if keys.get('w'):
            input_dir.y += 1
        if keys.get('s'):
            input_dir.y -= 1
        if keys.get('a'):
            input_dir.x -= 1
        if keys.get('d'):
            input_dir.x += 1

        # Normalize to prevent fast diagonal movement
        if input_dir.magnitude() > 0:
            input_dir = input_dir.normalized()

        # Set velocity based on input direction and speed
        self.velocity = input_dir * self.speed

    def update(self, dt):
        """Update position based on velocity"""
        self.position = self.position + self.velocity * dt

    def __repr__(self):
        return f"Player at ({self.position.x:.1f}, {self.position.y:.1f})"


# Simulate game loop
player = Player()

# Frame 1: Moving right
keys = {'d': True}
player.handle_input(keys)
player.update(0.016)  # ~60 FPS
print(f"After moving right: {player}")

# Frame 2: Moving diagonally (up-right)
keys = {'w': True, 'd': True}
player.handle_input(keys)
player.update(0.016)
print(f"After moving diagonal: {player}")

# Note: Both movements travel same distance per frame!
```

---

## Summary (요약)

| Operation | Korean | Formula | Use Case |
|-----------|--------|---------|----------|
| Addition | 덧셈 | A + B = (Ax+Bx, Ay+By) | Movement, combining forces |
| Subtraction | 뺄셈 | A - B = (Ax-Bx, Ay-By) | Finding direction |
| Scalar Multiply | 스칼라 곱 | k*A = (k*Ax, k*Ay) | Speed adjustment |
| Magnitude | 크기 | \|A\| = √(Ax²+Ay²) | Finding speed/length |
| Normalize | 정규화 | A/\|A\| | Unit direction vector |

---

## Practice Problems (연습 문제)

1. Add vectors (3, 5) and (-2, 8).
2. Find the direction vector from point (10, 20) to point (40, 60).
3. A character has velocity (8, 6). What is their speed?
4. Normalize the vector (3, 4). Verify the result has magnitude 1.
5. If input is diagonal (1, 1) and speed is 10, what should the velocity vector be?
