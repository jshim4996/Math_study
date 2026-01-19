# Chapter 07. Cross Product (외적)

> The cross product gives you perpendicular vectors and helps detect left/right relationships in 2D and 3D.

---

## What is the Cross Product? (외적이란?)

### Concept (개념)
The **cross product (외적)**, also called vector product, takes two vectors and returns a **new vector** that is perpendicular to both input vectors.

For 3D vectors:
```
A × B = (Ay*Bz - Az*By, Az*Bx - Ax*Bz, Ax*By - Ay*Bx)
```

Key properties:
- Result is perpendicular to both A and B
- Result magnitude = |A| * |B| * sin(θ)
- Not commutative: A × B = -(B × A)

In **2D**, we use a simplified version that returns a scalar:
```
A × B = Ax*By - Ay*Bx
```

This 2D cross product tells us the "signed area" and rotational relationship.

### Game Example
- Determining if a point is to the left or right of a direction
- Finding the normal vector of a surface
- Calculating torque (rotational force)

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def cross(self, other):
        """2D cross product (returns scalar)"""
        return self.x * other.y - self.y * other.x

    def __repr__(self):
        return f"({self.x}, {self.y})"


class Vector3:
    def __init__(self, x=0, y=0, z=0):
        self.x = x
        self.y = y
        self.z = z

    def cross(self, other):
        """3D cross product (returns vector)"""
        return Vector3(
            self.y * other.z - self.z * other.y,
            self.z * other.x - self.x * other.z,
            self.x * other.y - self.y * other.x
        )

    def __repr__(self):
        return f"({self.x}, {self.y}, {self.z})"


# 2D cross product
a = Vector2(1, 0)  # Right
b = Vector2(0, 1)  # Up

cross_2d = a.cross(b)
print(f"2D: {a} × {b} = {cross_2d}")  # 1 (positive = b is counterclockwise from a)

# 3D cross product
v1 = Vector3(1, 0, 0)  # X axis
v2 = Vector3(0, 1, 0)  # Y axis

cross_3d = v1.cross(v2)
print(f"3D: {v1} × {v2} = {cross_3d}")  # (0, 0, 1) - Z axis!
```

---

## The 3D Cross Product Formula (3D 외적 공식)

### Concept (개념)
For vectors A = (Ax, Ay, Az) and B = (Bx, By, Bz):

```
A × B = | i    j    k  |
        | Ax   Ay   Az |
        | Bx   By   Bz |

A × B = (Ay*Bz - Az*By)i - (Ax*Bz - Az*Bx)j + (Ax*By - Ay*Bx)k
```

Which gives us:
```
x = Ay*Bz - Az*By
y = Az*Bx - Ax*Bz  (note: this is negative of the middle term)
z = Ax*By - Ay*Bx
```

### Game Example
Finding the up vector when you know forward and right directions.

### Code Example
```python
class Vector3:
    def __init__(self, x=0, y=0, z=0):
        self.x = x
        self.y = y
        self.z = z

    def cross(self, other):
        return Vector3(
            self.y * other.z - self.z * other.y,
            self.z * other.x - self.x * other.z,
            self.x * other.y - self.y * other.x
        )

    def __repr__(self):
        return f"({self.x}, {self.y}, {self.z})"


# Standard basis vectors
right = Vector3(1, 0, 0)    # X axis
up = Vector3(0, 1, 0)       # Y axis
forward = Vector3(0, 0, 1)  # Z axis

# Cross products of basis vectors
print("Cross products of basis vectors:")
print(f"Right × Up = {right.cross(up)}")       # (0, 0, 1) = Forward
print(f"Up × Forward = {up.cross(forward)}")   # (1, 0, 0) = Right
print(f"Forward × Right = {forward.cross(right)}")  # (0, 1, 0) = Up

