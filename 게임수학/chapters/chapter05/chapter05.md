# Chapter 05. Vector Applications (벡터 활용)

> Apply vector math to create smooth character movement, targeting systems, and projectile physics.

---

## Character Movement (캐릭터 이동)

### Concept (개념)
Character movement in games follows this basic formula:

```
new_position = old_position + velocity * delta_time
```

Where:
- **position (위치)**: Current location vector
- **velocity (속도)**: Direction and speed of movement
- **delta_time (델타 타임)**: Time since last frame (for frame-rate independence)

### Game Example
A top-down game where the player moves using WASD:

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

    def __repr__(self):
        return f"({self.x:.2f}, {self.y:.2f})"


class Character:
    def __init__(self, x, y):
        self.position = Vector2(x, y)
        self.velocity = Vector2(0, 0)
        self.move_speed = 150  # pixels per second

    def set_input_direction(self, input_x, input_y):
        """Set movement direction from input"""
        input_dir = Vector2(input_x, input_y)

        # Normalize to prevent fast diagonal movement
        if input_dir.magnitude() > 0:
            input_dir = input_dir.normalized()

        self.velocity = input_dir * self.move_speed

    def update(self, dt):
        """Update position each frame"""
        self.position = self.position + self.velocity * dt


# Game simulation
player = Character(100, 100)

# Player presses 'D' (move right)
player.set_input_direction(1, 0)
print(f"Velocity: {player.velocity}")

# Simulate 10 frames at 60 FPS
dt = 1/60
for frame in range(10):
    player.update(dt)
    print(f"Frame {frame+1}: Position {player.position}")
```

---

## Moving Toward a Target (목표 지점으로 이동)

### Concept (개념)
To move an object toward a target:

1. Calculate direction: `direction = target - current_position`
2. Normalize: `direction = direction.normalized()`
3. Move: `new_position = position + direction * speed * dt`

### Game Example
Enemy AI that chases the player, homing missiles, or NPCs walking to waypoints.

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
        return math.sqrt(self.x**2 + self.y**2)

    def normalized(self):
        mag = self.magnitude()
        if mag == 0:
            return Vector2(0, 0)
        return Vector2(self.x / mag, self.y / mag)

    def __repr__(self):
        return f"({self.x:.1f}, {self.y:.1f})"


class Enemy:
    def __init__(self, x, y):
        self.position = Vector2(x, y)
        self.speed = 100

    def move_toward(self, target_pos, dt):
        """Move toward target position"""
        # Get direction to target
        direction = target_pos - self.position
        distance = direction.magnitude()

        # Stop if very close (within 1 pixel)
        if distance < 1:
            return

        # Normalize direction and move
        direction = direction.normalized()
        movement = direction * self.speed * dt

        # Don't overshoot the target
        if movement.magnitude() > distance:
            self.position = Vector2(target_pos.x, target_pos.y)
        else:
            self.position = self.position + movement


# Enemy chases player
enemy = Enemy(0, 0)
player_pos = Vector2(100, 100)

print("Enemy chasing player:")
dt = 0.1  # 10 FPS for visibility
for i in range(20):
    distance = (player_pos - enemy.position).magnitude()
    print(f"Step {i}: Enemy at {enemy.position}, distance: {distance:.1f}")
    enemy.move_toward(player_pos, dt)

    if distance < 1:
        print("Enemy reached player!")
        break
```

---

## Separating Speed and Direction (속도와 방향 분리)

### Concept (개념)
It's often useful to store movement as:
- **Direction**: Unit vector (normalized, magnitude = 1)
- **Speed**: Scalar (just a number)

Then: `velocity = direction * speed`

Benefits:
- Easy to change speed without affecting direction
- Easy to change direction without affecting speed
- Cleaner code for speed buffs/debuffs

### Game Example
A racing game where:
- The car's direction changes based on steering
- The car's speed changes based on acceleration/braking
- These are independent systems

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

    @staticmethod
    def from_angle(angle_deg):
        """Create unit vector from angle"""
        rad = math.radians(angle_deg)
        return Vector2(math.cos(rad), math.sin(rad))

    def __repr__(self):
        return f"({self.x:.2f}, {self.y:.2f})"


