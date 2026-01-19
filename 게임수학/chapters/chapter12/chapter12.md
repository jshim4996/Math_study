# Chapter 12. Transformation Matrices 2D (변환 행렬 2D)

> Apply translation, rotation, and scale using matrices: the foundation of all game object positioning.

---

## What is Transformation (변환이란 무엇인가)

### Concept (개념)

A **transformation** changes an object's position, orientation, or size. The three fundamental transforms are:

1. **Translation** (이동): Move to a new position
2. **Rotation** (회전): Spin around a point
3. **Scale** (크기 변환): Resize larger or smaller

Using matrices for transforms allows us to:
- Combine multiple transforms into one matrix
- Apply the same transform to many points efficiently
- Easily reverse transforms (inverse matrix)

### Game Example
Every game object has a transform: position, rotation, scale. The rendering system uses transformation matrices to place objects in the world.

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"({self.x:.2f}, {self.y:.2f})"


# Without matrices (direct calculation)
def translate_point(point, tx, ty):
    return Vector2(point.x + tx, point.y + ty)

def rotate_point(point, degrees):
    rad = math.radians(degrees)
    c, s = math.cos(rad), math.sin(rad)
    return Vector2(
        point.x * c - point.y * s,
        point.x * s + point.y * c
    )

def scale_point(point, sx, sy):
    return Vector2(point.x * sx, point.y * sy)


# Example: Transform a point
p = Vector2(10, 0)
print(f"Original: {p}")
print(f"Translated by (5, 3): {translate_point(p, 5, 3)}")
print(f"Rotated by 90°: {rotate_point(p, 90)}")
print(f"Scaled by (2, 2): {scale_point(p, 2, 2)}")
```

---

## Translation (이동 변환)

### Concept (개념)

Translation moves points by adding offset values:
```
new_x = x + tx
new_y = y + ty
```

**Problem:** A 2×2 matrix cannot represent translation (only linear transforms).

**Solution:** Use **homogeneous coordinates** with a 3×3 matrix:
```
[1  0  tx]   [x]   [x + tx]
[0  1  ty] × [y] = [y + ty]
[0  0   1]   [1]   [  1   ]
```

The extra `1` in the vector enables translation.

### Game Example
Moving a character from position (100, 200) by velocity (5, -3) each frame.

### Code Example
```python
class Matrix3x3:
    def __init__(self, data=None):
        if data:
            self.data = data
        else:
            self.data = [[1,0,0], [0,1,0], [0,0,1]]  # Identity

    def __repr__(self):
        return "\n".join(["[" + " ".join(f"{x:7.2f}" for x in row) + "]" for row in self.data])

    def transform_point(self, x, y):
        """Transform a 2D point using this 3x3 matrix (homogeneous coordinates)"""
        # Point as column vector [x, y, 1]
        new_x = self.data[0][0] * x + self.data[0][1] * y + self.data[0][2]
        new_y = self.data[1][0] * x + self.data[1][1] * y + self.data[1][2]
        return (new_x, new_y)

    @staticmethod
    def translation(tx, ty):
        """Create a translation matrix"""
        return Matrix3x3([
            [1, 0, tx],
            [0, 1, ty],
            [0, 0, 1]
        ])


# Translate a point
T = Matrix3x3.translation(50, 30)
print("Translation matrix (50, 30):")
print(T)

point = (10, 20)
new_point = T.transform_point(*point)
print(f"\nOriginal point: {point}")
print(f"Translated point: {new_point}")  # (60, 50)

# Multiple points with same matrix
points = [(0, 0), (10, 0), (10, 10), (0, 10)]  # A square
print("\nTranslating a square:")
for p in points:
    new_p = T.transform_point(*p)
    print(f"  {p} → ({new_p[0]:.0f}, {new_p[1]:.0f})")
