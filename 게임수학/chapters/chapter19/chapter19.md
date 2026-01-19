# Chapter 19. Curves (곡선)

> Beyond straight lines: Bezier curves for smooth paths, animations, and procedural content.

---

## Why We Need Curves (곡선이 필요한 이유)

### Concept (개념)

Linear interpolation creates straight-line movement. But games often need:
- Smooth, curved paths
- Natural acceleration/deceleration
- Artistic control over movement
- Complex trajectories

**Curves provide:**
- Smooth transitions
- Control points for designers
- Natural-feeling motion
- Flexible path creation

### Game Example
- Racing game tracks
- Character movement arcs
- Camera paths in cutscenes
- Projectile trajectories
- UI animations

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"({self.x:.1f}, {self.y:.1f})"


def lerp_vector(v1, v2, t):
    return Vector2(
        v1.x + (v2.x - v1.x) * t,
        v1.y + (v2.y - v1.y) * t
    )


# Problem: Linear path looks robotic
start = Vector2(0, 0)
end = Vector2(100, 100)

print("Linear movement (straight line):")
for t in [0, 0.25, 0.5, 0.75, 1.0]:
    pos = lerp_vector(start, end, t)
    print(f"  t={t}: {pos}")

# We want curves for:
# - Smooth turns
# - Interesting paths
# - Designer control
print("\n→ Curves allow smooth, interesting, controllable paths")
```

---

## Bezier Curves (베지어 곡선)

### Concept (개념)

**Bezier curves** are defined by control points. The curve:
- Starts at the first point
- Ends at the last point
- Is "pulled toward" the middle control points

**Types:**
- Linear (2 points): Straight line
- Quadratic (3 points): Simple curve
- Cubic (4 points): S-curves, complex paths

**Properties:**
- Always passes through start and end points
- Tangent at start points toward second point
- Tangent at end points from second-to-last point
- Always contained within convex hull of control points

### Game Example
- Font rendering
- Path editors
- Animation curves
- Vector graphics

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

    def __repr__(self):
        return f"({self.x:.1f}, {self.y:.1f})"


# Bezier curve concept visualization
print("Bezier Curve Control Points:")
print()
print("  P0 -------- P1")
print("   \\          /")
print("    \\   ?    /")
print("     \\      /")
print("      \\    /")
print("       \\  /")
print("        P2")
print()
print("The curve is 'pulled' toward control points")
print("but only passes through P0 and P2 (start/end)")
```

---

## Quadratic and Cubic Bezier (2차, 3차 베지어)

### Concept (개념)

**Quadratic Bezier (3 control points):**
```
B(t) = (1-t)²·P0 + 2(1-t)t·P1 + t²·P2
```

**Cubic Bezier (4 control points):**
```
B(t) = (1-t)³·P0 + 3(1-t)²t·P1 + 3(1-t)t²·P2 + t³·P3
```

**De Casteljau's algorithm** (recursive lerping):
- For quadratic: lerp between (P0,P1) and (P1,P2), then lerp results
- For cubic: same concept with more levels

### Game Example
- Quadratic: Simple curves, arcs
- Cubic: Complex paths, smooth animations, CSS transitions

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


def lerp_vector(v1, v2, t):
    return v1 * (1 - t) + v2 * t


def quadratic_bezier(p0, p1, p2, t):
    """
    Quadratic Bezier curve (3 control points).
    Using De Casteljau's algorithm.
    """
    # First level of lerping
    a = lerp_vector(p0, p1, t)
    b = lerp_vector(p1, p2, t)

    # Second level
    return lerp_vector(a, b, t)


def cubic_bezier(p0, p1, p2, p3, t):
    """
    Cubic Bezier curve (4 control points).
    Using De Casteljau's algorithm.
    """
    # First level
    a = lerp_vector(p0, p1, t)
    b = lerp_vector(p1, p2, t)
    c = lerp_vector(p2, p3, t)

    # Second level
    d = lerp_vector(a, b, t)
    e = lerp_vector(b, c, t)

    # Third level
    return lerp_vector(d, e, t)


# Quadratic Bezier example
print("Quadratic Bezier Curve:")
p0 = Vector2(0, 0)
p1 = Vector2(50, 100)  # Control point (curve bends toward this)
p2 = Vector2(100, 0)

for t in [0, 0.25, 0.5, 0.75, 1.0]:
    point = quadratic_bezier(p0, p1, p2, t)
    print(f"  t={t}: {point}")


# Cubic Bezier example
print("\nCubic Bezier Curve:")
p0 = Vector2(0, 0)
p1 = Vector2(30, 100)   # First control point
p2 = Vector2(70, 100)   # Second control point
p3 = Vector2(100, 0)

