# Chapter 22. Collision Response (충돌 반응)

> What happens after collision: reflection, bouncing, and elastic collision physics.

---

## Reflection After Collision (충돌 후 반사)

### Concept (개념)

When an object collides with a surface, its velocity **reflects** off the surface.

The reflection depends on:
- Incoming velocity
- Surface normal (perpendicular to surface)
- Material properties (bounciness)

**Basic reflection:**
```
reflected = incoming - 2 × (incoming · normal) × normal
```

### Game Example
- Pong paddle
- Breakout bricks
- Billiards
- Light rays in graphics

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

    def dot(self, other):
        return self.x * other.x + self.y * other.y

    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2)

    def normalized(self):
        mag = self.magnitude()
        if mag == 0:
            return Vector2(0, 0)
        return Vector2(self.x / mag, self.y / mag)

    def __repr__(self):
        return f"({self.x:.2f}, {self.y:.2f})"


def reflect(velocity, normal):
    """Reflect velocity off a surface with given normal"""
    # r = v - 2(v·n)n
    dot = velocity.dot(normal)
    return velocity - normal * (2 * dot)


# Ball bouncing off walls
ball_velocity = Vector2(5, -3)  # Moving right and down

# Floor (normal pointing up)
floor_normal = Vector2(0, 1)
reflected_floor = reflect(ball_velocity, floor_normal)

print("Ball reflection:")
print(f"  Incoming velocity: {ball_velocity}")
print(f"  Floor normal: {floor_normal}")
print(f"  Reflected velocity: {reflected_floor}")

# Wall (normal pointing left)
wall_normal = Vector2(-1, 0)
reflected_wall = reflect(ball_velocity, wall_normal)

print(f"\n  Wall normal: {wall_normal}")
print(f"  Reflected velocity: {reflected_wall}")

# Angled surface (45 degrees)
angled_normal = Vector2(-0.707, 0.707)  # Normalized
reflected_angled = reflect(ball_velocity, angled_normal)

print(f"\n  Angled surface (45°) normal: {angled_normal}")
print(f"  Reflected velocity: {reflected_angled}")
```

---

## Reflection Vector Calculation (반사 벡터 계산)

### Concept (개념)

**Formula derivation:**

1. Decompose velocity into parallel and perpendicular components relative to surface
2. Keep parallel component, reverse perpendicular component
3. Combine back

```
v_parallel = v - (v · n) × n
v_perpendicular = (v · n) × n
reflected = v_parallel - v_perpendicular
         = v - 2 × (v · n) × n
```

**With energy loss (bounciness):**
```
reflected = v - (1 + bounciness) × (v · n) × n
```
- bounciness = 1: Perfect reflection (no energy loss)
- bounciness = 0: No bounce (velocity parallel to surface only)
- bounciness = 0.5: 50% energy retained in bounce

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

    def dot(self, other):
        return self.x * other.x + self.y * other.y

    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2)

    def __repr__(self):
        return f"({self.x:.2f}, {self.y:.2f})"


def reflect_with_bounciness(velocity, normal, bounciness=1.0):
    """
    Reflect velocity with energy loss.
    bounciness: 0 = no bounce, 1 = perfect bounce
    """
    dot = velocity.dot(normal)

    # Only reflect if moving toward surface
    if dot >= 0:
        return velocity  # Moving away, no reflection

    return velocity - normal * ((1 + bounciness) * dot)


# Compare different bounciness values
velocity = Vector2(3, -4)  # Moving right and down
normal = Vector2(0, 1)     # Floor

print("Reflection with different bounciness:")
print(f"  Incoming: {velocity}, speed = {velocity.magnitude():.2f}")

for bounce in [1.0, 0.75, 0.5, 0.25, 0.0]:
    result = reflect_with_bounciness(velocity, normal, bounce)
    print(f"  Bounciness {bounce}: {result}, speed = {result.magnitude():.2f}")


# Practical: Ball in a box
class Ball:
    def __init__(self, x, y, radius):
        self.position = Vector2(x, y)
        self.velocity = Vector2(0, 0)
        self.radius = radius
        self.bounciness = 0.8

    def update(self, dt, box_width, box_height):
        # Update position
        self.position = self.position + self.velocity * dt

        # Check wall collisions
        # Left wall
        if self.position.x - self.radius < 0:
            self.position.x = self.radius
            self.velocity = reflect_with_bounciness(
                self.velocity, Vector2(1, 0), self.bounciness
            )

        # Right wall
        if self.position.x + self.radius > box_width:
            self.position.x = box_width - self.radius
            self.velocity = reflect_with_bounciness(
                self.velocity, Vector2(-1, 0), self.bounciness
            )

        # Bottom wall
        if self.position.y - self.radius < 0:
            self.position.y = self.radius
            self.velocity = reflect_with_bounciness(
                self.velocity, Vector2(0, 1), self.bounciness
            )

        # Top wall
        if self.position.y + self.radius > box_height:
            self.position.y = box_height - self.radius
            self.velocity = reflect_with_bounciness(
                self.velocity, Vector2(0, -1), self.bounciness
            )


# Simulate ball bouncing in box
ball = Ball(50, 80, radius=5)
ball.velocity = Vector2(100, -150)

print("\nBall bouncing in 100x100 box:")
dt = 0.05
for frame in range(20):
    print(f"  t={frame*dt:.2f}s: pos={ball.position}, vel=({ball.velocity.x:.0f}, {ball.velocity.y:.0f})")
    ball.update(dt, 100, 100)
```