```

---

## Scale (크기 변환)

### Concept (개념)

Scaling multiplies coordinates by scale factors:
```
new_x = x * sx
new_y = y * sy
```

As a 3×3 matrix:
```
[sx  0  0]   [x]   [x * sx]
[ 0 sy  0] × [y] = [y * sy]
[ 0  0  1]   [1]   [  1   ]
```

**Scale types:**
- **Uniform scale**: sx = sy (preserves proportions)
- **Non-uniform scale**: sx ≠ sy (stretches/squashes)
- **Negative scale**: Reflects/mirrors

**Important:** Scaling is relative to the **origin** (0, 0). To scale around another point, translate first.

### Game Example
Zoom in/out of a map, resize a sprite, create a growing/shrinking effect.

### Code Example
```python
class Matrix3x3:
    def __init__(self, data=None):
        self.data = data if data else [[1,0,0], [0,1,0], [0,0,1]]

    def __repr__(self):
        return "\n".join(["[" + " ".join(f"{x:7.2f}" for x in row) + "]" for row in self.data])

    def transform_point(self, x, y):
        new_x = self.data[0][0] * x + self.data[0][1] * y + self.data[0][2]
        new_y = self.data[1][0] * x + self.data[1][1] * y + self.data[1][2]
        return (new_x, new_y)

    @staticmethod
    def scale(sx, sy):
        """Create a scale matrix"""
        return Matrix3x3([
            [sx, 0, 0],
            [0, sy, 0],
            [0, 0, 1]
        ])


# Uniform scale (2x)
S_uniform = Matrix3x3.scale(2, 2)
print("Uniform scale (2x):")
print(S_uniform)

point = (10, 5)
print(f"Point {point} → {S_uniform.transform_point(*point)}")  # (20, 10)

# Non-uniform scale
S_stretch = Matrix3x3.scale(3, 0.5)
print("\nNon-uniform scale (3x width, 0.5x height):")
print(f"Point {point} → {S_stretch.transform_point(*point)}")  # (30, 2.5)

# Negative scale (mirror)
S_mirror_x = Matrix3x3.scale(-1, 1)  # Mirror horizontally
print("\nMirror horizontally:")
print(f"Point {point} → {S_mirror_x.transform_point(*point)}")  # (-10, 5)

# Scale a rectangle
print("\nScaling a rectangle by 2x:")
rect = [(0, 0), (10, 0), (10, 5), (0, 5)]
S = Matrix3x3.scale(2, 2)
for p in rect:
    print(f"  {p} → {S.transform_point(*p)}")
```

---

## Rotation (회전 변환)

### Concept (개념)

Rotation around the origin by angle θ:
```
new_x = x * cos(θ) - y * sin(θ)
new_y = x * sin(θ) + y * cos(θ)
```

As a 3×3 matrix:
```
[cos(θ)  -sin(θ)  0]   [x]   [x*cos - y*sin]
[sin(θ)   cos(θ)  0] × [y] = [x*sin + y*cos]
[  0        0     1]   [1]   [      1      ]
```

**Important:** Rotation is around the **origin**. To rotate around another point:
1. Translate so the pivot is at origin
2. Rotate
3. Translate back

### Game Example
Rotating a spaceship, spinning a coin, turning a turret.

### Code Example
```python
import math

class Matrix3x3:
    def __init__(self, data=None):
        self.data = data if data else [[1,0,0], [0,1,0], [0,0,1]]

    def __repr__(self):
        return "\n".join(["[" + " ".join(f"{x:7.2f}" for x in row) + "]" for row in self.data])

    def transform_point(self, x, y):
        new_x = self.data[0][0] * x + self.data[0][1] * y + self.data[0][2]
        new_y = self.data[1][0] * x + self.data[1][1] * y + self.data[1][2]
        return (new_x, new_y)

    @staticmethod
    def rotation(degrees):
        """Create a rotation matrix (counter-clockwise)"""
        rad = math.radians(degrees)
        c, s = math.cos(rad), math.sin(rad)
        return Matrix3x3([
            [c, -s, 0],
            [s,  c, 0],
            [0,  0, 1]
        ])


# Rotate 90° counter-clockwise
R90 = Matrix3x3.rotation(90)
print("Rotation matrix (90°):")
print(R90)

point = (10, 0)
print(f"\nPoint {point} rotated 90°: {R90.transform_point(*point)}")  # (0, 10)

