# Chapter 14. Coordinate Spaces (좌표 공간)

> Understanding how objects move between local, world, view, and screen coordinates.

---

## Local vs World Coordinates (로컬 좌표계 vs 월드 좌표계)

### Concept (개념)

**Local (Object) Space:**
- Coordinates relative to the object itself
- Origin is usually the object's center or pivot
- Used when modeling: artist creates mesh around origin

**World Space:**
- Global coordinate system for the entire scene
- All objects share this common reference
- Used for gameplay: positions, distances, collisions

**Transformation:** Local → World using the object's transformation matrix.

### Game Example
A car's wheel has local position (1, 0, 2) relative to the car. The car is at world position (100, 0, 50). The wheel's world position combines both.

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
    def translation(tx, ty, tz):
        return Matrix4x4([[1,0,0,tx], [0,1,0,ty], [0,0,1,tz], [0,0,0,1]])

    @staticmethod
    def rotation_y(degrees):
        rad = math.radians(degrees)
        c, s = math.cos(rad), math.sin(rad)
        return Matrix4x4([[c,0,s,0], [0,1,0,0], [-s,0,c,0], [0,0,0,1]])


class GameObject:
    def __init__(self, name):
        self.name = name
        self.local_position = (0, 0, 0)
        self.rotation_y = 0
        self.scale = (1, 1, 1)

    def get_world_matrix(self):
        T = Matrix4x4.translation(*self.local_position)
        R = Matrix4x4.rotation_y(self.rotation_y)
        return T * R

    def local_to_world(self, local_point):
        """Convert a local point to world coordinates"""
        return self.get_world_matrix().transform_point(*local_point)


# Car in world space
car = GameObject("Car")
car.local_position = (100, 0, 50)  # World position
car.rotation_y = 45  # Rotated 45 degrees

# Wheel positions in car's local space
wheel_positions_local = [
    ("Front-Left", (-1.5, 0, 2)),
    ("Front-Right", (1.5, 0, 2)),
    ("Back-Left", (-1.5, 0, -2)),
    ("Back-Right", (1.5, 0, -2)),
]

print(f"Car position: {car.local_position}")
print(f"Car rotation: {car.rotation_y}°")
print("\nWheel positions:")
for name, local_pos in wheel_positions_local:
    world_pos = car.local_to_world(local_pos)
    print(f"  {name}:")
    print(f"    Local: {local_pos}")
    print(f"    World: ({world_pos[0]:.2f}, {world_pos[1]:.2f}, {world_pos[2]:.2f})")
```

---

## Model, View, Projection Transforms (모델, 뷰, 투영 변환)

### Concept (개념)

The graphics pipeline transforms vertices through multiple coordinate spaces:

1. **Model Transform (M):** Local Space → World Space
   - Applies object's position, rotation, scale

2. **View Transform (V):** World Space → View/Camera Space
   - Makes camera the origin, looking down -Z
   - Inverse of camera's world transform

3. **Projection Transform (P):** View Space → Clip Space
   - Perspective: objects farther away appear smaller
   - Orthographic: no perspective (2D games, UI)

**Final:** Clip Space → NDC → Screen Space (handled by GPU)

### Game Example
Every vertex in a 3D game goes through MVP transformation before appearing on screen.

### Code Example
```python
import math

