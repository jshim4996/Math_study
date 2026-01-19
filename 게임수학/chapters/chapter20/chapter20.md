# Chapter 20. Motion (운동)

> The physics of movement: velocity, acceleration, and implementing gravity and jumps.

---

## Velocity (속도)

### Concept (개념)

**Velocity** is the rate of change of position over time:

```
velocity = change in position / change in time
position += velocity * dt
```

**In games:**
- Store velocity as a vector
- Each frame, add velocity × deltaTime to position
- Velocity has both magnitude (speed) and direction

**Units:**
- Pixels per second
- Meters per second (physics simulations)
- Units per frame (simpler but frame-rate dependent)

### Game Example
- Character movement
- Projectile flight
- Particle systems
- Any moving object

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

    def __repr__(self):
        return f"({self.x:.1f}, {self.y:.1f})"


class MovingObject:
    def __init__(self, x, y):
        self.position = Vector2(x, y)
        self.velocity = Vector2(0, 0)

    def update(self, dt):
        """Update position based on velocity"""
        self.position = self.position + self.velocity * dt


# Constant velocity movement
bullet = MovingObject(0, 100)
bullet.velocity = Vector2(500, 0)  # 500 pixels/second to the right

print("Bullet movement (constant velocity):")
dt = 0.016  # ~60 FPS
for frame in range(10):
    print(f"  Frame {frame}: position={bullet.position}")
    bullet.update(dt)


# Direction + speed approach
class DirectionalMover:
    def __init__(self, x, y):
        self.position = Vector2(x, y)
        self.direction = Vector2(1, 0)  # Unit vector
        self.speed = 0

    def set_direction_from_angle(self, degrees):
        rad = math.radians(degrees)
        self.direction = Vector2(math.cos(rad), math.sin(rad))

    def get_velocity(self):
        return self.direction * self.speed

    def update(self, dt):
        velocity = self.get_velocity()
        self.position = self.position + velocity * dt


# Moving at angle
mover = DirectionalMover(0, 0)
mover.set_direction_from_angle(45)  # 45 degrees
mover.speed = 100  # 100 pixels/second

print("\nMovement at 45° angle:")
for frame in range(5):
    print(f"  Frame {frame}: position={mover.position}")
    mover.update(0.1)
```

---

## Acceleration (가속도)

### Concept (개념)

**Acceleration** is the rate of change of velocity over time:

```
acceleration = change in velocity / change in time
velocity += acceleration * dt
position += velocity * dt
```

**Integration order matters:**
1. Apply acceleration to velocity
2. Apply velocity to position

This is called **Euler integration** (simple but has some inaccuracy).

### Game Example
- Car speeding up/slowing down
- Rocket thrust
- Gravity (constant acceleration)
- Friction (negative acceleration)

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

    def __repr__(self):
        return f"({self.x:.2f}, {self.y:.2f})"


class PhysicsObject:
    def __init__(self, x, y):
        self.position = Vector2(x, y)
        self.velocity = Vector2(0, 0)
        self.acceleration = Vector2(0, 0)

    def update(self, dt):
        """Euler integration"""
        # Update velocity with acceleration
        self.velocity = self.velocity + self.acceleration * dt
        # Update position with velocity
        self.position = self.position + self.velocity * dt


# Car accelerating
car = PhysicsObject(0, 0)
car.acceleration = Vector2(50, 0)  # 50 pixels/second² to the right

print("Car accelerating from rest:")
dt = 0.1
for frame in range(10):
    speed = car.velocity.magnitude()
    print(f"  t={frame*dt:.1f}s: pos={car.position.x:.1f}, vel={car.velocity.x:.1f}, speed={speed:.1f}")
    car.update(dt)


# Car braking (negative acceleration)
print("\nCar braking:")
car2 = PhysicsObject(0, 0)
car2.velocity = Vector2(100, 0)  # Starting at 100 px/s
car2.acceleration = Vector2(-30, 0)  # Braking

for frame in range(10):
    # Prevent moving backward
    if car2.velocity.x < 0:
        car2.velocity.x = 0
        car2.acceleration.x = 0

    print(f"  t={frame*dt:.1f}s: pos={car2.position.x:.1f}, vel={car2.velocity.x:.1f}")
    car2.update(dt)
```

---

## Uniform Motion, Uniformly Accelerated Motion (등속, 등가속 운동)

### Concept (개념)

**Uniform motion** (등속 운동):
- Constant velocity
- Zero acceleration
- Position changes linearly: x = x₀ + v×t

**Uniformly accelerated motion** (등가속 운동):
- Constant acceleration
- Velocity changes linearly: v = v₀ + a×t
- Position changes quadratically: x = x₀ + v₀×t + ½×a×t²

