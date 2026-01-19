# Chapter 13. Transformation Matrices 3D (변환 행렬 3D)

> Extend 2D transforms to 3D: 4×4 matrices, homogeneous coordinates, and the critical TRS order.

---

## 3D Translation, Scale, Rotation Matrices (3D 변환 행렬)

### Concept (개념)

3D transforms extend 2D concepts with an additional axis (Z):

**Translation:**
```
[1  0  0  tx]
[0  1  0  ty]
[0  0  1  tz]
[0  0  0   1]
```

**Scale:**
```
[sx  0  0  0]
[ 0 sy  0  0]
[ 0  0 sz  0]
[ 0  0  0  1]
```

**Rotation** is more complex in 3D - you rotate around an axis (X, Y, or Z).

### Game Example
Every 3D game object uses 4×4 matrices. Unity's Transform component, Unreal's FTransform - they all use these matrices internally.

### Code Example
```python
import math

class Matrix4x4:
    def __init__(self, data=None):
        if data:
            self.data = data
        else:
            # Identity matrix
            self.data = [
                [1, 0, 0, 0],
                [0, 1, 0, 0],
                [0, 0, 1, 0],
                [0, 0, 0, 1]
            ]

    def __repr__(self):
        return "\n".join(["[" + " ".join(f"{x:7.2f}" for x in row) + "]" for row in self.data])

    def __mul__(self, other):
        result = Matrix4x4()
        for i in range(4):
            for j in range(4):
                result.data[i][j] = sum(
                    self.data[i][k] * other.data[k][j] for k in range(4)
                )
        return result

    def transform_point(self, x, y, z):
        """Transform a 3D point"""
        new_x = self.data[0][0]*x + self.data[0][1]*y + self.data[0][2]*z + self.data[0][3]
        new_y = self.data[1][0]*x + self.data[1][1]*y + self.data[1][2]*z + self.data[1][3]
        new_z = self.data[2][0]*x + self.data[2][1]*y + self.data[2][2]*z + self.data[2][3]
        return (new_x, new_y, new_z)

    @staticmethod
    def translation(tx, ty, tz):
        return Matrix4x4([
            [1, 0, 0, tx],
            [0, 1, 0, ty],
            [0, 0, 1, tz],
            [0, 0, 0, 1]
        ])

    @staticmethod
    def scale(sx, sy, sz):
        return Matrix4x4([
            [sx, 0,  0,  0],
            [0,  sy, 0,  0],
            [0,  0,  sz, 0],
            [0,  0,  0,  1]
        ])


# Translation
T = Matrix4x4.translation(10, 20, 30)
print("Translation matrix (10, 20, 30):")
print(T)

point = (1, 2, 3)
print(f"\nPoint {point} translated: {T.transform_point(*point)}")

# Scale
S = Matrix4x4.scale(2, 2, 2)
print("\nScale matrix (2x uniform):")
print(S)
print(f"Point {point} scaled: {S.transform_point(*point)}")
```

---

## 3D Rotation Matrices (3D 회전 행렬)

### Concept (개념)

In 3D, rotation happens around an **axis**:

**Rotation around X-axis:**
```
[1    0       0    0]
[0  cos(θ) -sin(θ) 0]
[0  sin(θ)  cos(θ) 0]
[0    0       0    1]
```

**Rotation around Y-axis:**
```
[ cos(θ)  0  sin(θ)  0]
[   0     1    0     0]
[-sin(θ)  0  cos(θ)  0]
[   0     0    0     1]
```

**Rotation around Z-axis:**
```
[cos(θ) -sin(θ)  0  0]
[sin(θ)  cos(θ)  0  0]
[  0       0     1  0]
[  0       0     0  1]
```

### Game Example
- X rotation: Nodding head (pitch)
- Y rotation: Shaking head (yaw)
- Z rotation: Tilting head (roll)

