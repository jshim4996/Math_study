# Chapter 08. Trigonometry Basics (삼각함수 기초)

> Trigonometry connects angles to distances and directions - essential for rotation, aiming, and circular motion.

---

## Degrees and Radians (도와 라디안)

### Concept (개념)
Angles can be measured in two ways:

**Degrees (도)**:
- Full circle = 360°
- Right angle = 90°
- Intuitive for humans

**Radians (라디안)**:
- Full circle = 2π radians
- Right angle = π/2 radians
- Used in most math functions
- One radian = angle where arc length equals radius

### Conversion Formulas (변환 공식)
```
radians = degrees × (π / 180)
degrees = radians × (180 / π)
```

### Game Example
Most game engines use degrees for designers (easier to understand) but convert to radians internally for calculations.

### Code Example
```python
import math

def degrees_to_radians(degrees):
    """Convert degrees to radians"""
    return degrees * (math.pi / 180)

def radians_to_degrees(radians):
    """Convert radians to degrees"""
    return radians * (180 / math.pi)

# Common angles
angles_deg = [0, 30, 45, 60, 90, 180, 270, 360]

print("Degrees to Radians:")
for deg in angles_deg:
    rad = degrees_to_radians(deg)
    print(f"  {deg}° = {rad:.4f} rad = {rad/math.pi:.2f}π")

# In Python, math functions use radians
angle = 90
print(f"\nsin({angle}°) = {math.sin(degrees_to_radians(angle))}")  # 1.0
print(f"cos({angle}°) = {math.cos(degrees_to_radians(angle))}")    # 0.0
```

---

## Sine, Cosine, and Tangent (사인, 코사인, 탄젠트)

### Concept (개념)
In a right triangle:

```
        /|
       / |
  c   /  | b (opposite)
     /   |
    /θ___|
      a (adjacent)
```

**SOH-CAH-TOA**:
- **sin(θ)** = Opposite / Hypotenuse = b/c
- **cos(θ)** = Adjacent / Hypotenuse = a/c
- **tan(θ)** = Opposite / Adjacent = b/a = sin(θ)/cos(θ)

### Game Example
If you know the angle and distance to a target, you can find the horizontal and vertical components:
- Horizontal distance = distance × cos(angle)
- Vertical distance = distance × sin(angle)

### Code Example
```python
import math

def deg_to_rad(deg):
    return deg * math.pi / 180

# Right triangle with angle 30°, hypotenuse 10
angle_deg = 30
hypotenuse = 10

angle_rad = deg_to_rad(angle_deg)

opposite = hypotenuse * math.sin(angle_rad)   # Vertical
adjacent = hypotenuse * math.cos(angle_rad)   # Horizontal

print(f"Angle: {angle_deg}°")
print(f"Hypotenuse: {hypotenuse}")
print(f"Opposite (height): {opposite:.2f}")   # 5.0
print(f"Adjacent (width): {adjacent:.2f}")    # 8.66

# Verify with tan
print(f"\ntan({angle_deg}°) = {math.tan(angle_rad):.4f}")
print(f"opposite/adjacent = {opposite/adjacent:.4f}")  # Same!
```

---

## The Right Triangle (직각삼각형)

### Concept (개념)
Right triangles are everywhere in games:
- Distance calculations
- Slope/incline
- Component breakdown of vectors
- Projectile trajectories

Key relationships:
```
a² + b² = c²  (Pythagorean theorem)
sin²(θ) + cos²(θ) = 1
```

### Game Example
A ramp has length 20 and angle 30°. What's its height?
```
height = length × sin(30°) = 20 × 0.5 = 10
```