---

## Elastic Collision Concept (탄성 충돌 개념)

### Concept (개념)

**Elastic collision** between two objects conserves both momentum and kinetic energy.

**Conservation laws:**
- **Momentum:** m₁v₁ + m₂v₂ = m₁v₁' + m₂v₂'
- **Kinetic Energy:** ½m₁v₁² + ½m₂v₂² = ½m₁v₁'² + ½m₂v₂'²

**1D elastic collision formula:**
```
v₁' = ((m₁-m₂)v₁ + 2m₂v₂) / (m₁+m₂)
v₂' = ((m₂-m₁)v₂ + 2m₁v₁) / (m₁+m₂)
```

**Special cases:**
- Equal masses: Objects exchange velocities
- One object stationary with equal mass: First stops, second moves
- Heavy hits light: Light object moves fast, heavy barely changes

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

    def dot(self, other):
        return self.x * other.x + self.y * other.y

    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2)

    def normalized(self):
        mag = self.magnitude()
        if mag == 0:
            return Vector2(0, 0)
        return Vector2(self.x / mag, self.y / mag)

    def __repr__(self):
        return f"({self.x:.2f}, {self.y:.2f})"


def elastic_collision_1d(v1, v2, m1, m2):
    """1D elastic collision - returns new velocities"""
    v1_new = ((m1 - m2) * v1 + 2 * m2 * v2) / (m1 + m2)
    v2_new = ((m2 - m1) * v2 + 2 * m1 * v1) / (m1 + m2)
    return v1_new, v2_new


# 1D Examples
print("1D Elastic Collision Examples:")

# Equal masses, one moving
v1, v2, m1, m2 = 10, 0, 1, 1
new_v1, new_v2 = elastic_collision_1d(v1, v2, m1, m2)
print(f"\n  Equal mass (m=1), ball1 moving at 10:")
print(f"    Before: v1={v1}, v2={v2}")
print(f"    After:  v1={new_v1:.1f}, v2={new_v2:.1f}")

# Heavy hits light
v1, v2, m1, m2 = 10, 0, 10, 1
new_v1, new_v2 = elastic_collision_1d(v1, v2, m1, m2)
print(f"\n  Heavy (m=10) hits light (m=1):")
print(f"    Before: v1={v1}, v2={v2}")
print(f"    After:  v1={new_v1:.1f}, v2={new_v2:.1f}")

# Light hits heavy
v1, v2, m1, m2 = 10, 0, 1, 10
new_v1, new_v2 = elastic_collision_1d(v1, v2, m1, m2)
print(f"\n  Light (m=1) hits heavy (m=10):")
print(f"    Before: v1={v1}, v2={v2}")
print(f"    After:  v1={new_v1:.1f}, v2={new_v2:.1f}")