# Order matters! (anti-commutative)
print(f"\nUp × Right = {up.cross(right)}")     # (0, 0, -1) = -Forward
```

---

## The Result is Perpendicular (결과는 수직)

### Concept (개념)
The most important property of the cross product is that the result is **perpendicular (수직)** to both input vectors.

This is verified by the dot product being zero:
```
(A × B) · A = 0
(A × B) · B = 0
```

### Game Example
Creating a coordinate system from a single direction:
1. Start with forward direction
2. Cross with world up to get right
3. Cross right with forward to get local up

### Code Example
```python
import math

class Vector3:
    def __init__(self, x=0, y=0, z=0):
        self.x = x
        self.y = y
        self.z = z

    def cross(self, other):
        return Vector3(
            self.y * other.z - self.z * other.y,
            self.z * other.x - self.x * other.z,
            self.x * other.y - self.y * other.x
        )

    def dot(self, other):
        return self.x * other.x + self.y * other.y + self.z * other.z

    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2 + self.z**2)

    def normalized(self):
        mag = self.magnitude()
        if mag == 0:
            return Vector3(0, 0, 0)
        return Vector3(self.x / mag, self.y / mag, self.z / mag)

    def __repr__(self):
        return f"({self.x:.2f}, {self.y:.2f}, {self.z:.2f})"


# Verify perpendicularity
a = Vector3(1, 2, 3)
b = Vector3(4, 5, 6)

cross = a.cross(b)

print(f"A = {a}")
print(f"B = {b}")
print(f"A × B = {cross}")
print(f"(A × B) · A = {cross.dot(a):.6f}")  # Should be ~0
print(f"(A × B) · B = {cross.dot(b):.6f}")  # Should be ~0


# Build coordinate system from forward direction
def build_coordinate_system(forward):
    """Create right and up vectors from a forward direction"""
    forward = forward.normalized()

    # Use world up as reference (unless forward is parallel to up)
    world_up = Vector3(0, 1, 0)

    # Get right vector
    right = world_up.cross(forward).normalized()

    # If forward is parallel to world up, use different reference
    if right.magnitude() < 0.001:
        world_up = Vector3(0, 0, 1)
        right = world_up.cross(forward).normalized()

    # Get local up
    up = forward.cross(right).normalized()

    return forward, right, up


# Example: Camera looking at angle
camera_forward = Vector3(1, -0.5, 1)
forward, right, up = build_coordinate_system(camera_forward)

print(f"\nCamera coordinate system:")
print(f"Forward: {forward}")
print(f"Right: {right}")
print(f"Up: {up}")
```

---

## Left/Right Detection in 2D (2D에서 좌/우 판별)

### Concept (개념)
The 2D cross product sign tells us the rotational relationship:

```
A × B = Ax*By - Ay*Bx
```

- **Positive**: B is counterclockwise (left) of A
- **Zero**: A and B are parallel
- **Negative**: B is clockwise (right) of A

### Game Example
- Determining which way to turn to face a target
- Checking if a point is on the left or right side of a line
- Collision detection for convex polygons

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __sub__(self, other):
        return Vector2(self.x - other.x, self.y - other.y)

    def cross(self, other):
        return self.x * other.y - self.y * other.x

    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2)

    def normalized(self):
        mag = self.magnitude()
        if mag == 0:
            return Vector2(0, 0)
        return Vector2(self.x / mag, self.y / mag)

    def __repr__(self):
        return f"({self.x}, {self.y})"


def get_turn_direction(forward, target_direction):
    """Determine if we should turn left or right to face target"""
    cross = forward.cross(target_direction)

    if cross > 0.001:
        return "TURN LEFT"
    elif cross < -0.001:
        return "TURN RIGHT"
    else:
        return "ALREADY FACING"


# Character facing right, target in various directions
forward = Vector2(1, 0)

targets = [
    Vector2(1, 1),   # Upper right
    Vector2(-1, 1),  # Upper left
    Vector2(1, -1),  # Lower right
    Vector2(-1, 0),  # Behind (left)
]

print("Character facing RIGHT:")
for target in targets:
    direction = get_turn_direction(forward, target.normalized())
    print(f"  Target at {target}: {direction}")


# Point vs Line side detection
def point_side_of_line(line_start, line_end, point):
    """Determine which side of a line a point is on"""
    line_dir = line_end - line_start
    to_point = point - line_start

    cross = line_dir.cross(to_point)

    if cross > 0:
        return "LEFT"
    elif cross < 0:
        return "RIGHT"
    else:
        return "ON LINE"


# Line from (0,0) to (10,0) - horizontal line
line_a = Vector2(0, 0)
line_b = Vector2(10, 0)

test_points = [
    Vector2(5, 5),   # Above
    Vector2(5, -5),  # Below
    Vector2(5, 0),   # On line
]

print("\nPoint side of horizontal line:")
for p in test_points:
    side = point_side_of_line(line_a, line_b, p)
    print(f"  Point {p}: {side}")
```