### Code Example
```python
import math

class Matrix4x4:
    def __init__(self, data=None):
        if data:
            self.data = data
        else:
            self.data = [[1,0,0,0], [0,1,0,0], [0,0,1,0], [0,0,0,1]]

    def __repr__(self):
        return "\n".join(["[" + " ".join(f"{x:7.3f}" for x in row) + "]" for row in self.data])

    def __mul__(self, other):
        result = Matrix4x4()
        for i in range(4):
            for j in range(4):
                result.data[i][j] = sum(self.data[i][k] * other.data[k][j] for k in range(4))
        return result

    def transform_point(self, x, y, z):
        new_x = self.data[0][0]*x + self.data[0][1]*y + self.data[0][2]*z + self.data[0][3]
        new_y = self.data[1][0]*x + self.data[1][1]*y + self.data[1][2]*z + self.data[1][3]
        new_z = self.data[2][0]*x + self.data[2][1]*y + self.data[2][2]*z + self.data[2][3]
        return (new_x, new_y, new_z)

    @staticmethod
    def rotation_x(degrees):
        """Rotation around X axis"""
        rad = math.radians(degrees)
        c, s = math.cos(rad), math.sin(rad)
        return Matrix4x4([
            [1,  0,  0, 0],
            [0,  c, -s, 0],
            [0,  s,  c, 0],
            [0,  0,  0, 1]
        ])

    @staticmethod
    def rotation_y(degrees):
        """Rotation around Y axis"""
        rad = math.radians(degrees)
        c, s = math.cos(rad), math.sin(rad)
        return Matrix4x4([
            [ c, 0, s, 0],
            [ 0, 1, 0, 0],
            [-s, 0, c, 0],
            [ 0, 0, 0, 1]
        ])

    @staticmethod
    def rotation_z(degrees):
        """Rotation around Z axis"""
        rad = math.radians(degrees)
        c, s = math.cos(rad), math.sin(rad)
        return Matrix4x4([
            [c, -s, 0, 0],
            [s,  c, 0, 0],
            [0,  0, 1, 0],
            [0,  0, 0, 1]
        ])


# Point along X axis
point = (1, 0, 0)

# Rotate around each axis
print(f"Original point: {point}")

Rx = Matrix4x4.rotation_x(90)
print(f"\nRotated 90° around X: {Rx.transform_point(*point)}")  # X stays, YZ rotate

Ry = Matrix4x4.rotation_y(90)
print(f"Rotated 90° around Y: {Ry.transform_point(*point)}")  # (0, 0, -1)

Rz = Matrix4x4.rotation_z(90)
print(f"Rotated 90° around Z: {Rz.transform_point(*point)}")  # (0, 1, 0)

# Point along Y axis
point_y = (0, 1, 0)
print(f"\nPoint {point_y} rotated 90° around X: {Rx.transform_point(*point_y)}")  # (0, 0, 1)
```

---

## Homogeneous Coordinates (동차 좌표계)

### Concept (개념)

**Homogeneous coordinates** add a 4th component (w) to 3D points:
- Point: (x, y, z) → (x, y, z, 1)
- Vector/Direction: (x, y, z) → (x, y, z, 0)

**Why?**
- w=1: Translation affects the point
- w=0: Translation doesn't affect it (useful for direction vectors)

To convert back: (x, y, z, w) → (x/w, y/w, z/w)

### Game Example
Light direction vectors shouldn't be affected by translation - they use w=0. Light positions use w=1.

### Code Example
```python
class Vector4:
    def __init__(self, x, y, z, w):
        self.x = x
        self.y = y
        self.z = z
        self.w = w

    def to_3d(self):
        """Convert homogeneous to 3D coordinates"""
        if self.w == 0:
            return (self.x, self.y, self.z)  # Direction vector
        return (self.x / self.w, self.y / self.w, self.z / self.w)

    def __repr__(self):
        return f"({self.x:.2f}, {self.y:.2f}, {self.z:.2f}, {self.w:.2f})"


class Matrix4x4:
    def __init__(self, data=None):
        self.data = data if data else [[1,0,0,0], [0,1,0,0], [0,0,1,0], [0,0,0,1]]

    def transform(self, v):
        """Transform a Vector4"""
        x = self.data[0][0]*v.x + self.data[0][1]*v.y + self.data[0][2]*v.z + self.data[0][3]*v.w
        y = self.data[1][0]*v.x + self.data[1][1]*v.y + self.data[1][2]*v.z + self.data[1][3]*v.w
        z = self.data[2][0]*v.x + self.data[2][1]*v.y + self.data[2][2]*v.z + self.data[2][3]*v.w
        w = self.data[3][0]*v.x + self.data[3][1]*v.y + self.data[3][2]*v.z + self.data[3][3]*v.w
        return Vector4(x, y, z, w)

    @staticmethod
    def translation(tx, ty, tz):
        return Matrix4x4([[1,0,0,tx], [0,1,0,ty], [0,0,1,tz], [0,0,0,1]])


T = Matrix4x4.translation(100, 200, 300)

# Point (w=1) - affected by translation
point = Vector4(1, 2, 3, 1)
print(f"Point {point}")
transformed_point = T.transform(point)
print(f"After translation: {transformed_point}")
print(f"As 3D: {transformed_point.to_3d()}")  # (101, 202, 303)

# Direction (w=0) - NOT affected by translation
direction = Vector4(1, 0, 0, 0)
print(f"\nDirection {direction}")
transformed_dir = T.transform(direction)
print(f"After translation: {transformed_dir}")  # Still (1, 0, 0, 0)
print(f"As 3D: {transformed_dir.to_3d()}")
```