class Matrix4x4:
    def __init__(self, data=None):
        self.data = data if data else [[1,0,0,0], [0,1,0,0], [0,0,1,0], [0,0,0,1]]

    def __repr__(self):
        return "\n".join(["[" + " ".join(f"{x:8.3f}" for x in row) + "]" for row in self.data])

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
        new_w = self.data[3][0]*x + self.data[3][1]*y + self.data[3][2]*z + self.data[3][3]
        return (new_x, new_y, new_z, new_w)

    @staticmethod
    def translation(tx, ty, tz):
        return Matrix4x4([[1,0,0,tx], [0,1,0,ty], [0,0,1,tz], [0,0,0,1]])

    @staticmethod
    def rotation_y(degrees):
        rad = math.radians(degrees)
        c, s = math.cos(rad), math.sin(rad)
        return Matrix4x4([[c,0,s,0], [0,1,0,0], [-s,0,c,0], [0,0,0,1]])

    @staticmethod
    def look_at(eye, target, up):
        """Create view matrix (simplified)"""
        # Calculate forward, right, up vectors
        fx, fy, fz = target[0]-eye[0], target[1]-eye[1], target[2]-eye[2]
        f_len = math.sqrt(fx*fx + fy*fy + fz*fz)
        fx, fy, fz = fx/f_len, fy/f_len, fz/f_len

        # Right = forward × up
        rx = fy * up[2] - fz * up[1]
        ry = fz * up[0] - fx * up[2]
        rz = fx * up[1] - fy * up[0]
        r_len = math.sqrt(rx*rx + ry*ry + rz*rz)
        rx, ry, rz = rx/r_len, ry/r_len, rz/r_len

        # Recalculate up = right × forward
        ux = ry * fz - rz * fy
        uy = rz * fx - rx * fz
        uz = rx * fy - ry * fx

        return Matrix4x4([
            [rx, ry, rz, -(rx*eye[0] + ry*eye[1] + rz*eye[2])],
            [ux, uy, uz, -(ux*eye[0] + uy*eye[1] + uz*eye[2])],
            [-fx, -fy, -fz, fx*eye[0] + fy*eye[1] + fz*eye[2]],
            [0, 0, 0, 1]
        ])

    @staticmethod
    def perspective(fov_degrees, aspect, near, far):
        """Create perspective projection matrix"""
        fov_rad = math.radians(fov_degrees)
        f = 1.0 / math.tan(fov_rad / 2)
        return Matrix4x4([
            [f/aspect, 0, 0, 0],
            [0, f, 0, 0],
            [0, 0, (far+near)/(near-far), (2*far*near)/(near-far)],
            [0, 0, -1, 0]
        ])


# Scene setup
# Object at world position
model_matrix = Matrix4x4.translation(0, 0, -5)  # 5 units in front

# Camera at (0, 2, 0), looking at origin
view_matrix = Matrix4x4.look_at(
    eye=(0, 2, 0),
    target=(0, 0, -5),
    up=(0, 1, 0)
)

# Perspective projection
projection_matrix = Matrix4x4.perspective(
    fov_degrees=60,
    aspect=16/9,
    near=0.1,
    far=100
)

# MVP matrix
mvp = projection_matrix * view_matrix * model_matrix

print("Model Matrix (object transform):")
print(model_matrix)
print("\n--- Vertex transformation pipeline ---")

# Transform a vertex
vertex_local = (1, 1, 0)  # Corner of an object
print(f"\n1. Local vertex: {vertex_local}")

# To world space
world = model_matrix.transform_point(*vertex_local)
print(f"2. World space: ({world[0]:.2f}, {world[1]:.2f}, {world[2]:.2f})")

# To view space
view = view_matrix.transform_point(world[0], world[1], world[2])
print(f"3. View space: ({view[0]:.2f}, {view[1]:.2f}, {view[2]:.2f})")

# To clip space (with perspective)
clip = projection_matrix.transform_point(view[0], view[1], view[2])
print(f"4. Clip space: ({clip[0]:.2f}, {clip[1]:.2f}, {clip[2]:.2f}, w={clip[3]:.2f})")

# To NDC (divide by w)
if clip[3] != 0:
    ndc = (clip[0]/clip[3], clip[1]/clip[3], clip[2]/clip[3])
    print(f"5. NDC: ({ndc[0]:.2f}, {ndc[1]:.2f}, {ndc[2]:.2f})")