### Code Example
```python
import math

def solve_right_triangle(known_values):
    """
    Solve a right triangle given some known values.
    known_values can contain: angle, hypotenuse, opposite, adjacent
    """
    result = dict(known_values)

    # If we have angle and hypotenuse
    if 'angle' in result and 'hypotenuse' in result:
        angle_rad = math.radians(result['angle'])
        result['opposite'] = result['hypotenuse'] * math.sin(angle_rad)
        result['adjacent'] = result['hypotenuse'] * math.cos(angle_rad)

    # If we have angle and adjacent
    elif 'angle' in result and 'adjacent' in result:
        angle_rad = math.radians(result['angle'])
        result['hypotenuse'] = result['adjacent'] / math.cos(angle_rad)
        result['opposite'] = result['adjacent'] * math.tan(angle_rad)

    # If we have angle and opposite
    elif 'angle' in result and 'opposite' in result:
        angle_rad = math.radians(result['angle'])
        result['hypotenuse'] = result['opposite'] / math.sin(angle_rad)
        result['adjacent'] = result['opposite'] / math.tan(angle_rad)

    return result


# Example: Aiming at 45° angle, projectile travels 100 units
trajectory = solve_right_triangle({
    'angle': 45,
    'hypotenuse': 100
})

print("45° trajectory of 100 units:")
print(f"  Horizontal distance: {trajectory['adjacent']:.2f}")
print(f"  Vertical distance: {trajectory['opposite']:.2f}")


# Example: Ramp with 30° incline, 5 meters high
ramp = solve_right_triangle({
    'angle': 30,
    'opposite': 5  # height
})

print(f"\nRamp with 30° incline, 5m high:")
print(f"  Ramp length: {ramp['hypotenuse']:.2f}m")
print(f"  Horizontal span: {ramp['adjacent']:.2f}m")
```

---

## The Unit Circle (단위원)

### Concept (개념)
The **unit circle (단위원)** is a circle with radius 1 centered at the origin.

For any angle θ, the point on the unit circle is:
```
x = cos(θ)
y = sin(θ)
```

This is incredibly powerful:
- Any direction can be represented as (cos(θ), sin(θ))
- The direction is automatically normalized (length 1)
- Easy to create rotation

### Game Example
Creating a direction vector from an angle:
```python
direction_x = cos(angle)
direction_y = sin(angle)
```

### Code Example
```python
import math

def angle_to_direction(angle_degrees):
    """Convert angle to unit direction vector"""
    angle_rad = math.radians(angle_degrees)
    return (math.cos(angle_rad), math.sin(angle_rad))

def direction_to_angle(x, y):
    """Convert direction vector to angle"""
    return math.degrees(math.atan2(y, x))

# Standard directions
print("Standard directions from angles:")
angles = [0, 45, 90, 135, 180, 225, 270, 315]
direction_names = ['Right', 'Up-Right', 'Up', 'Up-Left',
                   'Left', 'Down-Left', 'Down', 'Down-Right']

for angle, name in zip(angles, direction_names):
    dx, dy = angle_to_direction(angle)
    print(f"  {angle:3}° ({name:10}): ({dx:6.3f}, {dy:6.3f})")

# Verify these are unit vectors (length = 1)
print("\nVerifying unit length:")
dx, dy = angle_to_direction(37)  # Arbitrary angle
length = math.sqrt(dx**2 + dy**2)
print(f"  Direction at 37°: ({dx:.4f}, {dy:.4f}), length: {length}")
```

---

## Key Angle Values (주요 각도 값)

### Concept (개념)
Memorizing these common values helps with quick calculations:

| Angle | sin | cos | tan |
|-------|-----|-----|-----|
| 0° | 0 | 1 | 0 |
| 30° | 1/2 | √3/2 | 1/√3 |
| 45° | √2/2 | √2/2 | 1 |
| 60° | √3/2 | 1/2 | √3 |
| 90° | 1 | 0 | undefined |

### Code Example
```python
import math

# Key angles to remember
key_angles = [0, 30, 45, 60, 90, 180, 270, 360]

print("Key Trigonometric Values:")
print("-" * 50)
print(f"{'Angle':>6} | {'sin':>8} | {'cos':>8} | {'tan':>8}")
print("-" * 50)

for deg in key_angles:
    rad = math.radians(deg)
    s = math.sin(rad)
    c = math.cos(rad)

    # Handle tan undefined at 90°, 270°
    if deg in [90, 270]:
        t = "undef"
    else:
        t = f"{math.tan(rad):8.4f}"

    print(f"{deg:>6}° | {s:8.4f} | {c:8.4f} | {t}")
```

---