# Both moving
v1, v2, m1, m2 = 10, -5, 1, 1
new_v1, new_v2 = elastic_collision_1d(v1, v2, m1, m2)
print(f"\n  Equal mass, both moving (v1=10, v2=-5):")
print(f"    Before: v1={v1}, v2={v2}")
print(f"    After:  v1={new_v1:.1f}, v2={new_v2:.1f}")
```

---

## Simple Ball Bouncing Implementation (간단한 공 튕기기 구현)

### Concept (개념)

For 2D ball-ball collision:

1. Check if balls are overlapping
2. Calculate collision normal (direction from center to center)
3. Calculate relative velocity along normal
4. Apply 1D elastic collision formula along normal
5. Separate balls to remove overlap

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

    def dot(self, other):
        return self.x * other.x + self.y * other.y

    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2)

    def normalized(self):
        mag = self.magnitude()
        if mag == 0:
            return Vector2(0, 0)
        return Vector2(self.x / mag, self.y / mag)

    def __repr__(self):
        return f"({self.x:.2f}, {self.y:.2f})"


class Ball:
    def __init__(self, x, y, radius, mass=1):
        self.position = Vector2(x, y)
        self.velocity = Vector2(0, 0)
        self.radius = radius
        self.mass = mass

    def update(self, dt):
        self.position = self.position + self.velocity * dt


def balls_colliding(ball1, ball2):
    """Check if two balls are overlapping"""
    diff = ball2.position - ball1.position
    dist = diff.magnitude()
    return dist < ball1.radius + ball2.radius


def resolve_ball_collision(ball1, ball2, restitution=1.0):
    """Resolve collision between two balls"""
    diff = ball2.position - ball1.position
    dist = diff.magnitude()

    if dist == 0:
        return  # Balls at same position, skip

    # Collision normal
    normal = diff.normalized()

    # Relative velocity
    rel_vel = ball1.velocity - ball2.velocity

    # Relative velocity along normal
    vel_along_normal = rel_vel.dot(normal)

    # Only resolve if balls moving toward each other
    if vel_along_normal > 0:
        return

    # Calculate impulse
    j = -(1 + restitution) * vel_along_normal
    j /= (1 / ball1.mass + 1 / ball2.mass)

    # Apply impulse
    impulse = normal * j
    ball1.velocity = ball1.velocity + impulse * (1 / ball1.mass)
    ball2.velocity = ball2.velocity - impulse * (1 / ball2.mass)

    # Separate balls
    overlap = ball1.radius + ball2.radius - dist
    separation = normal * (overlap / 2 + 0.1)
    ball1.position = ball1.position - separation
    ball2.position = ball2.position + separation


# Simulation: Two balls colliding
print("Two Ball Collision Simulation:")

ball1 = Ball(0, 50, radius=10, mass=1)
ball1.velocity = Vector2(50, 0)  # Moving right

ball2 = Ball(80, 50, radius=10, mass=1)
ball2.velocity = Vector2(-30, 0)  # Moving left

dt = 0.05

print("\nBefore collision:")
print(f"  Ball1: pos={ball1.position}, vel={ball1.velocity}")
print(f"  Ball2: pos={ball2.position}, vel={ball2.velocity}")

for frame in range(30):
    # Check collision
    if balls_colliding(ball1, ball2):
        print(f"\n  Collision at frame {frame}!")
        resolve_ball_collision(ball1, ball2, restitution=1.0)
        print(f"  Ball1: pos={ball1.position}, vel={ball1.velocity}")
        print(f"  Ball2: pos={ball2.position}, vel={ball2.velocity}")

    ball1.update(dt)
    ball2.update(dt)

print("\nAfter simulation:")
print(f"  Ball1: pos={ball1.position}, vel={ball1.velocity}")
print(f"  Ball2: pos={ball2.position}, vel={ball2.velocity}")


# Pool/Billiards simulation
print("\n--- Billiards Simulation ---")

balls = [
    Ball(100, 100, 10, 1),  # Cue ball
    Ball(200, 100, 10, 1),
    Ball(220, 90, 10, 1),
    Ball(220, 110, 10, 1),
]

# Hit cue ball
balls[0].velocity = Vector2(150, 5)

print("Initial state:")
for i, ball in enumerate(balls):
    print(f"  Ball {i}: pos={ball.position}, vel={ball.velocity}")

# Simulate
for frame in range(100):
    # Check all pairs for collision
    for i in range(len(balls)):
        for j in range(i + 1, len(balls)):
            if balls_colliding(balls[i], balls[j]):
                resolve_ball_collision(balls[i], balls[j], restitution=0.95)

    # Update positions
    for ball in balls:
        # Apply friction
        if ball.velocity.magnitude() > 0.1:
            friction = ball.velocity.normalized() * -0.5
            ball.velocity = ball.velocity + friction
        else:
            ball.velocity = Vector2(0, 0)

        ball.update(dt)

print("\nAfter collision:")
for i, ball in enumerate(balls):
    print(f"  Ball {i}: pos={ball.position}, vel={ball.velocity}")
```

---

## Summary (요약)

| Concept | Korean | Formula |
|---------|--------|---------|
| Reflection | 반사 | r = v - 2(v·n)n |
| Bounciness | 탄성 | r = v - (1+e)(v·n)n |
| Elastic Collision | 탄성 충돌 | Conserves momentum + energy |
| Impulse | 충격량 | j = -(1+e) × v_rel·n / (1/m₁ + 1/m₂) |

**Collision Response Steps:**
1. Detect collision
2. Calculate normal
3. Calculate relative velocity
4. Apply impulse to both objects
5. Separate objects

---

## Practice Problems (연습 문제)

1. A ball with velocity (5, -3) hits a floor (normal = (0, 1)). What's the reflected velocity with bounciness 0.8?

2. Two equal mass balls: Ball A moving at 10 m/s hits stationary Ball B. What are their velocities after elastic collision?

3. A 2kg ball moving at 5 m/s hits a 1kg ball moving at -2 m/s. Calculate post-collision velocities.

4. Implement a "breakout" style game where a ball bounces off walls and bricks.