---

## 4×4 Transformation Matrix (4×4 변환 행렬)

### Concept (개념)

A 4×4 matrix structure:

```
[R R R T]   R = Rotation/Scale (3×3)
[R R R T]   T = Translation
[R R R T]
[0 0 0 1]   Bottom row is always [0 0 0 1] for affine transforms
```

You can extract components:
- **Position**: Column 3 (indices [0][3], [1][3], [2][3])
- **Right vector**: Column 0
- **Up vector**: Column 1
- **Forward vector**: Column 2

### Game Example
Game engines store object transforms as 4×4 matrices. You can read position directly from the matrix.

### Code Example
```python
import math

class Matrix4x4:
    def __init__(self, data=None):
        self.data = data if data else [[1,0,0,0], [0,1,0,0], [0,0,1,0], [0,0,0,1]]

    def __repr__(self):
        return "\n".join(["[" + " ".join(f"{x:7.2f}" for x in row) + "]" for row in self.data])

    def __mul__(self, other):
        result = Matrix4x4()
        for i in range(4):
            for j in range(4):
                result.data[i][j] = sum(self.data[i][k] * other.data[k][j] for k in range(4))
        return result

    def get_position(self):
        """Extract translation from matrix"""
        return (self.data[0][3], self.data[1][3], self.data[2][3])

    def get_right(self):
        """Get right direction (X axis)"""
        return (self.data[0][0], self.data[1][0], self.data[2][0])

    def get_up(self):
        """Get up direction (Y axis)"""
        return (self.data[0][1], self.data[1][1], self.data[2][1])

    def get_forward(self):
        """Get forward direction (Z axis)"""
        return (self.data[0][2], self.data[1][2], self.data[2][2])

    @staticmethod
    def translation(tx, ty, tz):
        return Matrix4x4([[1,0,0,tx], [0,1,0,ty], [0,0,1,tz], [0,0,0,1]])

    @staticmethod
    def rotation_y(degrees):
        rad = math.radians(degrees)
        c, s = math.cos(rad), math.sin(rad)
        return Matrix4x4([[c,0,s,0], [0,1,0,0], [-s,0,c,0], [0,0,0,1]])

    @staticmethod
    def scale(sx, sy, sz):
        return Matrix4x4([[sx,0,0,0], [0,sy,0,0], [0,0,sz,0], [0,0,0,1]])


# Build a transform: position (10, 5, 20), rotated 45° around Y
T = Matrix4x4.translation(10, 5, 20)
R = Matrix4x4.rotation_y(45)
S = Matrix4x4.scale(1, 1, 1)

# TRS order
transform = T * R * S

print("Combined transform matrix:")
print(transform)

print(f"\nExtracted position: {transform.get_position()}")
print(f"Right vector: {transform.get_right()}")
print(f"Up vector: {transform.get_up()}")
print(f"Forward vector: {transform.get_forward()}")
```

---

## Transformation Order: TRS (변환 순서의 중요성)

### Concept (개념)

The standard order for combining transforms is **TRS** (Translation × Rotation × Scale):

```
FinalMatrix = Translation × Rotation × Scale
```

When applied to a point (reading right to left):
1. **Scale** first - resize the object
2. **Rotate** - orient the object
3. **Translate** - position the object

**Why this order?**
- Scale and rotate happen at origin
- Then move to final position
- Different order = different result!

