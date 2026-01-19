# Chapter 09. Trigonometry Applications (삼각함수 활용)

> Put trigonometry to work: rotation, circular motion, aiming systems, and area-of-effect attacks.

---

## Direction Vector from Angle (각도에서 방향 벡터)

### Concept (개념)
Converting an angle to a direction vector is one of the most common operations in games:

```
direction_x = cos(angle)
direction_y = sin(angle)
```

This creates a unit vector (length 1) pointing in the direction of the angle.

- 0° = (1, 0) = Right
- 90° = (0, 1) = Up
- 180° = (-1, 0) = Left
- 270° = (0, -1) = Down

### Game Example
A spaceship rotates and then thrusts forward. The thrust direction comes from the ship's angle.

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

    @staticmethod
    def from_angle(degrees):
        """Create unit direction vector from angle"""
        rad = math.radians(degrees)
        return Vector2(math.cos(rad), math.sin(rad))

    def __repr__(self):
        return f"({self.x:.3f}, {self.y:.3f})"


class Spaceship:
    def __init__(self, x, y):
        self.position = Vector2(x, y)
        self.velocity = Vector2(0, 0)
        self.angle = 90  # Facing up
        self.thrust_power = 100

    def rotate(self, delta_angle):
        self.angle = (self.angle + delta_angle) % 360

    def thrust(self, dt):
        """Apply thrust in facing direction"""
        direction = Vector2.from_angle(self.angle)
        acceleration = direction * self.thrust_power
        self.velocity = self.velocity + acceleration * dt

    def update(self, dt):
        self.position = self.position + self.velocity * dt


# Spaceship simulation
ship = Spaceship(400, 300)

print("Spaceship control:")
print(f"Initial: pos={ship.position}, angle={ship.angle}°")

# Rotate right 45°
ship.rotate(-45)
print(f"After rotate right: angle={ship.angle}°")

# Thrust
ship.thrust(0.1)
print(f"Direction vector: {Vector2.from_angle(ship.angle)}")
print(f"Velocity after thrust: {ship.velocity}")

# Update position
ship.update(0.1)
print(f"New position: {ship.position}")
```

---

## Circular Motion (원운동)

### Concept (개념)
To move an object in a circle:

```
x = center_x + radius * cos(angle)
y = center_y + radius * sin(angle)
```

Increase the angle over time for continuous motion:
```
angle += angular_speed * dt
```

Angular speed can be in:
- Radians per second
- Degrees per second
- Revolutions per second (multiply by 2π for radians)

### Game Example
- Orbiting objects (moons, particles)
- Circular enemy patterns
- Rotating menu items
- Clock hands

### Code Example
```python
import math

class OrbitingObject:
    def __init__(self, center_x, center_y, radius, speed_deg_per_sec):
        self.center_x = center_x
        self.center_y = center_y
        self.radius = radius
        self.angle = 0  # Current angle in degrees
        self.speed = speed_deg_per_sec

    def update(self, dt):
        """Update angle and position"""
        self.angle += self.speed * dt
        self.angle %= 360  # Keep in [0, 360)

    def get_position(self):
        """Calculate current position on circle"""
        rad = math.radians(self.angle)
        x = self.center_x + self.radius * math.cos(rad)
        y = self.center_y + self.radius * math.sin(rad)
        return (x, y)


# Moon orbiting a planet
moon = OrbitingObject(
    center_x=400, center_y=300,  # Planet position
    radius=100,                   # Orbit radius
    speed_deg_per_sec=45         # 45° per second = 8 seconds per orbit
)

print("Moon orbit simulation:")
dt = 0.5
for i in range(9):
    pos = moon.get_position()
    print(f"t={i*dt:.1f}s: angle={moon.angle:6.1f}°, pos=({pos[0]:6.1f}, {pos[1]:6.1f})")
    moon.update(dt)


# Multiple objects orbiting
class CircularFormation:
    def __init__(self, center_x, center_y, radius, num_objects, rotation_speed=30):
        self.center_x = center_x
        self.center_y = center_y
        self.radius = radius
        self.num_objects = num_objects
        self.base_angle = 0
        self.rotation_speed = rotation_speed

    def update(self, dt):
        self.base_angle += self.rotation_speed * dt

    def get_positions(self):
        """Get positions of all objects in formation"""
        positions = []
        angle_step = 360 / self.num_objects

        for i in range(self.num_objects):
            angle = self.base_angle + i * angle_step
            rad = math.radians(angle)
            x = self.center_x + self.radius * math.cos(rad)
            y = self.center_y + self.radius * math.sin(rad)
            positions.append((x, y))

        return positions


# 4 enemies in circular formation around boss
formation = CircularFormation(400, 300, radius=80, num_objects=4)

print("\nEnemy formation:")
for i, pos in enumerate(formation.get_positions()):
    print(f"  Enemy {i+1}: ({pos[0]:.1f}, {pos[1]:.1f})")