**Kinematic equations:**
```
v = v₀ + a×t
x = x₀ + v₀×t + ½×a×t²
v² = v₀² + 2×a×(x - x₀)
```

### Game Example
- Falling objects (constant gravity)
- Projectile motion
- Physics puzzles

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

    def __repr__(self):
        return f"({self.x:.2f}, {self.y:.2f})"


# Kinematic equations (analytical solution)
def position_at_time(x0, v0, a, t):
    """Calculate position using kinematic equation"""
    return x0 + v0 * t + 0.5 * a * t * t


def velocity_at_time(v0, a, t):
    """Calculate velocity using kinematic equation"""
    return v0 + a * t


# Compare numerical (Euler) vs analytical solution
print("Falling object - Numerical vs Analytical:")
print("(Initial height: 100, Initial velocity: 0, Gravity: -100)")

# Numerical simulation
class FallingObject:
    def __init__(self):
        self.y = 100
        self.vy = 0
        self.gravity = -100

    def update(self, dt):
        self.vy += self.gravity * dt
        self.y += self.vy * dt


obj = FallingObject()
dt = 0.1

print("\n  Time    Numerical    Analytical    Error")
for frame in range(11):
    t = frame * dt

    # Analytical
    y_analytical = position_at_time(100, 0, -100, t)
    v_analytical = velocity_at_time(0, -100, t)

    error = abs(obj.y - y_analytical)

    print(f"  {t:.1f}s    y={obj.y:7.2f}    y={y_analytical:7.2f}    {error:.3f}")

    obj.update(dt)


# Projectile motion (2D uniformly accelerated)
print("\nProjectile motion (launched at 45°, speed 100):")

class Projectile:
    def __init__(self, x, y, speed, angle_deg):
        self.position = Vector2(x, y)
        angle_rad = math.radians(angle_deg)
        self.velocity = Vector2(
            speed * math.cos(angle_rad),
            speed * math.sin(angle_rad)
        )
        self.gravity = Vector2(0, -200)

    def update(self, dt):
        self.velocity = self.velocity + self.gravity * dt
        self.position = self.position + self.velocity * dt


proj = Projectile(0, 0, 100, 45)

for frame in range(15):
    t = frame * 0.05
    print(f"  t={t:.2f}s: pos={proj.position}")
    if proj.position.y < 0:
        print("  (Hit ground)")
        break
    proj.update(0.05)
```

---

## Implementing Gravity (중력 구현)

### Concept (개념)

**Gravity** is a constant downward acceleration:

```python
gravity = Vector2(0, -980)  # ~9.8 m/s² in pixels
velocity += gravity * dt
position += velocity * dt
```

**Typical values:**
- Earth gravity: ~9.8 m/s² (or 980 pixels/s² for pixel games)
- Platformer games often use higher values for snappy feel

**Ground collision:**
When object hits ground, stop downward velocity.

### Game Example
- Platformer characters
- Falling objects
- Particle effects
- Ragdoll physics

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

    def __repr__(self):
        return f"({self.x:.1f}, {self.y:.1f})"


class GravityObject:
    def __init__(self, x, y):
        self.position = Vector2(x, y)
        self.velocity = Vector2(0, 0)
        self.gravity = -500  # pixels/second²
        self.ground_y = 0

    def update(self, dt):
        # Apply gravity to velocity
        self.velocity.y += self.gravity * dt

        # Update position
        self.position = self.position + self.velocity * dt

        # Ground collision
        if self.position.y <= self.ground_y:
            self.position.y = self.ground_y
            self.velocity.y = 0

    def is_on_ground(self):
        return self.position.y <= self.ground_y


# Dropping object
print("Dropping object from height 200:")
obj = GravityObject(0, 200)

dt = 0.033  # ~30 FPS
for frame in range(30):
    print(f"  Frame {frame:2d}: y={obj.position.y:6.1f}, vy={obj.velocity.y:7.1f}")
    obj.update(dt)
    if obj.is_on_ground():
        print("  (Hit ground)")
        break


# Multiple objects with different masses fall at same rate
print("\nMultiple objects falling (same gravity, different 'masses'):")
print("(In games without air resistance, mass doesn't affect fall speed)")

objects = [
    ("Heavy", GravityObject(0, 100)),
    ("Light", GravityObject(50, 100)),
    ("Tiny", GravityObject(100, 100)),
]

for frame in range(10):
    heights = [f"{name}: y={obj.position.y:.1f}" for name, obj in objects]
    print(f"  Frame {frame}: {', '.join(heights)}")
    for _, obj in objects:
        obj.update(0.05)
```

---

## Implementing Jump (점프 구현)

### Concept (개념)

**Jump** gives an instant upward velocity:

```python
if can_jump:
    velocity.y = jump_velocity
```

**Jump height calculation:**
Using kinematic equation: v² = v₀² + 2a×h