### Game Example
Unity applies transforms in SRT order internally (Scale → Rotate → Translate), which means you construct the matrix as T × R × S.

### Code Example
```python
import math

class Matrix4x4:
    def __init__(self, data=None):
        self.data = data if data else [[1,0,0,0], [0,1,0,0], [0,0,1,0], [0,0,0,1]]

    def __repr__(self):
        return "\n".join(["[" + " ".join(f"{x:7.2f}" for x in row) + "]" for row in self.data])

    def __mul__(self, other):
        result = Matrix4x4()
        for i in range(4):
            for j in range(4):
                result.data[i][j] = sum(self.data[i][k] * other.data[k][j] for k in range(4))
        return result

    def transform_point(self, x, y, z):
        new_x = self.data[0][0]*x + self.data[0][1]*y + self.data[0][2]*z + self.data[0][3]
        new_y = self.data[1][0]*x + self.data[1][1]*y + self.data[1][2]*z + self.data[1][3]
        new_z = self.data[2][0]*x + self.data[2][1]*y + self.data[2][2]*z + self.data[2][3]
        return (new_x, new_y, new_z)

    @staticmethod
    def translation(tx, ty, tz):
        return Matrix4x4([[1,0,0,tx], [0,1,0,ty], [0,0,1,tz], [0,0,0,1]])

    @staticmethod
    def rotation_z(degrees):
        rad = math.radians(degrees)
        c, s = math.cos(rad), math.sin(rad)
        return Matrix4x4([[c,-s,0,0], [s,c,0,0], [0,0,1,0], [0,0,0,1]])

    @staticmethod
    def scale(sx, sy, sz):
        return Matrix4x4([[sx,0,0,0], [0,sy,0,0], [0,0,sz,0], [0,0,0,1]])


# Test point
point = (1, 0, 0)

T = Matrix4x4.translation(10, 0, 0)
R = Matrix4x4.rotation_z(90)
S = Matrix4x4.scale(2, 2, 2)

# Correct order: TRS (scale → rotate → translate)
TRS = T * R * S
print("TRS order (correct for typical use):")
print(f"  Point {point} → {TRS.transform_point(*point)}")
# Scale (1,0,0) → (2,0,0)
# Rotate 90° → (0,2,0)
# Translate → (10,2,0)

# Different order: TSR
TSR = T * S * R
print("\nTSR order:")
print(f"  Point {point} → {TSR.transform_point(*point)}")

# Different order: RTS
RTS = R * T * S
print("\nRTS order:")
print(f"  Point {point} → {RTS.transform_point(*point)}")

# Different order: SRT
SRT = S * R * T
print("\nSRT order:")
print(f"  Point {point} → {SRT.transform_point(*point)}")

print("\n--- Practical Example ---")
# Character at (100, 0, 0), rotated 45°, scaled 0.5x
char_pos = Matrix4x4.translation(100, 0, 0)
char_rot = Matrix4x4.rotation_z(45)
char_scale = Matrix4x4.scale(0.5, 0.5, 0.5)

char_transform = char_pos * char_rot * char_scale

# Transform a point on the character (e.g., hand at local position (2, 0, 0))
hand_local = (2, 0, 0)
hand_world = char_transform.transform_point(*hand_local)
print(f"Character hand local {hand_local}")
print(f"Character hand world: ({hand_world[0]:.2f}, {hand_world[1]:.2f}, {hand_world[2]:.2f})")
```

---

## Euler Angles and Gimbal Lock (오일러 각과 짐벌락)

### Concept (개념)

**Euler angles** represent 3D rotation as three separate rotations around X, Y, Z axes (pitch, yaw, roll).

**Gimbal lock** occurs when two rotation axes align, losing one degree of freedom. This happens when pitch = ±90°.

**Solutions:**
- Use quaternions for smooth interpolation
- Avoid 90° pitch angles
- Be aware of the limitation

### Game Example
FPS camera: yaw (left/right) and pitch (up/down). Pitch is typically clamped to avoid gimbal lock at straight up/down.