class Car:
    def __init__(self, x, y):
        self.position = Vector2(x, y)
        self.direction = Vector2(1, 0)  # Facing right (unit vector)
        self.speed = 0
        self.max_speed = 200
        self.acceleration = 100
        self.rotation_speed = 180  # degrees per second

    def accelerate(self, dt):
        """Increase speed"""
        self.speed = min(self.speed + self.acceleration * dt, self.max_speed)

    def brake(self, dt):
        """Decrease speed"""
        self.speed = max(self.speed - self.acceleration * 2 * dt, 0)

    def turn(self, direction, dt):
        """Turn left (-1) or right (+1)"""
        # Get current angle
        current_angle = math.degrees(math.atan2(self.direction.y, self.direction.x))

        # Rotate
        new_angle = current_angle + direction * self.rotation_speed * dt

        # Update direction (keep it normalized)
        self.direction = Vector2.from_angle(new_angle)

    def update(self, dt):
        """Update position"""
        velocity = self.direction * self.speed
        self.position = self.position + velocity * dt


# Racing simulation
car = Car(0, 0)

print("Racing simulation:")
dt = 0.1

# Accelerate for a bit
for i in range(5):
    car.accelerate(dt)
    car.update(dt)
    print(f"Accelerating: pos={car.position}, speed={car.speed:.1f}")

# Turn right while moving
for i in range(5):
    car.turn(1, dt)  # Turn right
    car.update(dt)
    print(f"Turning: pos={car.position}, dir={car.direction}")
```

---

## Projectile Movement (발사체 이동)

### Concept (개념)
Projectiles (bullets, arrows, fireballs) typically:
1. Start at a spawn position
2. Have an initial velocity (direction * speed)
3. Move in a straight line (or curve with gravity)

For straight projectiles:
```
position = start_position + velocity * time
```

### Game Example
A bullet fired from a gun toward the mouse cursor position.

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
        return math.sqrt(self.x**2 + self.y**2)

    def normalized(self):
        mag = self.magnitude()
        if mag == 0:
            return Vector2(0, 0)
        return Vector2(self.x / mag, self.y / mag)

    def __repr__(self):
        return f"({self.x:.1f}, {self.y:.1f})"


class Projectile:
    def __init__(self, start_pos, target_pos, speed):
        self.position = Vector2(start_pos.x, start_pos.y)

        # Calculate direction to target
        direction = (target_pos - start_pos).normalized()

        # Set velocity
        self.velocity = direction * speed
        self.alive = True
        self.max_range = 500
        self.distance_traveled = 0

    def update(self, dt):
        """Update projectile position"""
        if not self.alive:
            return

        # Move
        movement = self.velocity * dt
        self.position = self.position + movement
        self.distance_traveled += movement.magnitude()

        # Check if exceeded range
        if self.distance_traveled > self.max_range:
            self.alive = False


class Weapon:
    def __init__(self, owner_pos):
        self.position = owner_pos
        self.projectiles = []
        self.fire_rate = 0.2  # seconds between shots
        self.cooldown = 0
        self.projectile_speed = 400

    def fire(self, target_pos):
        """Fire a projectile toward target"""
        if self.cooldown > 0:
            return None

        projectile = Projectile(self.position, target_pos, self.projectile_speed)
        self.projectiles.append(projectile)
        self.cooldown = self.fire_rate
        return projectile

    def update(self, dt):
        """Update all projectiles and cooldown"""
        self.cooldown = max(0, self.cooldown - dt)

        for proj in self.projectiles:
            proj.update(dt)

        # Remove dead projectiles
        self.projectiles = [p for p in self.projectiles if p.alive]


# Shooting simulation
player_pos = Vector2(100, 100)
weapon = Weapon(player_pos)

# Fire at enemy
enemy_pos = Vector2(400, 200)
bullet = weapon.fire(enemy_pos)

print("Bullet trajectory:")
dt = 0.05
while bullet.alive:
    weapon.update(dt)
    print(f"Bullet at {bullet.position}")
```

---

## Multiple Movement Patterns

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
        return math.sqrt(self.x**2 + self.y**2)

    def normalized(self):
        mag = self.magnitude()
        if mag == 0:
            return Vector2(0, 0)
        return Vector2(self.x / mag, self.y / mag)