# Rotate by various angles
print("\nRotating point (1, 0):")
for angle in [0, 45, 90, 135, 180, 270]:
    R = Matrix3x3.rotation(angle)
    result = R.transform_point(1, 0)
    print(f"  {angle:3}°: ({result[0]:6.3f}, {result[1]:6.3f})")

# Rotate a square
print("\nRotating a square 45°:")
square = [(1, 1), (1, -1), (-1, -1), (-1, 1)]
R45 = Matrix3x3.rotation(45)
for p in square:
    new_p = R45.transform_point(*p)
    print(f"  {p} → ({new_p[0]:.3f}, {new_p[1]:.3f})")
```

---

## Combining Transformation Matrices (변환 행렬 조합)

### Concept (개념)

The power of matrices: **multiply matrices to combine transforms**.

```
Combined = Translation × Rotation × Scale
```

Apply to a point with one multiplication instead of three separate operations.

**Order matters!** Matrix multiplication is not commutative.
- Scale then Rotate ≠ Rotate then Scale
- Typically: Scale → Rotate → Translate (SRT order)

### Game Example
A game object has position, rotation, and scale. The engine combines them into one matrix for efficient rendering.

### Code Example
```python
import math

class Matrix3x3:
    def __init__(self, data=None):
        self.data = data if data else [[1,0,0], [0,1,0], [0,0,1]]

    def __repr__(self):
        return "\n".join(["[" + " ".join(f"{x:7.2f}" for x in row) + "]" for row in self.data])

    def __mul__(self, other):
        """Matrix multiplication"""
        result = Matrix3x3()
        for i in range(3):
            for j in range(3):
                result.data[i][j] = sum(
                    self.data[i][k] * other.data[k][j] for k in range(3)
                )
        return result

    def transform_point(self, x, y):
        new_x = self.data[0][0] * x + self.data[0][1] * y + self.data[0][2]
        new_y = self.data[1][0] * x + self.data[1][1] * y + self.data[1][2]
        return (new_x, new_y)

    @staticmethod
    def identity():
        return Matrix3x3([[1,0,0], [0,1,0], [0,0,1]])

    @staticmethod
    def translation(tx, ty):
        return Matrix3x3([[1, 0, tx], [0, 1, ty], [0, 0, 1]])

    @staticmethod
    def rotation(degrees):
        rad = math.radians(degrees)
        c, s = math.cos(rad), math.sin(rad)
        return Matrix3x3([[c, -s, 0], [s, c, 0], [0, 0, 1]])

    @staticmethod
    def scale(sx, sy):
        return Matrix3x3([[sx, 0, 0], [0, sy, 0], [0, 0, 1]])


# Combine transforms: Scale → Rotate → Translate
S = Matrix3x3.scale(2, 2)       # Double size
R = Matrix3x3.rotation(45)      # Rotate 45°
T = Matrix3x3.translation(100, 50)  # Move to (100, 50)

# Combined matrix (apply right to left: S first, then R, then T)
combined = T * R * S

print("Individual matrices:")
print("Scale (2x):")
print(S)
print("\nRotation (45°):")
print(R)
print("\nTranslation (100, 50):")
print(T)
print("\nCombined (T × R × S):")
print(combined)

# Transform a point
point = (10, 0)
print(f"\nTransforming point {point}:")

# Step by step
p1 = S.transform_point(*point)
print(f"  After scale: {p1}")

p2 = R.transform_point(*p1)
print(f"  After rotate: ({p2[0]:.2f}, {p2[1]:.2f})")

p3 = T.transform_point(*p2)
print(f"  After translate: ({p3[0]:.2f}, {p3[1]:.2f})")

# Using combined matrix (same result!)
p_combined = combined.transform_point(*point)
print(f"  Using combined matrix: ({p_combined[0]:.2f}, {p_combined[1]:.2f})")


# ORDER MATTERS demonstration
print("\n--- Order matters! ---")
point2 = (10, 0)

# Rotate then translate
RT = T * R
print(f"Rotate then translate {point2}: {RT.transform_point(*point2)}")

