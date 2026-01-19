# Chapter 03. Vector Basics (벡터 기초)

> Understanding vectors - the language of movement, direction, and force in games.

---

## What is a Vector? (벡터란?)

### Concept (개념)
A **vector (벡터)** is a mathematical object that has both **magnitude (크기)** and **direction (방향)**.

Think of it as an arrow:
- The length of the arrow represents magnitude (how much)
- The direction the arrow points represents direction (which way)

Vectors are fundamental in games because they perfectly describe things like:
- Movement (5 units to the right)
- Velocity (moving north at 10 m/s)
- Forces (pushing upward with 100 newtons)

### Game Example
When a character moves, we don't just care about "how fast" - we need to know "which direction." A vector captures both: "moving at 5 units per second toward the east."

### Code Example
```python
class Vector2:
    """A 2D vector with magnitude and direction"""
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Vector2({self.x}, {self.y})"

# This vector represents "3 units right, 4 units up"
movement = Vector2(3, 4)
# Magnitude (length) would be 5
# Direction points toward upper-right
```

---

## Scalar vs Vector (스칼라 vs 벡터)

### Concept (개념)
**Scalar (스칼라)**: A single number representing only magnitude.
- Examples: speed (50 km/h), temperature (25°C), health (100 HP)

**Vector (벡터)**: Multiple numbers representing magnitude AND direction.
- Examples: velocity (50 km/h north), force (10N upward), displacement (3m right, 4m forward)

| Scalar | Vector |
|--------|--------|
| Speed | Velocity |
| Distance | Displacement |
| Mass | Force |

### Game Example
- **Scalar**: The player's speed is 10 (just a number)
- **Vector**: The player's velocity is (10, 0) (moving right at speed 10)

A character with speed 10 could be moving in any direction. A character with velocity (10, 0) is specifically moving right.

### Code Example
```python
# Scalar - just magnitude
player_speed = 10  # How fast, but which direction?

# Vector - magnitude AND direction
player_velocity = Vector2(10, 0)  # Moving right at speed 10
player_velocity = Vector2(0, 10)  # Moving up at speed 10
player_velocity = Vector2(7.07, 7.07)  # Moving diagonally at ~speed 10
```

---

## Position Vectors (위치 벡터)

### Concept (개념)
A **position vector (위치 벡터)** represents a point's location relative to the origin (0, 0).

It's the vector from the origin to that point. While technically vectors don't have a fixed position (they can be moved anywhere), position vectors are "anchored" at the origin.

### Game Example
When we say a character is at position (100, 200), we're describing the position vector from origin (0, 0) to point (100, 200).

### Code Example
```python
import math

class Vector2:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def magnitude(self):
        """The length of this vector"""
        return math.sqrt(self.x**2 + self.y**2)

# Position vector - character's location
player_position = Vector2(100, 200)

# Distance from origin
dist_from_origin = player_position.magnitude()
print(f"Player is {dist_from_origin:.2f} units from origin")
```

---

## Vector Representation (벡터 표현)

### Concept (개념)
Vectors can be represented in several ways:

1. **Component form (성분 표현)**: (x, y) or (x, y, z)
   - Most common in programming

2. **Column notation (열벡터)**:
   ```
   [x]
   [y]
   ```

3. **Unit vector notation (단위벡터 표현)**: xi + yj
   - i is the unit vector in x direction (1, 0)
   - j is the unit vector in y direction (0, 1)

4. **Magnitude-angle form (크기-각도 표현)**: (r, θ)
   - r is length, θ is angle from positive x-axis

### Game Example
Different representations are useful for different purposes:
- Component form: Easy for calculations
- Magnitude-angle: Useful for rotation-based movement

### Code Example
```python
import math

class Vector2:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    @classmethod
    def from_angle(cls, angle_degrees, magnitude=1):
        """Create vector from angle and magnitude"""
        angle_rad = math.radians(angle_degrees)
        x = magnitude * math.cos(angle_rad)
        y = magnitude * math.sin(angle_rad)
        return cls(x, y)

    def to_angle(self):
        """Get angle in degrees"""
        return math.degrees(math.atan2(self.y, self.x))

    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2)

    def __repr__(self):
        return f"({self.x:.2f}, {self.y:.2f})"

# Component form
v1 = Vector2(3, 4)
print(f"Component: {v1}")
print(f"Magnitude: {v1.magnitude()}")
print(f"Angle: {v1.to_angle():.1f}°")

# From angle and magnitude
v2 = Vector2.from_angle(45, 10)  # 45 degrees, length 10
print(f"From angle 45°, magnitude 10: {v2}")
```

---

## Vectors in Games (게임에서의 벡터)

### Concept (개념)
Vectors are used everywhere in games:

1. **Position (위치)**: Where something is
2. **Velocity (속도)**: How fast and which direction something moves
3. **Acceleration (가속도)**: Change in velocity
4. **Direction (방향)**: Which way something faces
5. **Force (힘)**: Push or pull on objects
6. **Normal (법선)**: Perpendicular direction for surfaces

### Game Example

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

    def __repr__(self):
        return f"({self.x:.1f}, {self.y:.1f})"


class GameObject:
    def __init__(self, name):
        self.name = name
        self.position = Vector2(0, 0)      # Where am I?
        self.velocity = Vector2(0, 0)      # How am I moving?
        self.acceleration = Vector2(0, 0)  # How is my movement changing?

    def update(self, dt):
        """Update position based on velocity and acceleration"""
        # v = v + a * dt
        self.velocity = self.velocity + self.acceleration * dt
        # p = p + v * dt
        self.position = self.position + self.velocity * dt

    def __repr__(self):
        return f"{self.name} at {self.position}, velocity {self.velocity}"


# Create a ball with initial velocity
ball = GameObject("Ball")
ball.position = Vector2(100, 300)
ball.velocity = Vector2(50, 0)  # Moving right
ball.acceleration = Vector2(0, -98)  # Gravity pulling down

# Simulate 1 second in 0.1s steps
print("Ball trajectory:")
for i in range(11):
    print(f"  t={i*0.1:.1f}s: {ball}")
    ball.update(0.1)
```

---

## Common Vector Types in Game Engines

### Code Example
```python
import math

class Vector2:
    """2D Vector - for 2D games, UI, screen positions"""
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    # Common pre-defined vectors
    @classmethod
    def zero(cls):
        return cls(0, 0)

    @classmethod
    def one(cls):
        return cls(1, 1)

    @classmethod
    def up(cls):
        return cls(0, 1)

    @classmethod
    def down(cls):
        return cls(0, -1)

    @classmethod
    def left(cls):
        return cls(-1, 0)

    @classmethod
    def right(cls):
        return cls(1, 0)


class Vector3:
    """3D Vector - for 3D games, world positions"""
    def __init__(self, x=0, y=0, z=0):
        self.x = x
        self.y = y
        self.z = z

    @classmethod
    def zero(cls):
        return cls(0, 0, 0)

    @classmethod
    def one(cls):
        return cls(1, 1, 1)

    @classmethod
    def up(cls):
        return cls(0, 1, 0)

    @classmethod
    def forward(cls):
        return cls(0, 0, 1)

    @classmethod
    def right(cls):
        return cls(1, 0, 0)


# Using pre-defined vectors
player_pos = Vector2.zero()  # Start at origin
move_direction = Vector2.right()  # Move right

print(f"Player starts at {player_pos.x}, {player_pos.y}")
print(f"Moving in direction {move_direction.x}, {move_direction.y}")
```

---

## Visualizing Vectors

### Concept (개념)
It helps to visualize vectors as arrows:
- Start point (tail)
- End point (head)
- Length (magnitude)
- Direction (where it points)

### Code Example
```python
# ASCII visualization of vectors
def visualize_vector(v, scale=1, grid_size=10):
    """Simple ASCII visualization of a 2D vector"""
    grid = [['.' for _ in range(grid_size*2+1)] for _ in range(grid_size*2+1)]

    # Origin
    origin_x, origin_y = grid_size, grid_size
    grid[origin_y][origin_x] = 'O'

    # Vector endpoint
    end_x = int(origin_x + v.x * scale)
    end_y = int(origin_y - v.y * scale)  # Flip Y for display

    # Clamp to grid
    end_x = max(0, min(grid_size*2, end_x))
    end_y = max(0, min(grid_size*2, end_y))

    grid[end_y][end_x] = '*'

    # Draw line (simplified)
    print(f"Vector ({v.x}, {v.y}):")
    for row in grid:
        print(' '.join(row))

# Example
v = Vector2(5, 3)
visualize_vector(v)
```

---

## Summary (요약)

| Term | Korean | Description |
|------|--------|-------------|
| Vector | 벡터 | Has magnitude and direction |
| Scalar | 스칼라 | Only has magnitude |
| Magnitude | 크기 | Length of vector |
| Direction | 방향 | Where vector points |
| Position Vector | 위치 벡터 | Vector from origin to a point |
| Component | 성분 | x and y values of vector |
| Unit Vector | 단위 벡터 | Vector with length 1 |

---

## Practice Problems (연습 문제)

1. Is "the player has 100 health" a scalar or vector quantity?
2. A character is at position (50, 80). What is the magnitude of their position vector?
3. Create a vector from angle 60° with magnitude 5. What are its x and y components?
4. If velocity is (3, -4), which direction is the object moving (up-right, down-right, etc.)?