for t in [0, 0.25, 0.5, 0.75, 1.0]:
    point = cubic_bezier(p0, p1, p2, p3, t)
    print(f"  t={t}: {point}")


# Compare different control points
print("\nEffect of control points on cubic curve (t=0.5):")
curves = [
    ("Flat top", Vector2(0,0), Vector2(30,0), Vector2(70,0), Vector2(100,0)),
    ("Slight curve", Vector2(0,0), Vector2(30,50), Vector2(70,50), Vector2(100,0)),
    ("Sharp peak", Vector2(0,0), Vector2(30,150), Vector2(70,150), Vector2(100,0)),
    ("S-curve", Vector2(0,0), Vector2(30,100), Vector2(70,-100), Vector2(100,0)),
]

for name, a, b, c, d in curves:
    mid = cubic_bezier(a, b, c, d, 0.5)
    print(f"  {name}: midpoint = {mid}")
```

---

## Curved Path Movement (곡선 경로 이동)

### Concept (개념)

To move an object along a Bezier curve:
1. Increment t over time
2. Calculate position using Bezier formula
3. Update object position

**Challenge:** t doesn't correspond to distance. Moving t linearly doesn't give constant speed along the curve.

**Solutions:**
- Arc-length parameterization (complex but accurate)
- Approximate with many small segments
- Accept variable speed (often looks fine)

### Game Example
- Enemies following patrol paths
- Thrown objects
- Racing game track following

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
        return f"({self.x:.1f}, {self.y:.1f})"


def lerp_vector(v1, v2, t):
    return v1 * (1 - t) + v2 * t


def cubic_bezier(p0, p1, p2, p3, t):
    a = lerp_vector(p0, p1, t)
    b = lerp_vector(p1, p2, t)
    c = lerp_vector(p2, p3, t)
    d = lerp_vector(a, b, t)
    e = lerp_vector(b, c, t)
    return lerp_vector(d, e, t)


class CurveFollower:
    """Object that follows a Bezier curve"""

    def __init__(self, p0, p1, p2, p3, duration):
        self.p0 = p0
        self.p1 = p1
        self.p2 = p2
        self.p3 = p3
        self.duration = duration
        self.elapsed = 0
        self.position = Vector2(p0.x, p0.y)

    def update(self, dt):
        self.elapsed += dt
        t = min(self.elapsed / self.duration, 1.0)
        self.position = cubic_bezier(self.p0, self.p1, self.p2, self.p3, t)
        return t >= 1.0  # Returns True when complete

    def get_t(self):
        return min(self.elapsed / self.duration, 1.0)


# Simulate an enemy following a curved patrol path
patrol_path = CurveFollower(
    p0=Vector2(0, 100),
    p1=Vector2(100, 200),
    p2=Vector2(200, 0),
    p3=Vector2(300, 100),
    duration=3.0
)

print("Enemy patrol along Bezier curve:")
dt = 0.3
while True:
    t = patrol_path.get_t()
    print(f"  t={t:.2f}: position={patrol_path.position}")
    done = patrol_path.update(dt)
    if done:
        print(f"  t=1.00: position={patrol_path.position} (complete)")
        break


# Multiple connected curves (path)
class BezierPath:
    """Multiple connected Bezier curves"""

    def __init__(self):
        self.curves = []  # List of (p0, p1, p2, p3) tuples

    def add_curve(self, p0, p1, p2, p3):
        self.curves.append((p0, p1, p2, p3))

    def get_point(self, t):
        """Get point at t (0 to num_curves)"""
        if len(self.curves) == 0:
            return Vector2(0, 0)

        # Determine which curve segment
        num_curves = len(self.curves)
        curve_index = min(int(t), num_curves - 1)
        local_t = t - curve_index

        p0, p1, p2, p3 = self.curves[curve_index]
        return cubic_bezier(p0, p1, p2, p3, local_t)


# Create a path with multiple curves
path = BezierPath()
path.add_curve(
    Vector2(0, 0), Vector2(50, 50),
    Vector2(100, 50), Vector2(100, 0)
)
path.add_curve(
    Vector2(100, 0), Vector2(100, -50),
    Vector2(150, -50), Vector2(200, 0)
)

print("\nMulti-segment Bezier path:")
for i in range(21):
    t = i / 10.0  # 0 to 2 (two curves)
    point = path.get_point(t)
    print(f"  t={t:.1f}: {point}")
```

---

## Animation Curves: Ease In/Out (애니메이션 커브)

### Concept (개념)

**Easing functions** modify the t parameter to create different motion feels:

- **Linear**: Constant speed
- **Ease In**: Start slow, speed up
- **Ease Out**: Start fast, slow down
- **Ease In/Out**: Slow start and end, fast middle

**Common easing formulas:**
```python
# Quadratic
ease_in = t * t
ease_out = t * (2 - t)
ease_in_out = 2*t*t if t < 0.5 else -1 + (4 - 2*t) * t
```

### Game Example
- UI animations
- Camera movements
- Character landing
- Menu transitions

### Code Example
```python
import math

def ease_linear(t):
    """No easing - constant speed"""
    return t


def ease_in_quad(t):
    """Quadratic ease in - slow start"""
    return t * t


def ease_out_quad(t):
    """Quadratic ease out - slow end"""
    return t * (2 - t)


def ease_in_out_quad(t):
    """Quadratic ease in/out - slow start and end"""
    if t < 0.5:
        return 2 * t * t
    return -1 + (4 - 2 * t) * t


def ease_in_cubic(t):
    """Cubic ease in - slower start"""
    return t * t * t


def ease_out_cubic(t):
    """Cubic ease out - slower end"""
    return 1 - (1 - t) ** 3


def ease_in_out_cubic(t):
    """Cubic ease in/out"""
    if t < 0.5:
        return 4 * t * t * t
    return 1 - ((-2 * t + 2) ** 3) / 2


def ease_out_bounce(t):
    """Bouncy ease out"""
    n1, d1 = 7.5625, 2.75
    if t < 1 / d1:
        return n1 * t * t
    elif t < 2 / d1:
        t -= 1.5 / d1
        return n1 * t * t + 0.75
    elif t < 2.5 / d1:
        t -= 2.25 / d1
        return n1 * t * t + 0.9375
    else:
        t -= 2.625 / d1
        return n1 * t * t + 0.984375


def ease_out_elastic(t):
    """Elastic/springy ease out"""
    if t == 0 or t == 1:
        return t
    return 2 ** (-10 * t) * math.sin((t * 10 - 0.75) * (2 * math.pi) / 3) + 1


# Compare easing functions
print("Easing function comparison (position over time):")
print("t      Linear  EaseIn  EaseOut  InOut")
print("-" * 45)

for i in range(11):
    t = i / 10.0
    linear = ease_linear(t)
    ein = ease_in_quad(t)
    eout = ease_out_quad(t)
    einout = ease_in_out_quad(t)
    print(f"{t:.1f}    {linear:.2f}    {ein:.2f}    {eout:.2f}     {einout:.2f}")


# Using easing with movement
class EasedMovement:
    def __init__(self, start, end, duration, easing_func):
        self.start = start
        self.end = end
        self.duration = duration
        self.easing = easing_func
        self.elapsed = 0

    def update(self, dt):
        self.elapsed += dt
        raw_t = min(self.elapsed / self.duration, 1.0)
        eased_t = self.easing(raw_t)

        # Lerp with eased t
        position = self.start + (self.end - self.start) * eased_t
        return position, raw_t >= 1.0


# Compare movement feels
print("\nMovement comparison (x position):")
movements = [
    ("Linear", EasedMovement(0, 100, 1.0, ease_linear)),
    ("Ease In", EasedMovement(0, 100, 1.0, ease_in_quad)),
    ("Ease Out", EasedMovement(0, 100, 1.0, ease_out_quad)),
]

dt = 0.1
for name, mover in movements:
    positions = []
    for _ in range(11):
        pos, _ = mover.update(dt)
        positions.append(f"{pos:5.1f}")
        mover.elapsed = min(mover.elapsed, mover.duration)

    # Reset for demo
    print(f"  {name:8}: {' → '.join(positions[:6])}")
```

---

## Summary (요약)

| Curve Type | Korean | Points | Use Case |
|------------|--------|--------|----------|
| Linear | 선형 | 2 | Straight movement |
| Quadratic Bezier | 2차 베지어 | 3 | Simple curves |
| Cubic Bezier | 3차 베지어 | 4 | Complex paths, S-curves |
| Easing | 이징 | - | Animation timing |

**Key Easing Types:**
- Ease In: t² (slow start)
- Ease Out: 1-(1-t)² (slow end)
- Ease In/Out: Slow both ends

---

## Practice Problems (연습 문제)

1. Create a quadratic Bezier with P0=(0,0), P1=(50,100), P2=(100,0). What's the position at t=0.5?

2. Design control points for a cubic Bezier that creates an S-curve.

3. An animation uses ease_out_quad. At t=0.5 (raw), what's the eased t value? Is the object past or before the halfway point?

4. Create a bouncing ball animation using ease_out_bounce for landing.