---

## Normal Vectors (법선 벡터)

### Concept (개념)
A **normal vector (법선 벡터)** is perpendicular to a surface. Cross product is the primary way to calculate normals.

For a triangle with vertices A, B, C:
```
edge1 = B - A
edge2 = C - A
normal = edge1 × edge2
```

### Game Example
- Collision detection and response
- Lighting calculations (surface brightness)
- Determining which way a surface faces

### Code Example
```python
import math

class Vector3:
    def __init__(self, x=0, y=0, z=0):
        self.x = x
        self.y = y
        self.z = z

    def __sub__(self, other):
        return Vector3(self.x - other.x, self.y - other.y, self.z - other.z)

    def cross(self, other):
        return Vector3(
            self.y * other.z - self.z * other.y,
            self.z * other.x - self.x * other.z,
            self.x * other.y - self.y * other.x
        )

    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2 + self.z**2)

    def normalized(self):
        mag = self.magnitude()
        if mag == 0:
            return Vector3(0, 0, 0)
        return Vector3(self.x / mag, self.y / mag, self.z / mag)

    def __repr__(self):
        return f"({self.x:.2f}, {self.y:.2f}, {self.z:.2f})"


def calculate_triangle_normal(a, b, c):
    """Calculate the normal vector of a triangle"""
    edge1 = b - a
    edge2 = c - a
    normal = edge1.cross(edge2)
    return normal.normalized()


# Triangle on the XY plane (facing up in Z)
vertex_a = Vector3(0, 0, 0)
vertex_b = Vector3(1, 0, 0)
vertex_c = Vector3(0, 1, 0)

normal = calculate_triangle_normal(vertex_a, vertex_b, vertex_c)
print(f"Triangle normal: {normal}")  # (0, 0, 1) - pointing up


# Floor polygon
floor_a = Vector3(0, 0, 0)
floor_b = Vector3(10, 0, 0)
floor_c = Vector3(10, 0, 10)

floor_normal = calculate_triangle_normal(floor_a, floor_b, floor_c)
print(f"Floor normal: {floor_normal}")  # (0, 1, 0) - pointing up in Y


# Wall polygon
wall_a = Vector3(0, 0, 0)
wall_b = Vector3(10, 0, 0)
wall_c = Vector3(10, 5, 0)

wall_normal = calculate_triangle_normal(wall_a, wall_b, wall_c)
print(f"Wall normal: {wall_normal}")  # (0, 0, -1) - pointing in -Z
```

---

## Perpendicular Vectors in 2D (2D에서 수직 벡터)

### Concept (개념)
In 2D, getting a perpendicular vector is simple:
```
perpendicular of (x, y) = (-y, x) or (y, -x)
```

This is equivalent to rotating by 90 degrees.

### Game Example
- Creating wall normals in 2D
- Moving perpendicular to a direction (strafing)
- Calculating reflection vectors