class Enemy:
    def __init__(self, x, y, pattern="chase"):
        self.position = Vector2(x, y)
        self.speed = 80
        self.pattern = pattern
        self.time = 0
        self.start_pos = Vector2(x, y)

    def update(self, dt, player_pos=None):
        self.time += dt

        if self.pattern == "chase":
            self._chase_pattern(dt, player_pos)
        elif self.pattern == "patrol":
            self._patrol_pattern(dt)
        elif self.pattern == "circle":
            self._circle_pattern(dt)
        elif self.pattern == "zigzag":
            self._zigzag_pattern(dt)

    def _chase_pattern(self, dt, player_pos):
        """Move directly toward player"""
        if player_pos is None:
            return
        direction = (player_pos - self.position).normalized()
        self.position = self.position + direction * self.speed * dt

    def _patrol_pattern(self, dt):
        """Move back and forth horizontally"""
        offset = math.sin(self.time * 2) * 100
        self.position.x = self.start_pos.x + offset

    def _circle_pattern(self, dt):
        """Move in a circle"""
        radius = 50
        self.position.x = self.start_pos.x + math.cos(self.time * 2) * radius
        self.position.y = self.start_pos.y + math.sin(self.time * 2) * radius

    def _zigzag_pattern(self, dt):
        """Move forward with zigzag"""
        # Move forward
        self.position.x += self.speed * dt
        # Zigzag vertically
        self.position.y = self.start_pos.y + math.sin(self.time * 5) * 30


# Demonstrate patterns
enemies = [
    Enemy(100, 100, "patrol"),
    Enemy(100, 200, "circle"),
    Enemy(100, 300, "zigzag"),
]

print("Enemy movement patterns:")
dt = 0.1
for frame in range(10):
    print(f"\nFrame {frame}:")
    for i, enemy in enumerate(enemies):
        enemy.update(dt)
        print(f"  {enemy.pattern}: ({enemy.position.x:.1f}, {enemy.position.y:.1f})")
```

---

## Smooth Following (부드러운 추적)

### Concept (개념)
Instead of instantly moving toward a target, we can smoothly interpolate for more natural movement:

```
position = lerp(position, target, smoothness * dt)
```

Or using velocity dampening:
```
velocity = (target - position) * follow_speed
position = position + velocity * dt
```

### Game Example
Camera following player, UI elements sliding into position, enemies with "soft" chase behavior.

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

    @staticmethod
    def lerp(a, b, t):
        """Linear interpolation between two vectors"""
        return Vector2(
            a.x + (b.x - a.x) * t,
            a.y + (b.y - a.y) * t
        )

    def __repr__(self):
        return f"({self.x:.1f}, {self.y:.1f})"


class SmoothFollower:
    def __init__(self, x, y, smoothness=5):
        self.position = Vector2(x, y)
        self.smoothness = smoothness

    def follow(self, target, dt):
        """Smoothly move toward target"""
        # Lerp toward target (smoothness controls how fast)
        t = min(1, self.smoothness * dt)
        self.position = Vector2.lerp(self.position, target, t)


# Camera smoothly following player
camera = SmoothFollower(0, 0, smoothness=3)
player_pos = Vector2(200, 150)

print("Smooth camera follow:")
dt = 0.1
for frame in range(20):
    camera.follow(player_pos, dt)
    print(f"Frame {frame}: Camera at {camera.position}")
```

---

## Summary (요약)

| Pattern | Korean | Description |
|---------|--------|-------------|
| Direct Movement | 직접 이동 | position += velocity * dt |
| Move Toward | 목표 추적 | direction = (target - pos).normalized() |
| Separate Speed/Dir | 속도/방향 분리 | velocity = direction * speed |
| Projectile | 발사체 | Spawn, set direction, update position |
| Smooth Follow | 부드러운 추적 | Use lerp for gradual movement |

---

## Practice Problems (연습 문제)

1. A character at (50, 50) needs to move toward (200, 100) at speed 60. What is the velocity vector?
2. Implement a "flee" behavior where an enemy moves away from the player.
3. Create a projectile that fires at a 45-degree angle upward.
4. Make a camera that follows the player but stays within screen bounds.
