# Chapter 18. Interpolation (보간)

> Smoothly transition between values: positions, colors, sizes, and more.

---

## What is Interpolation (보간이란 무엇인가)

### Concept (개념)

**Interpolation** finds values between two known points. Given start and end values, we calculate in-between values based on a parameter t (0 to 1).

**Why use interpolation?**
- Smooth animations
- Gradual transitions
- Natural movement
- Blending between states

**The parameter t:**
- t = 0: Start value
- t = 1: End value
- t = 0.5: Halfway between

### Game Example
- Camera smoothly following player
- Health bar filling up
- Color transitions
- Object movement along a path

### Code Example
```python
# Basic concept: t goes from 0 to 1
def show_t_progression():
    print("t progression over time:")
    duration = 1.0  # seconds
    for frame in range(11):
        t = frame / 10.0  # 0.0, 0.1, 0.2, ... 1.0
        print(f"  t = {t:.1f}")


show_t_progression()


# Calculating t from time
class Timer:
    def __init__(self, duration):
        self.duration = duration
        self.elapsed = 0

    def update(self, dt):
        self.elapsed += dt

    def get_t(self):
        """Get interpolation parameter (0 to 1)"""
        return min(self.elapsed / self.duration, 1.0)

    def is_complete(self):
        return self.elapsed >= self.duration


# Simulate timer
timer = Timer(duration=2.0)
print("\nTimer simulation (2 second duration):")
for i in range(25):
    t = timer.get_t()
    print(f"  Elapsed: {timer.elapsed:.1f}s, t = {t:.2f}")
    timer.update(0.1)  # 100ms per frame
    if timer.is_complete():
        break
```

---

## Linear Interpolation: Lerp (선형 보간)

### Concept (개념)

**Lerp** (Linear Interpolation) is the simplest form:

```
result = start + (end - start) * t
```

Or equivalently:
```
result = start * (1 - t) + end * t
```

Works for any numeric value: numbers, vectors, colors.

### Game Example
- Moving an object from A to B over time
- Fading UI elements
- Transitioning between day/night lighting

### Code Example
```python
def lerp(start, end, t):
    """Linear interpolation between start and end"""
    return start + (end - start) * t


# Lerp numbers
print("Lerping from 0 to 100:")
for t in [0, 0.25, 0.5, 0.75, 1.0]:
    value = lerp(0, 100, t)
    print(f"  t={t}: {value}")


# Lerp vectors
class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"({self.x:.1f}, {self.y:.1f})"


def lerp_vector(v1, v2, t):
    """Lerp between two vectors"""
    return Vector2(
        lerp(v1.x, v2.x, t),
        lerp(v1.y, v2.y, t)
    )


start_pos = Vector2(0, 0)
end_pos = Vector2(100, 50)

print("\nLerping position:")
for t in [0, 0.25, 0.5, 0.75, 1.0]:
    pos = lerp_vector(start_pos, end_pos, t)
    print(f"  t={t}: {pos}")


# Lerp colors (RGB)
def lerp_color(c1, c2, t):
    """Lerp between two RGB colors"""
    return (
        int(lerp(c1[0], c2[0], t)),
        int(lerp(c1[1], c2[1], t)),
        int(lerp(c1[2], c2[2], t))
    )


red = (255, 0, 0)
blue = (0, 0, 255)

print("\nLerping color (red to blue):")
for t in [0, 0.25, 0.5, 0.75, 1.0]:
    color = lerp_color(red, blue, t)
    print(f"  t={t}: RGB{color}")
```

---

## Smooth Movement Implementation (부드러운 이동 구현)

### Concept (개념)

For smooth movement toward a target, there are several approaches:

**1. Lerp with fixed t (smooth follow):**
```python
position = lerp(position, target, 0.1)  # Each frame, move 10% closer
```

**2. Lerp over fixed duration:**
```python
t = elapsed_time / duration
position = lerp(start, end, t)
```

**3. Move with max speed:**
```python
direction = (target - position).normalized()
position += direction * speed * dt
```

### Game Example
- Camera smooth follow
- UI elements sliding in
- Cursor following mouse

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

    def __repr__(self):
        return f"({self.x:.2f}, {self.y:.2f})"