```

---

## Rotation (회전)

### Concept (개념)
To rotate a point around the origin by angle θ:

```
new_x = x * cos(θ) - y * sin(θ)
new_y = x * sin(θ) + y * cos(θ)
```

To rotate around an arbitrary center point:
1. Translate point so center is at origin
2. Apply rotation
3. Translate back

### Game Example
- Rotating sprites
- Turret aiming
- Weapon swing arcs
- Camera rotation

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def rotate(self, angle_degrees):
        """Rotate this vector around origin"""
        rad = math.radians(angle_degrees)
        cos_a = math.cos(rad)
        sin_a = math.sin(rad)

        new_x = self.x * cos_a - self.y * sin_a
        new_y = self.x * sin_a + self.y * cos_a

        return Vector2(new_x, new_y)

    def rotate_around(self, center, angle_degrees):
        """Rotate this vector around a center point"""
        # Translate to origin
        translated = Vector2(self.x - center.x, self.y - center.y)

        # Rotate
        rotated = translated.rotate(angle_degrees)

        # Translate back
        return Vector2(rotated.x + center.x, rotated.y + center.y)

    def __repr__(self):
        return f"({self.x:.2f}, {self.y:.2f})"


# Rotate a direction vector
forward = Vector2(1, 0)  # Facing right

print("Rotating direction vector:")
for angle in [0, 45, 90, 135, 180]:
    rotated = forward.rotate(angle)
    print(f"  Rotated {angle}°: {rotated}")


# Rotate a point around another point
sword_tip = Vector2(150, 100)
player_pos = Vector2(100, 100)

print("\nSword swing (rotating around player):")
for angle in [0, 30, 60, 90]:
    rotated_tip = sword_tip.rotate_around(player_pos, angle)
    print(f"  Angle {angle}°: sword tip at {rotated_tip}")
```

---

## Firing at Angles (각도로 발사)

### Concept (개념)
To fire a projectile at a specific angle:

1. Convert angle to direction: `dir = (cos(angle), sin(angle))`
2. Multiply by speed: `velocity = dir * speed`
3. Update position: `position += velocity * dt`

### Game Example
Weapons that fire at fixed angles, spread shots, or player-aimed projectiles.

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

    @staticmethod
    def from_angle(degrees, magnitude=1):
        rad = math.radians(degrees)
        return Vector2(math.cos(rad) * magnitude, math.sin(rad) * magnitude)

    def __repr__(self):
        return f"({self.x:.1f}, {self.y:.1f})"


class Projectile:
    def __init__(self, start_pos, angle, speed):
        self.position = Vector2(start_pos.x, start_pos.y)
        self.velocity = Vector2.from_angle(angle, speed)

    def update(self, dt):
        self.position = self.position + self.velocity * dt


def fire_spread_shot(position, center_angle, spread_angle, num_bullets, speed):
    """Fire multiple bullets in a spread pattern"""
    bullets = []

    if num_bullets == 1:
        bullets.append(Projectile(position, center_angle, speed))
    else:
        # Calculate angle between each bullet
        angle_step = spread_angle / (num_bullets - 1)
        start_angle = center_angle - spread_angle / 2

        for i in range(num_bullets):
            angle = start_angle + i * angle_step
            bullets.append(Projectile(position, angle, speed))

    return bullets


# Fire a 5-bullet spread shot
player_pos = Vector2(100, 100)
facing_angle = 45  # degrees

bullets = fire_spread_shot(
    position=player_pos,
    center_angle=facing_angle,
    spread_angle=30,  # 30° total spread
    num_bullets=5,
    speed=200
)

print(f"Spread shot at {facing_angle}° with 30° spread:")
for i, bullet in enumerate(bullets):
    angle = math.degrees(math.atan2(bullet.velocity.y, bullet.velocity.x))
    print(f"  Bullet {i+1}: angle={angle:.1f}°, velocity={bullet.velocity}")

# Simulate flight
print("\nBullet positions after 0.5 seconds:")
for bullet in bullets:
    bullet.update(0.5)
    print(f"  {bullet.position}")