## Trigonometry in Games

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
        """Create vector from angle"""
        rad = math.radians(degrees)
        return Vector2(
            math.cos(rad) * magnitude,
            math.sin(rad) * magnitude
        )

    def __repr__(self):
        return f"({self.x:.2f}, {self.y:.2f})"


class Turret:
    def __init__(self, x, y):
        self.position = Vector2(x, y)
        self.angle = 0  # Facing right
        self.rotation_speed = 90  # degrees per second
        self.projectile_speed = 200

    def rotate(self, direction, dt):
        """Rotate turret. direction: -1=left, 1=right"""
        self.angle += direction * self.rotation_speed * dt
        # Keep angle in [0, 360)
        self.angle = self.angle % 360

    def get_direction(self):
        """Get direction vector turret is facing"""
        return Vector2.from_angle(self.angle)

    def fire(self):
        """Create projectile in facing direction"""
        direction = Vector2.from_angle(self.angle)
        velocity = direction * self.projectile_speed
        return {
            'position': Vector2(self.position.x, self.position.y),
            'velocity': velocity
        }


# Turret simulation
turret = Turret(100, 100)

print("Turret simulation:")
print(f"Initial angle: {turret.angle}°, direction: {turret.get_direction()}")

# Rotate 45 degrees
turret.rotate(1, 0.5)  # Right for 0.5 seconds
print(f"After rotating: {turret.angle}°, direction: {turret.get_direction()}")

# Fire!
projectile = turret.fire()
print(f"Projectile velocity: {projectile['velocity']}")


# Simulate projectile flight
print("\nProjectile trajectory:")
dt = 0.1
for i in range(5):
    projectile['position'] = projectile['position'] + projectile['velocity'] * dt
    print(f"  t={i*dt+dt:.1f}s: {projectile['position']}")
```

---

## Periodic Nature of Trig Functions (삼각함수의 주기성)

### Concept (개념)
Trigonometric functions repeat (are periodic):
- sin and cos repeat every 2π (360°)
- tan repeats every π (180°)

Useful identities:
```
sin(θ + 360°) = sin(θ)
cos(θ + 360°) = cos(θ)
sin(-θ) = -sin(θ)
cos(-θ) = cos(θ)
sin(90° - θ) = cos(θ)
cos(90° - θ) = sin(θ)
```

### Game Example
Creating smooth oscillating motion (breathing, floating, pulsing):

```python
import math

def oscillate(time, frequency=1, amplitude=1, offset=0):
    """Create smooth oscillating value"""
    return math.sin(time * frequency * 2 * math.pi) * amplitude + offset

# Breathing effect: scale between 0.9 and 1.1
time = 0
dt = 0.1

print("Breathing animation (scale factor):")
for i in range(20):
    time += dt
    scale = oscillate(time, frequency=0.5, amplitude=0.1, offset=1.0)
    bar = "█" * int(scale * 20)
    print(f"  t={time:.1f}: {scale:.2f} {bar}")
```

---

## Summary (요약)

| Term | Korean | Description |
|------|--------|-------------|
| Degree | 도 | 360° = full circle |
| Radian | 라디안 | 2π = full circle |
| Sine | 사인 | opposite / hypotenuse |
| Cosine | 코사인 | adjacent / hypotenuse |
| Tangent | 탄젠트 | opposite / adjacent |
| Unit Circle | 단위원 | Circle with radius 1 |
| Periodic | 주기적 | Repeats at regular intervals |

---

## Conversion Quick Reference

| Degrees | Radians | sin | cos |
|---------|---------|-----|-----|
| 0° | 0 | 0 | 1 |
| 30° | π/6 | 0.5 | 0.866 |
| 45° | π/4 | 0.707 | 0.707 |
| 60° | π/3 | 0.866 | 0.5 |
| 90° | π/2 | 1 | 0 |
| 180° | π | 0 | -1 |
| 270° | 3π/2 | -1 | 0 |

---

## Practice Problems (연습 문제)

1. Convert 135° to radians.
2. A projectile is fired at 60° with speed 100. What are the horizontal and vertical velocity components?
3. Create a direction vector for angle 225°.
4. A platform oscillates up and down. If it moves with sin(t), when is it at max height (t=0 to 2π)?