```

---

## MVP Matrix Concept (MVP 행렬)

### Concept (개념)

Instead of transforming vertices three times, combine M, V, P into one matrix:

```
MVP = Projection × View × Model
```

Then transform each vertex once:
```
clipPosition = MVP × localPosition
```

This is more efficient for rendering many vertices.

### Game Example
Shaders receive the MVP matrix as a uniform and apply it to every vertex in a single operation.

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

    def transform(self, x, y, z, w=1):
        new_x = self.data[0][0]*x + self.data[0][1]*y + self.data[0][2]*z + self.data[0][3]*w
        new_y = self.data[1][0]*x + self.data[1][1]*y + self.data[1][2]*z + self.data[1][3]*w
        new_z = self.data[2][0]*x + self.data[2][1]*y + self.data[2][2]*z + self.data[2][3]*w
        new_w = self.data[3][0]*x + self.data[3][1]*y + self.data[3][2]*z + self.data[3][3]*w
        return (new_x, new_y, new_z, new_w)

    @staticmethod
    def translation(tx, ty, tz):
        return Matrix4x4([[1,0,0,tx], [0,1,0,ty], [0,0,1,tz], [0,0,0,1]])

    @staticmethod
    def identity():
        return Matrix4x4()


# Simulate rendering multiple objects efficiently
class Renderer:
    def __init__(self):
        self.view_matrix = Matrix4x4.identity()
        self.projection_matrix = Matrix4x4.identity()

    def set_camera(self, view, projection):
        self.view_matrix = view
        self.projection_matrix = projection
        # Pre-compute VP (same for all objects)
        self.vp_matrix = self.projection_matrix * self.view_matrix

    def render_object(self, model_matrix, vertices):
        """Render an object with its vertices"""
        # Compute MVP for this object
        mvp = self.vp_matrix * model_matrix

        transformed = []
        for v in vertices:
            clip = mvp.transform(v[0], v[1], v[2], 1)
            # Perspective divide
            if clip[3] != 0:
                ndc = (clip[0]/clip[3], clip[1]/clip[3], clip[2]/clip[3])
                transformed.append(ndc)

        return transformed


# Setup renderer
renderer = Renderer()
renderer.set_camera(
    view=Matrix4x4.translation(0, 0, 10),  # Simple: move world back
    projection=Matrix4x4.identity()  # Simplified (no perspective)
)

# Define a simple triangle
triangle_vertices = [
    (0, 1, 0),   # Top
    (-1, -1, 0), # Bottom-left
    (1, -1, 0),  # Bottom-right
]

# Render at different positions
positions = [
    (0, 0, 0),
    (3, 0, 0),
    (-3, 0, 0),
]

print("Rendering triangle at different world positions:")
for pos in positions:
    model = Matrix4x4.translation(*pos)
    result = renderer.render_object(model, triangle_vertices)
    print(f"\nPosition {pos}:")
    for i, v in enumerate(result):
        print(f"  Vertex {i}: ({v[0]:.2f}, {v[1]:.2f}, {v[2]:.2f})")
```

---

## Parent-Child Transforms: Hierarchy (부모-자식 변환)

### Concept (개념)

Objects can be organized in a **hierarchy**:
- Child's world transform = Parent's world transform × Child's local transform
- Moving parent moves all children
- Child's local transform is relative to parent

This creates a **scene graph** structure.

### Game Example
- Robot arm: shoulder → elbow → wrist → hand
- Car: body → wheels, doors
- Character: root → spine → head, arms, legs

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
    def translation(tx, ty, tz):
        return Matrix4x4([[1,0,0,tx], [0,1,0,ty], [0,0,1,tz], [0,0,0,1]])

    @staticmethod
    def rotation_z(degrees):
        rad = math.radians(degrees)
        c, s = math.cos(rad), math.sin(rad)
        return Matrix4x4([[c,-s,0,0], [s,c,0,0], [0,0,1,0], [0,0,0,1]])


class Transform:
    def __init__(self, name):
        self.name = name
        self.position = (0, 0, 0)
        self.rotation = 0  # Z rotation only for simplicity
        self.parent = None
        self.children = []

    def add_child(self, child):
        child.parent = self
        self.children.append(child)

    def get_local_matrix(self):
        T = Matrix4x4.translation(*self.position)
        R = Matrix4x4.rotation_z(self.rotation)
        return T * R

    def get_world_matrix(self):
        local = self.get_local_matrix()
        if self.parent:
            return self.parent.get_world_matrix() * local
        return local

    def get_world_position(self):
        world = self.get_world_matrix()
        return (world.data[0][3], world.data[1][3], world.data[2][3])


# Robot arm hierarchy
shoulder = Transform("Shoulder")
shoulder.position = (0, 5, 0)  # Shoulder height

elbow = Transform("Elbow")
elbow.position = (3, 0, 0)  # Elbow relative to shoulder
shoulder.add_child(elbow)

wrist = Transform("Wrist")
wrist.position = (2.5, 0, 0)  # Wrist relative to elbow
elbow.add_child(wrist)

hand = Transform("Hand")
hand.position = (1, 0, 0)  # Hand relative to wrist
wrist.add_child(hand)

print("Robot arm - Initial state:")
for part in [shoulder, elbow, wrist, hand]:
    pos = part.get_world_position()
    print(f"  {part.name}: world pos = ({pos[0]:.2f}, {pos[1]:.2f}, {pos[2]:.2f})")