```

---

## Fan-Shaped Attacks (부채꼴 공격)

### Concept (개념)
A fan-shaped (sector) attack hits targets within:
1. A certain distance (radius)
2. A certain angle range (arc)

To check if a target is hit:
1. Check distance: `distance <= attack_range`
2. Check angle: angle to target is within attack arc

### Game Example
- Melee weapon swings
- Cone attacks
- Breath weapons
- Flashlight illumination

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __sub__(self, other):
        return Vector2(self.x - other.x, self.y - other.y)

    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2)

    def angle(self):
        """Get angle of this vector in degrees"""
        return math.degrees(math.atan2(self.y, self.x))


def normalize_angle(angle):
    """Normalize angle to [-180, 180]"""
    while angle > 180:
        angle -= 360
    while angle < -180:
        angle += 360
    return angle


def is_in_fan(attacker_pos, attacker_angle, fan_range, fan_arc, target_pos):
    """
    Check if target is within a fan-shaped area.

    attacker_pos: Position of attacker
    attacker_angle: Direction attacker is facing (degrees)
    fan_range: Maximum distance of the fan
    fan_arc: Total arc angle of the fan (degrees)
    target_pos: Position to check
    """
    # Get vector to target
    to_target = target_pos - attacker_pos
    distance = to_target.magnitude()

    # Check distance
    if distance > fan_range:
        return False

    # Check angle
    angle_to_target = to_target.angle()
    angle_diff = normalize_angle(angle_to_target - attacker_angle)

    # Target must be within half the arc on either side
    half_arc = fan_arc / 2
    return abs(angle_diff) <= half_arc


class MeleeAttack:
    def __init__(self, attacker_pos, facing_angle, attack_range=100, arc=90):
        self.position = attacker_pos
        self.angle = facing_angle
        self.range = attack_range
        self.arc = arc

    def get_hit_targets(self, targets):
        """Return list of targets hit by this attack"""
        hit = []
        for target in targets:
            if is_in_fan(self.position, self.angle, self.range, self.arc, target['pos']):
                hit.append(target)
        return hit


# Player swings sword
player_pos = Vector2(0, 0)
player_facing = 45  # Facing upper-right

attack = MeleeAttack(player_pos, player_facing, attack_range=80, arc=90)

# Enemies at various positions
enemies = [
    {'name': 'Goblin A', 'pos': Vector2(50, 50)},   # In range, in arc
    {'name': 'Goblin B', 'pos': Vector2(100, 0)},   # In arc, out of range
    {'name': 'Goblin C', 'pos': Vector2(0, 50)},    # In range, edge of arc
    {'name': 'Goblin D', 'pos': Vector2(-50, 0)},   # Behind player
    {'name': 'Goblin E', 'pos': Vector2(40, 30)},   # In range, in arc
]

print(f"Sword swing: 90° arc, 80 range, facing {player_facing}°")
print("\nChecking enemies:")

for enemy in enemies:
    to_enemy = enemy['pos'] - player_pos
    dist = to_enemy.magnitude()
    angle = to_enemy.angle()
    hit = is_in_fan(player_pos, player_facing, 80, 90, enemy['pos'])
    status = "HIT!" if hit else "miss"
    print(f"  {enemy['name']}: dist={dist:.1f}, angle={angle:.1f}°, {status}")

hit_enemies = attack.get_hit_targets(enemies)
print(f"\nEnemies hit: {[e['name'] for e in hit_enemies]}")
```

---

## Spiral Motion (나선 운동)

### Concept (개념)
Combine circular motion with changing radius:

```
x = center_x + radius(t) * cos(angle)
y = center_y + radius(t) * sin(angle)
```

Where radius changes over time (increasing = spiral out, decreasing = spiral in).

### Game Example
- Tornado effects
- Spiral bullet patterns
- Particle effects

### Code Example
```python
import math

class SpiralBullet:
    def __init__(self, center_x, center_y, start_radius=0,
                 expansion_rate=50, angular_speed=180):
        self.center_x = center_x
        self.center_y = center_y
        self.radius = start_radius
        self.expansion_rate = expansion_rate  # pixels per second
        self.angular_speed = angular_speed    # degrees per second
        self.angle = 0

    def update(self, dt):
        self.angle += self.angular_speed * dt
        self.radius += self.expansion_rate * dt

    def get_position(self):
        rad = math.radians(self.angle)
        x = self.center_x + self.radius * math.cos(rad)
        y = self.center_y + self.radius * math.sin(rad)
        return (x, y)


# Create expanding spiral
spiral = SpiralBullet(200, 200, start_radius=10, expansion_rate=30, angular_speed=270)

print("Spiral bullet pattern:")
dt = 0.1
for i in range(15):
    pos = spiral.get_position()
    print(f"t={i*dt:.1f}s: angle={spiral.angle:6.1f}°, radius={spiral.radius:5.1f}, pos=({pos[0]:6.1f}, {pos[1]:6.1f})")
    spiral.update(dt)
```

---

## Summary (요약)

| Pattern | Korean | Formula |
|---------|--------|---------|
| Angle to Direction | 각도→방향 | (cos(θ), sin(θ)) |
| Circular Motion | 원운동 | (cx + r*cos(θ), cy + r*sin(θ)) |
| Rotation | 회전 | x'=x*cos-y*sin, y'=x*sin+y*cos |
| Fan Attack | 부채꼴 공격 | Check distance AND angle |
| Spiral | 나선 | Circular + changing radius |

---

## Practice Problems (연습 문제)

1. Create a direction vector for 135° angle.
2. An object orbits at radius 50, angular speed 90°/s. Where is it after 2 seconds?
3. Rotate point (10, 0) by 60° around the origin.
4. A fan attack has 120° arc and player faces 30°. What angle range does it cover?