def lerp_vector(v1, v2, t):
    return Vector2(
        v1.x + (v2.x - v1.x) * t,
        v1.y + (v2.y - v1.y) * t
    )


# Method 1: Smooth follow (exponential approach)
class SmoothFollowCamera:
    def __init__(self, start_pos):
        self.position = Vector2(start_pos.x, start_pos.y)
        self.smoothness = 0.1  # Lower = smoother but slower

    def update(self, target):
        self.position = lerp_vector(self.position, target, self.smoothness)


# Method 2: Timed movement
class TimedMove:
    def __init__(self, start, end, duration):
        self.start = start
        self.end = end
        self.duration = duration
        self.elapsed = 0
        self.position = Vector2(start.x, start.y)

    def update(self, dt):
        self.elapsed += dt
        t = min(self.elapsed / self.duration, 1.0)
        self.position = lerp_vector(self.start, self.end, t)
        return t >= 1.0  # Return True when complete


# Simulate smooth follow
print("Smooth Follow Camera:")
camera = SmoothFollowCamera(Vector2(0, 0))
target = Vector2(100, 50)

for frame in range(15):
    print(f"  Frame {frame:2d}: camera={camera.position}, target={target}")
    camera.update(target)


# Simulate timed movement
print("\nTimed Movement (2 seconds):")
mover = TimedMove(Vector2(0, 0), Vector2(100, 50), duration=2.0)

dt = 0.2
frame = 0
while True:
    t = mover.elapsed / mover.duration
    print(f"  t={t:.1f}: position={mover.position}")
    done = mover.update(dt)
    if done:
        print(f"  Complete: position={mover.position}")
        break
    frame += 1
```

---

## Color Interpolation, Size Interpolation (색상 보간, 크기 보간)

### Concept (개념)

Lerp works for any numeric property:

**Colors:**
- RGB: lerp each component
- HSV: lerp hue for better color transitions

**Sizes:**
- Lerp width and height
- Uniform scale: lerp single scale factor

**Opacity:**
- Lerp alpha value (0 to 255 or 0 to 1)

### Game Example
- Damage flash (white to normal color)
- Health bar changing color (green → yellow → red)
- Growing/shrinking effects
- Fade in/out

### Code Example
```python
def lerp(a, b, t):
    return a + (b - a) * t


def clamp(value, min_val, max_val):
    return max(min_val, min(value, max_val))


class Color:
    def __init__(self, r, g, b, a=255):
        self.r = r
        self.g = g
        self.b = b
        self.a = a

    def lerp_to(self, other, t):
        return Color(
            int(lerp(self.r, other.r, t)),
            int(lerp(self.g, other.g, t)),
            int(lerp(self.b, other.b, t)),
            int(lerp(self.a, other.a, t))
        )

    def __repr__(self):
        return f"RGBA({self.r}, {self.g}, {self.b}, {self.a})"


# Health bar color based on health percentage
def get_health_color(health_percent):
    """Return color from green (100%) to red (0%)"""
    green = Color(0, 255, 0)
    yellow = Color(255, 255, 0)
    red = Color(255, 0, 0)

    if health_percent > 0.5:
        # Green to Yellow (100% to 50%)
        t = (health_percent - 0.5) / 0.5  # 1 at 100%, 0 at 50%
        return yellow.lerp_to(green, t)
    else:
        # Yellow to Red (50% to 0%)
        t = health_percent / 0.5  # 1 at 50%, 0 at 0%
        return red.lerp_to(yellow, t)


print("Health bar colors:")
for hp in [1.0, 0.75, 0.5, 0.25, 0.0]:
    color = get_health_color(hp)
    print(f"  {int(hp*100):3d}% health: {color}")


# Damage flash effect
class DamageFlash:
    def __init__(self, original_color):
        self.original = original_color
        self.flash_color = Color(255, 255, 255)  # White flash
        self.duration = 0.2
        self.elapsed = 0
        self.is_flashing = False

    def trigger(self):
        self.is_flashing = True
        self.elapsed = 0

    def update(self, dt):
        if not self.is_flashing:
            return self.original

        self.elapsed += dt
        t = self.elapsed / self.duration

        if t >= 1.0:
            self.is_flashing = False
            return self.original

        # Flash white then return to original
        return self.flash_color.lerp_to(self.original, t)