# Rotate shoulder
print("\nAfter rotating shoulder 45°:")
shoulder.rotation = 45
for part in [shoulder, elbow, wrist, hand]:
    pos = part.get_world_position()
    print(f"  {part.name}: world pos = ({pos[0]:.2f}, {pos[1]:.2f}, {pos[2]:.2f})")

# Rotate elbow additionally
print("\nAfter also rotating elbow -30°:")
elbow.rotation = -30
for part in [shoulder, elbow, wrist, hand]:
    pos = part.get_world_position()
    print(f"  {part.name}: world pos = ({pos[0]:.2f}, {pos[1]:.2f}, {pos[2]:.2f})")
```

---

## Inverse Transform (역변환)

### Concept (개념)

Sometimes you need to convert **from world to local** space (inverse of local-to-world).

Uses:
- Check if a world point is inside an object's bounds (convert to local, compare)
- Camera: view matrix is inverse of camera's world transform
- AI: convert target position to local space for decision making

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
    def translation(tx, ty, tz):
        return Matrix4x4([[1,0,0,tx], [0,1,0,ty], [0,0,1,tz], [0,0,0,1]])

    @staticmethod
    def rotation_y(degrees):
        rad = math.radians(degrees)
        c, s = math.cos(rad), math.sin(rad)
        return Matrix4x4([[c,0,s,0], [0,1,0,0], [-s,0,c,0], [0,0,0,1]])

    @staticmethod
    def inverse_translation(tx, ty, tz):
        """Inverse of translation = translate by negative"""
        return Matrix4x4.translation(-tx, -ty, -tz)

    @staticmethod
    def inverse_rotation_y(degrees):
        """Inverse of rotation = rotate by negative"""
        return Matrix4x4.rotation_y(-degrees)


class GameObject:
    def __init__(self):
        self.position = (0, 0, 0)
        self.rotation_y = 0

    def get_world_matrix(self):
        T = Matrix4x4.translation(*self.position)
        R = Matrix4x4.rotation_y(self.rotation_y)
        return T * R

    def get_inverse_world_matrix(self):
        """Get matrix that transforms world to local space"""
        # Inverse of T*R is R^-1 * T^-1 (reverse order!)
        inv_R = Matrix4x4.inverse_rotation_y(self.rotation_y)
        inv_T = Matrix4x4.inverse_translation(*self.position)
        return inv_R * inv_T

    def world_to_local(self, world_point):
        """Convert world point to local coordinates"""
        inverse = self.get_inverse_world_matrix()
        return inverse.transform_point(*world_point)

    def local_to_world(self, local_point):
        """Convert local point to world coordinates"""
        return self.get_world_matrix().transform_point(*local_point)


# Test inverse transform
player = GameObject()
player.position = (100, 0, 50)
player.rotation_y = 30

# A point in world space
world_point = (110, 0, 60)

# Convert to player's local space
local_point = player.world_to_local(world_point)
print(f"World point: {world_point}")
print(f"In player's local space: ({local_point[0]:.2f}, {local_point[1]:.2f}, {local_point[2]:.2f})")

# Convert back to verify
back_to_world = player.local_to_world(local_point)
print(f"Back to world: ({back_to_world[0]:.2f}, {back_to_world[1]:.2f}, {back_to_world[2]:.2f})")

# Use case: Is point in front of player?
# In local space, positive Z is forward
print(f"\nIs point in front of player? {local_point[2] > 0}")
```

---

## Summary (요약)

| Space | Korean | Description |
|-------|--------|-------------|
| Local/Object | 로컬/오브젝트 | Relative to object origin |
| World | 월드 | Global scene coordinates |
| View/Camera | 뷰/카메라 | Relative to camera |
| Clip | 클립 | After projection, before divide |
| NDC | NDC | Normalized [-1, 1] |
| Screen | 스크린 | Pixel coordinates |

**Key Transforms:**
- Model (M): Local → World
- View (V): World → Camera
- Projection (P): Camera → Clip
- MVP combined for efficiency

---

## Practice Problems (연습 문제)

1. An object is at world position (10, 0, 5). A point in local space is (1, 2, 0). What's the world position of that point?

2. Why is the View matrix the inverse of the camera's world transform?

3. Create a hierarchy: Car → Wheel. If the car is at (50, 0, 0) rotated 90°, and the wheel is at local position (2, 0, 3), what's the wheel's world position?

4. When would you need to convert a world position to an object's local space?