### Code Example
```python
class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def perpendicular_left(self):
        """Get perpendicular vector (90° counterclockwise)"""
        return Vector2(-self.y, self.x)

    def perpendicular_right(self):
        """Get perpendicular vector (90° clockwise)"""
        return Vector2(self.y, -self.x)

    def __repr__(self):
        return f"({self.x}, {self.y})"


# Forward direction
forward = Vector2(1, 0)

# Get strafe directions
strafe_left = forward.perpendicular_left()
strafe_right = forward.perpendicular_right()

print(f"Forward: {forward}")
print(f"Strafe left: {strafe_left}")   # (0, 1)
print(f"Strafe right: {strafe_right}") # (0, -1)


# Wall normal calculation
def get_wall_normal(wall_start, wall_end, inside_point):
    """Get normal pointing toward inside_point"""
    wall_dir = Vector2(wall_end.x - wall_start.x, wall_end.y - wall_start.y)

    # Get perpendicular (could be either direction)
    normal = wall_dir.perpendicular_left()

    # Check if it points toward inside_point
    to_inside = Vector2(inside_point.x - wall_start.x, inside_point.y - wall_start.y)

    dot = normal.x * to_inside.x + normal.y * to_inside.y

    # If not, flip it
    if dot < 0:
        normal = wall_dir.perpendicular_right()

    return normal


# Room with walls
wall_start = Vector2(0, 0)
wall_end = Vector2(10, 0)
room_center = Vector2(5, 5)

wall_normal = get_wall_normal(wall_start, wall_end, room_center)
print(f"\nWall normal (toward room): {wall_normal}")
```

---

## Magnitude of Cross Product

### Concept (개념)
The magnitude of the cross product equals:
```
|A × B| = |A| * |B| * sin(θ)
```

This also equals the area of the parallelogram formed by A and B.

### Game Example
Calculating the area of triangles (half the parallelogram area).

### Code Example
```python
import math

class Vector3:
    def __init__(self, x=0, y=0, z=0):
        self.x = x
        self.y = y
        self.z = z

    def __sub__(self, other):
        return Vector3(self.x - other.x, self.y - other.y, self.z - other.z)

    def cross(self, other):
        return Vector3(
            self.y * other.z - self.z * other.y,
            self.z * other.x - self.x * other.z,
            self.x * other.y - self.y * other.x
        )

    def magnitude(self):
        return math.sqrt(self.x**2 + self.y**2 + self.z**2)


def triangle_area(a, b, c):
    """Calculate area of triangle using cross product"""
    edge1 = b - a
    edge2 = c - a
    cross = edge1.cross(edge2)
    # Area is half the parallelogram area
    return cross.magnitude() / 2


# Triangle with vertices
a = Vector3(0, 0, 0)
b = Vector3(4, 0, 0)
c = Vector3(0, 3, 0)

area = triangle_area(a, b, c)
print(f"Triangle area: {area}")  # 6.0 (half of 4*3=12)
```

---

## Summary (요약)

| Use Case | Korean | Formula/Result |
|----------|--------|----------------|
| 3D Cross Product | 3D 외적 | Vector perpendicular to both inputs |
| 2D Cross Product | 2D 외적 | Scalar: Ax*By - Ay*Bx |
| Left/Right Detection | 좌우 판별 | Positive = left, Negative = right |
| Surface Normal | 표면 법선 | edge1 × edge2 |
| 2D Perpendicular | 2D 수직 | (-y, x) or (y, -x) |
| Parallelogram Area | 평행사변형 면적 | \|A × B\| |

---

## Practice Problems (연습 문제)

1. Calculate (1, 2, 3) × (4, 5, 6).
2. Determine if point (5, 5) is left or right of line from (0, 0) to (10, 5).
3. A character faces direction (0.707, 0.707). What are its perpendicular directions?
4. Calculate the area of a triangle with vertices at (0,0,0), (3,0,0), (0,4,0).