# Translate then rotate
TR = R * T
print(f"Translate then rotate {point2}: {TR.transform_point(*point2)}")
```

---

## Rotation Around a Point (특정 점을 중심으로 회전)

### Concept (개념)

To rotate around point P (not the origin):
1. Translate so P is at origin: T(-P)
2. Rotate: R(θ)
3. Translate back: T(P)

Combined: T(P) × R(θ) × T(-P)

### Game Example
Rotate a sprite around its center (not corner), swing a weapon around the character's hand position.

### Code Example
```python
import math

class Matrix3x3:
    def __init__(self, data=None):
        self.data = data if data else [[1,0,0], [0,1,0], [0,0,1]]

    def __mul__(self, other):
        result = Matrix3x3()
        for i in range(3):
            for j in range(3):
                result.data[i][j] = sum(
                    self.data[i][k] * other.data[k][j] for k in range(3)
                )
        return result

    def transform_point(self, x, y):
        new_x = self.data[0][0] * x + self.data[0][1] * y + self.data[0][2]
        new_y = self.data[1][0] * x + self.data[1][1] * y + self.data[1][2]
        return (new_x, new_y)

    @staticmethod
    def translation(tx, ty):
        return Matrix3x3([[1, 0, tx], [0, 1, ty], [0, 0, 1]])

    @staticmethod
    def rotation(degrees):
        rad = math.radians(degrees)
        c, s = math.cos(rad), math.sin(rad)
        return Matrix3x3([[c, -s, 0], [s, c, 0], [0, 0, 1]])

    @staticmethod
    def rotation_around(degrees, pivot_x, pivot_y):
        """Create a rotation matrix around a specific point"""
        T_to_origin = Matrix3x3.translation(-pivot_x, -pivot_y)
        R = Matrix3x3.rotation(degrees)
        T_back = Matrix3x3.translation(pivot_x, pivot_y)
        return T_back * R * T_to_origin


# Rotate around origin vs around a point
point = (20, 10)
pivot = (10, 10)

# Rotate around origin
R_origin = Matrix3x3.rotation(90)
result_origin = R_origin.transform_point(*point)
print(f"Point {point} rotated 90° around ORIGIN: ({result_origin[0]:.1f}, {result_origin[1]:.1f})")

# Rotate around pivot (10, 10)
R_pivot = Matrix3x3.rotation_around(90, *pivot)
result_pivot = R_pivot.transform_point(*point)
print(f"Point {point} rotated 90° around {pivot}: ({result_pivot[0]:.1f}, {result_pivot[1]:.1f})")

# Visualize: rotate corners of a square around its center
print("\nRotating square around its center:")
# Square with corners at (0,0), (20,0), (20,20), (0,20)
# Center is at (10, 10)
square = [(0, 0), (20, 0), (20, 20), (0, 20)]
center = (10, 10)
R_center = Matrix3x3.rotation_around(45, *center)

for corner in square:
    new_corner = R_center.transform_point(*corner)
    print(f"  {corner} → ({new_corner[0]:.2f}, {new_corner[1]:.2f})")
```

---

## Summary (요약)

| Transform | Korean | Matrix (3×3) |
|-----------|--------|--------------|
| Translation | 이동 | [1,0,tx], [0,1,ty], [0,0,1] |
| Scale | 크기 | [sx,0,0], [0,sy,0], [0,0,1] |
| Rotation | 회전 | [c,-s,0], [s,c,0], [0,0,1] |
| Combined | 조합 | T × R × S (right to left) |

**Key Points:**
- Use 3×3 matrices with homogeneous coordinates for 2D transforms
- Order matters: Scale → Rotate → Translate (SRT)
- To rotate around a point: translate to origin, rotate, translate back

---

## Practice Problems (연습 문제)

1. Create a transformation matrix that moves a point by (30, 20) and scales it by 1.5x.

2. A sprite is at position (100, 100). Create a matrix to rotate it 45° around its center, assuming the sprite is 50×50 pixels.

3. What's the difference in result between (Scale 2x → Rotate 90°) vs (Rotate 90° → Scale 2x)?

4. Create a "flip and move" transform: mirror horizontally and then translate by (200, 0).