### Code Example
```python
import math

class Matrix4x4:
    def __init__(self, data=None):
        self.data = data if data else [[1,0,0,0], [0,1,0,0], [0,0,1,0], [0,0,0,1]]

    def __mul__(self, other):
        result = Matrix4x4()
        for i in range(4):
            for j in range(4):
                result.data[i][j] = sum(self.data[i][k] * other.data[k][j] for k in range(4))
        return result

    def transform_point(self, x, y, z):
        new_x = self.data[0][0]*x + self.data[0][1]*y + self.data[0][2]*z + self.data[0][3]
        new_y = self.data[1][0]*x + self.data[1][1]*y + self.data[1][2]*z + self.data[1][3]
        new_z = self.data[2][0]*x + self.data[2][1]*y + self.data[2][2]*z + self.data[2][3]
        return (new_x, new_y, new_z)

    @staticmethod
    def rotation_x(degrees):
        rad = math.radians(degrees)
        c, s = math.cos(rad), math.sin(rad)
        return Matrix4x4([[1,0,0,0], [0,c,-s,0], [0,s,c,0], [0,0,0,1]])

    @staticmethod
    def rotation_y(degrees):
        rad = math.radians(degrees)
        c, s = math.cos(rad), math.sin(rad)
        return Matrix4x4([[c,0,s,0], [0,1,0,0], [-s,0,c,0], [0,0,0,1]])

    @staticmethod
    def rotation_z(degrees):
        rad = math.radians(degrees)
        c, s = math.cos(rad), math.sin(rad)
        return Matrix4x4([[c,-s,0,0], [s,c,0,0], [0,0,1,0], [0,0,0,1]])

    @staticmethod
    def from_euler(pitch, yaw, roll):
        """Create rotation matrix from Euler angles (in degrees)"""
        # Common order: Y (yaw) → X (pitch) → Z (roll)
        Ry = Matrix4x4.rotation_y(yaw)
        Rx = Matrix4x4.rotation_x(pitch)
        Rz = Matrix4x4.rotation_z(roll)
        return Ry * Rx * Rz


# FPS camera example
class FPSCamera:
    def __init__(self):
        self.yaw = 0      # Left/right rotation
        self.pitch = 0    # Up/down rotation
        self.pitch_limit = 89  # Prevent gimbal lock

    def rotate(self, delta_yaw, delta_pitch):
        self.yaw += delta_yaw
        self.pitch += delta_pitch
        # Clamp pitch to avoid gimbal lock
        self.pitch = max(-self.pitch_limit, min(self.pitch_limit, self.pitch))

    def get_rotation_matrix(self):
        return Matrix4x4.from_euler(self.pitch, self.yaw, 0)

    def get_forward(self):
        """Get forward direction vector"""
        # Forward is -Z in most conventions
        matrix = self.get_rotation_matrix()
        forward = matrix.transform_point(0, 0, -1)
        return forward


camera = FPSCamera()
print("FPS Camera simulation:")
print(f"Initial forward: {camera.get_forward()}")

camera.rotate(45, 0)  # Look 45° right
print(f"After 45° yaw: {camera.get_forward()}")

camera.rotate(0, 30)  # Look 30° up
print(f"After 30° pitch: {camera.get_forward()}")

# Try to look straight up
camera.rotate(0, 60)  # Would be 90° total, but clamped
print(f"After attempting +60° pitch: pitch is {camera.pitch}° (clamped)")
```

---

## Summary (요약)

| Concept | Korean | Key Point |
|---------|--------|-----------|
| Translation 3D | 3D 이동 | Add to position |
| Scale 3D | 3D 크기 | Multiply each axis |
| Rotation 3D | 3D 회전 | Around X, Y, or Z axis |
| Homogeneous | 동차 좌표 | w=1 for points, w=0 for directions |
| 4×4 Matrix | 4×4 행렬 | Standard for 3D transforms |
| TRS Order | TRS 순서 | Scale → Rotate → Translate |
| Euler Angles | 오일러 각 | Pitch, Yaw, Roll |
| Gimbal Lock | 짐벌락 | Lose axis at ±90° pitch |

---

## Practice Problems (연습 문제)

1. Create a 4×4 matrix for an object at position (5, 10, 15), scaled 2x uniformly, rotated 90° around the Y axis.

2. A direction vector (0, 0, 1) is transformed by a translation matrix. What is the result? Why?

3. Extract the forward direction from a rotation matrix that represents 45° rotation around Y axis.

4. Why do FPS games clamp the pitch angle? What would happen without clamping?