At peak: v = 0
So: 0 = v₀² + 2×(-gravity)×h
Thus: v₀ = sqrt(2 × gravity × desired_height)

**Common jump mechanics:**
- Variable jump height (hold longer = higher)
- Double jump
- Coyote time (jump shortly after leaving platform)
- Jump buffering

### Game Example
- Platformer characters
- Action game combat
- Sports games

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

    def __repr__(self):
        return f"({self.x:.1f}, {self.y:.1f})"


class PlatformerCharacter:
    def __init__(self, x, y):
        self.position = Vector2(x, y)
        self.velocity = Vector2(0, 0)

        # Physics parameters
        self.gravity = -1000  # pixels/second²
        self.ground_y = 0

        # Jump parameters
        self.jump_height = 100  # desired jump height in pixels
        self.jump_velocity = math.sqrt(2 * abs(self.gravity) * self.jump_height)

        # State
        self.is_grounded = True

    def update(self, dt):
        # Apply gravity
        self.velocity.y += self.gravity * dt

        # Update position
        self.position = self.position + self.velocity * dt

        # Ground collision
        if self.position.y <= self.ground_y:
            self.position.y = self.ground_y
            self.velocity.y = 0
            self.is_grounded = True
        else:
            self.is_grounded = False

    def jump(self):
        """Initiate a jump"""
        if self.is_grounded:
            self.velocity.y = self.jump_velocity
            self.is_grounded = False
            return True
        return False


# Basic jump
print(f"Basic Jump (target height: 100 pixels)")
player = PlatformerCharacter(0, 0)
print(f"  Jump velocity: {player.jump_velocity:.1f} pixels/second")

player.jump()
dt = 0.016

max_height = 0
for frame in range(60):
    max_height = max(max_height, player.position.y)
    if frame % 5 == 0:
        state = "airborne" if not player.is_grounded else "grounded"
        print(f"  Frame {frame:2d}: y={player.position.y:6.1f}, vy={player.velocity.y:7.1f}, {state}")

    player.update(dt)

    if player.is_grounded and frame > 0:
        print(f"  Landed! Max height reached: {max_height:.1f}")
        break


# Variable height jump (hold to jump higher)
class VariableJumpCharacter:
    def __init__(self, x, y):
        self.position = Vector2(x, y)
        self.velocity = Vector2(0, 0)
        self.gravity = -1000
        self.ground_y = 0

        # Jump parameters
        self.min_jump_velocity = 200
        self.max_jump_velocity = 450
        self.jump_cut_multiplier = 0.5  # Reduce velocity when releasing button

        self.is_grounded = True
        self.is_jumping = False

    def update(self, dt):
        self.velocity.y += self.gravity * dt
        self.position = self.position + self.velocity * dt

        if self.position.y <= self.ground_y:
            self.position.y = self.ground_y
            self.velocity.y = 0
            self.is_grounded = True
            self.is_jumping = False
        else:
            self.is_grounded = False

    def start_jump(self):
        """Called when jump button pressed"""
        if self.is_grounded:
            self.velocity.y = self.max_jump_velocity
            self.is_jumping = True
            self.is_grounded = False

    def release_jump(self):
        """Called when jump button released"""
        if self.is_jumping and self.velocity.y > self.min_jump_velocity:
            # Cut the jump short
            self.velocity.y *= self.jump_cut_multiplier


print("\nVariable height jump:")
player2 = VariableJumpCharacter(0, 0)

# Short jump (release quickly)
player2.start_jump()
player2.update(0.05)  # One frame
player2.release_jump()

max_height = 0
while not player2.is_grounded:
    max_height = max(max_height, player2.position.y)
    player2.update(0.016)
print(f"  Short jump (quick release): max height = {max_height:.1f}")

# Full jump (hold button)
player2 = VariableJumpCharacter(0, 0)
player2.start_jump()

max_height = 0
while not player2.is_grounded or max_height == 0:
    max_height = max(max_height, player2.position.y)
    player2.update(0.016)
print(f"  Full jump (hold button): max height = {max_height:.1f}")
```

---

## Summary (요약)

| Concept | Korean | Formula |
|---------|--------|---------|
| Velocity | 속도 | position += velocity × dt |
| Acceleration | 가속도 | velocity += acceleration × dt |
| Gravity | 중력 | constant downward acceleration |
| Jump | 점프 | instant upward velocity |
| Jump velocity | 점프 속도 | v = √(2 × g × h) |

---

## Practice Problems (연습 문제)

1. An object at x=0 moves with velocity 50 pixels/second. Where is it after 3 seconds?

2. A car starts at rest and accelerates at 20 pixels/s². What's its velocity after 5 seconds?

3. Calculate the jump velocity needed for a 150 pixel jump with gravity of -800 pixels/s².

4. Implement double jump: allow a second jump while in the air.