# Size interpolation for UI
class ScaleAnimation:
    def __init__(self, start_scale, end_scale, duration):
        self.start = start_scale
        self.end = end_scale
        self.duration = duration
        self.elapsed = 0

    def update(self, dt):
        self.elapsed = min(self.elapsed + dt, self.duration)
        t = self.elapsed / self.duration
        return lerp(self.start, self.end, t)

    def is_complete(self):
        return self.elapsed >= self.duration


# Pop-up animation (scale from 0 to 1)
print("\nPop-up scale animation:")
popup = ScaleAnimation(0.0, 1.0, duration=0.3)
for i in range(7):
    scale = popup.update(0.05)
    print(f"  Frame {i}: scale = {scale:.2f}")
```

---

## Spherical Linear Interpolation: Slerp Concept (구면 선형 보간)

### Concept (개념)

**Slerp** interpolates along the surface of a sphere. Used for:
- Rotation interpolation (quaternions)
- Direction vectors (unit vectors)

**Why not just lerp?**
- Lerping unit vectors doesn't maintain unit length
- Lerped rotations take shortcuts through invalid states

**Formula (simplified for 2D unit vectors):**
```
omega = angle between v1 and v2
slerp = (sin((1-t)*omega)/sin(omega)) * v1 + (sin(t*omega)/sin(omega)) * v2
```

### Game Example
- Smooth camera rotation
- Character turning
- Quaternion interpolation for 3D rotation

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
        return f"({self.x:.3f}, {self.y:.3f})"


def lerp_vector(v1, v2, t):
    return Vector2(
        v1.x + (v2.x - v1.x) * t,
        v1.y + (v2.y - v1.y) * t
    )


def slerp(v1, v2, t):
    """Spherical linear interpolation for unit vectors"""
    # Clamp dot product to avoid acos errors
    dot = max(-1.0, min(1.0, v1.dot(v2)))

    # Angle between vectors
    omega = math.acos(dot)

    # If vectors are very close, use lerp
    if omega < 0.001:
        return lerp_vector(v1, v2, t).normalized()

    sin_omega = math.sin(omega)
    a = math.sin((1 - t) * omega) / sin_omega
    b = math.sin(t * omega) / sin_omega

    return Vector2(
        v1.x * a + v2.x * b,
        v1.y * a + v2.y * b
    )


# Compare lerp vs slerp for direction vectors
dir1 = Vector2(1, 0)   # Pointing right
dir2 = Vector2(0, 1)   # Pointing up

print("Lerp vs Slerp for unit direction vectors:")
print("(Lerp doesn't maintain unit length!)\n")

for t in [0, 0.25, 0.5, 0.75, 1.0]:
    lerped = lerp_vector(dir1, dir2, t)
    slerped = slerp(dir1, dir2, t)

    print(f"t={t}:")
    print(f"  Lerp:  {lerped}, length={lerped.magnitude():.3f}")
    print(f"  Slerp: {slerped}, length={slerped.magnitude():.3f}")


# Practical: smooth direction change
print("\nSmooth direction change over time:")
current_dir = Vector2(1, 0)
target_dir = Vector2(-1, 0)  # Opposite direction

for i in range(11):
    t = i / 10.0
    direction = slerp(current_dir, target_dir, t)
    angle = math.degrees(math.atan2(direction.y, direction.x))
    print(f"  t={t:.1f}: direction={direction}, angle={angle:6.1f}°")
```

---

## Summary (요약)

| Type | Korean | Use Case |
|------|--------|----------|
| Lerp | 선형 보간 | Numbers, positions, colors |
| Smooth Follow | 부드러운 추적 | Camera, UI elements |
| Color Lerp | 색상 보간 | Transitions, effects |
| Slerp | 구면 선형 보간 | Rotations, directions |

**Formula:**
```
lerp(a, b, t) = a + (b - a) * t
             = a * (1 - t) + b * t
```

---

## Practice Problems (연습 문제)

1. Lerp from 20 to 80 with t=0.3. What's the result?

2. A camera at (0, 0) follows a player at (100, 50) using lerp with t=0.15. Where is the camera after 3 frames?

3. Create a color that's 70% blue and 30% red using lerp.

4. Why does lerping between two unit vectors not give a unit vector at t=0.5?
