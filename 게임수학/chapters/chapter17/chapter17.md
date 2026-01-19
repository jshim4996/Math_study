# Chapter 17. Advanced Collision Detection (고급 충돌 감지)

> Beyond basic shapes: OBB, SAT, raycasting, and optimization techniques.

---

## OBB: Oriented Bounding Box Concept (OBB 개념)

### Concept (개념)

**OBB** (Oriented Bounding Box) is a rectangle that can rotate with the object.

Unlike AABB:
- Can rotate to any angle
- Tighter fit for rotated objects
- More expensive collision check

**Definition:**
- Center point
- Half-extents (half-width, half-height)
- Rotation angle (or local axes)

### Game Example
- Rotated sprites
- Cars, ships, planes
- Any object that rotates significantly

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

    def rotate(self, degrees):
        rad = math.radians(degrees)
        c, s = math.cos(rad), math.sin(rad)
        return Vector2(
            self.x * c - self.y * s,
            self.x * s + self.y * c
        )

    def __repr__(self):
        return f"({self.x:.2f}, {self.y:.2f})"


class OBB:
    def __init__(self, center_x, center_y, half_width, half_height, rotation_deg=0):
        self.center = Vector2(center_x, center_y)
        self.half_extents = Vector2(half_width, half_height)
        self.rotation = rotation_deg

    def get_axes(self):
        """Get local X and Y axes (unit vectors)"""
        rad = math.radians(self.rotation)
        c, s = math.cos(rad), math.sin(rad)
        return (
            Vector2(c, s),   # Local X axis
            Vector2(-s, c)   # Local Y axis
        )

    def get_corners(self):
        """Get the four corner points"""
        ax, ay = self.get_axes()

        # Half-extent vectors
        hx = ax * self.half_extents.x
        hy = ay * self.half_extents.y

        return [
            self.center + hx + hy,  # Top-right
            self.center - hx + hy,  # Top-left
            self.center - hx - hy,  # Bottom-left
            self.center + hx - hy,  # Bottom-right
        ]


# Create and visualize OBBs
box1 = OBB(100, 100, 40, 20, rotation_deg=0)
box2 = OBB(150, 100, 30, 15, rotation_deg=45)

print("OBB 1 (no rotation):")
print(f"  Center: {box1.center}")
print(f"  Half-extents: {box1.half_extents}")
print(f"  Corners: {box1.get_corners()}")

print("\nOBB 2 (45° rotation):")
print(f"  Center: {box2.center}")
print(f"  Axes: {box2.get_axes()}")
print(f"  Corners: {box2.get_corners()}")
```

---

## Separating Axis Theorem (SAT) Concept (분리 축 정리)

### Concept (개념)

**SAT** states: Two convex shapes do NOT collide if there exists an axis onto which their projections don't overlap.

**For OBB vs OBB:**
Test 4 axes (2 from each OBB):
1. OBB1's local X axis
2. OBB1's local Y axis
3. OBB2's local X axis
4. OBB2's local Y axis

If projections overlap on ALL axes → Collision
If projections separate on ANY axis → No collision

### Game Example
Precise collision for rotated rectangles (tanks, racing cars, etc.)

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


class OBB:
    def __init__(self, cx, cy, hw, hh, rotation=0):
        self.center = Vector2(cx, cy)
        self.half_extents = Vector2(hw, hh)
        self.rotation = rotation

    def get_axes(self):
        rad = math.radians(self.rotation)
        c, s = math.cos(rad), math.sin(rad)
        return [Vector2(c, s), Vector2(-s, c)]

    def get_corners(self):
        ax, ay = self.get_axes()
        hx = ax * self.half_extents.x
        hy = ay * self.half_extents.y
        return [
            self.center + hx + hy,
            self.center - hx + hy,
            self.center - hx - hy,
            self.center + hx - hy,
        ]


def project_polygon(corners, axis):
    """Project polygon corners onto axis, return min/max"""
    min_proj = float('inf')
    max_proj = float('-inf')

    for corner in corners:
        proj = corner.dot(axis)
        min_proj = min(min_proj, proj)
        max_proj = max(max_proj, proj)

    return min_proj, max_proj


def intervals_overlap(min1, max1, min2, max2):
    """Check if two 1D intervals overlap"""
    return max1 >= min2 and max2 >= min1


def obb_vs_obb(obb1, obb2):
    """SAT collision test for two OBBs"""
    corners1 = obb1.get_corners()
    corners2 = obb2.get_corners()

    # Get all axes to test (2 from each OBB)
    axes = obb1.get_axes() + obb2.get_axes()

    for axis in axes:
        # Project both OBBs onto this axis
        min1, max1 = project_polygon(corners1, axis)
        min2, max2 = project_polygon(corners2, axis)

        # If projections don't overlap, no collision
        if not intervals_overlap(min1, max1, min2, max2):
            return False

    # Overlap on all axes = collision
    return True


# Test OBB collision
box1 = OBB(100, 100, 40, 20, rotation=0)
box2 = OBB(160, 100, 30, 15, rotation=45)
box3 = OBB(250, 100, 25, 25, rotation=30)

print("OBB Collision Tests (SAT):")
print(f"Box1 vs Box2: {obb_vs_obb(box1, box2)}")  # Likely True
print(f"Box1 vs Box3: {obb_vs_obb(box1, box3)}")  # Likely False
print(f"Box2 vs Box3: {obb_vs_obb(box2, box3)}")  # Likely False

# Rotating collision test
print("\nRotating Box1 to test collision:")
for angle in range(0, 91, 15):
    box1.rotation = angle
    collision = obb_vs_obb(box1, box2)
    print(f"  Box1 at {angle}°: {'COLLISION' if collision else 'clear'}")
```

---

## Ray Casting (레이캐스팅)

### Concept (개념)

**Raycasting** shoots a ray (line) from a point in a direction and finds what it hits.

**Ray definition:**
- Origin point
- Direction (unit vector)
- Optional max distance

**Uses:**
- Line of sight
- Shooting/aiming
- Ground detection
- Mouse picking in 3D

### Game Example
- Bullet trajectory
- AI sight checks
- Click detection in 3D
- Laser beams

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


class Ray:
    def __init__(self, origin, direction):
        self.origin = origin
        self.direction = direction.normalized()

    def point_at(self, t):
        """Get point along ray at distance t"""
        return self.origin + self.direction * t


class Circle:
    def __init__(self, x, y, radius):
        self.center = Vector2(x, y)
        self.radius = radius


def ray_circle_intersection(ray, circle):
    """
    Returns distance to intersection, or None if no hit.
    Uses quadratic formula.
    """
    # Vector from ray origin to circle center
    oc = ray.origin - circle.center

    # Quadratic coefficients
    a = ray.direction.dot(ray.direction)  # Always 1 if normalized
    b = 2.0 * oc.dot(ray.direction)
    c = oc.dot(oc) - circle.radius * circle.radius

    discriminant = b*b - 4*a*c

    if discriminant < 0:
        return None  # No intersection

    # Find nearest intersection (smallest positive t)
    sqrt_d = math.sqrt(discriminant)
    t1 = (-b - sqrt_d) / (2*a)
    t2 = (-b + sqrt_d) / (2*a)

    # Return nearest positive t
    if t1 > 0:
        return t1
    if t2 > 0:
        return t2
    return None  # Both intersections behind ray


class AABB:
    def __init__(self, x, y, width, height):
        self.min_x = x
        self.min_y = y
        self.max_x = x + width
        self.max_y = y + height


def ray_aabb_intersection(ray, aabb):
    """
    Returns distance to intersection, or None if no hit.
    Uses slab method.
    """
    t_min = 0
    t_max = float('inf')

    # Check X slab
    if ray.direction.x != 0:
        t1 = (aabb.min_x - ray.origin.x) / ray.direction.x
        t2 = (aabb.max_x - ray.origin.x) / ray.direction.x
        if t1 > t2:
            t1, t2 = t2, t1
        t_min = max(t_min, t1)
        t_max = min(t_max, t2)
    elif ray.origin.x < aabb.min_x or ray.origin.x > aabb.max_x:
        return None

    # Check Y slab
    if ray.direction.y != 0:
        t1 = (aabb.min_y - ray.origin.y) / ray.direction.y
        t2 = (aabb.max_y - ray.origin.y) / ray.direction.y
        if t1 > t2:
            t1, t2 = t2, t1
        t_min = max(t_min, t1)
        t_max = min(t_max, t2)
    elif ray.origin.y < aabb.min_y or ray.origin.y > aabb.max_y:
        return None

    if t_max < t_min or t_max < 0:
        return None

    return t_min if t_min > 0 else t_max


# Shooting example
gun_pos = Vector2(0, 0)
aim_dir = Vector2(1, 0.5).normalized()  # Aiming right and up

ray = Ray(gun_pos, aim_dir)

# Targets
targets = [
    ("Enemy1", Circle(100, 40, 20)),
    ("Enemy2", Circle(150, 80, 25)),
    ("Wall", AABB(200, 0, 20, 150)),
]

print(f"Ray: origin={ray.origin}, direction=({ray.direction.x:.2f}, {ray.direction.y:.2f})")
print("\nRaycast results:")

hits = []
for name, obj in targets:
    if isinstance(obj, Circle):
        t = ray_circle_intersection(ray, obj)
    else:
        t = ray_aabb_intersection(ray, obj)

    if t is not None:
        hit_point = ray.point_at(t)
        hits.append((t, name, hit_point))
        print(f"  {name}: HIT at distance {t:.2f}, point=({hit_point.x:.1f}, {hit_point.y:.1f})")
    else:
        print(f"  {name}: miss")

# First hit
if hits:
    hits.sort(key=lambda x: x[0])
    print(f"\nFirst hit: {hits[0][1]} at distance {hits[0][0]:.2f}")
```

---

## Collision Detection Optimization (충돌 감지 최적화)

### Concept (개념)

Checking every object against every other is O(n²). Optimizations:

**1. Spatial Partitioning:**
- Grid: Divide world into cells, only check objects in same/nearby cells
- Quadtree: Adaptive subdivision for non-uniform distribution
- BVH (Bounding Volume Hierarchy): Tree of bounding boxes

**2. Broad Phase + Narrow Phase:**
- Broad: Quick AABB check to filter pairs
- Narrow: Precise check only for broad-phase hits

**3. Temporal Coherence:**
- Objects don't move much frame-to-frame
- Cache results, update incrementally

### Code Example
```python
class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y


class AABB:
    def __init__(self, x, y, width, height):
        self.min_x = x
        self.min_y = y
        self.max_x = x + width
        self.max_y = y + height

    def intersects(self, other):
        return (self.max_x >= other.min_x and
                self.min_x <= other.max_x and
                self.max_y >= other.min_y and
                self.min_y <= other.max_y)


class SpatialGrid:
    """Simple spatial partitioning using a grid"""

    def __init__(self, cell_size, width, height):
        self.cell_size = cell_size
        self.cols = int(width / cell_size) + 1
        self.rows = int(height / cell_size) + 1
        self.cells = {}  # (col, row) -> list of objects

    def _get_cell(self, x, y):
        """Get cell coordinates for a point"""
        col = int(x / self.cell_size)
        row = int(y / self.cell_size)
        return (col, row)

    def _get_cells_for_aabb(self, aabb):
        """Get all cells that an AABB overlaps"""
        min_col = int(aabb.min_x / self.cell_size)
        max_col = int(aabb.max_x / self.cell_size)
        min_row = int(aabb.min_y / self.cell_size)
        max_row = int(aabb.max_y / self.cell_size)

        cells = []
        for col in range(min_col, max_col + 1):
            for row in range(min_row, max_row + 1):
                cells.append((col, row))
        return cells

    def clear(self):
        """Clear all objects from grid"""
        self.cells = {}

    def insert(self, obj, aabb):
        """Insert object into grid"""
        for cell in self._get_cells_for_aabb(aabb):
            if cell not in self.cells:
                self.cells[cell] = []
            self.cells[cell].append((obj, aabb))

    def get_potential_collisions(self, aabb):
        """Get all objects that might collide with given AABB"""
        potential = set()
        for cell in self._get_cells_for_aabb(aabb):
            if cell in self.cells:
                for obj, _ in self.cells[cell]:
                    potential.add(obj)
        return potential


class CollisionSystem:
    """Two-phase collision detection system"""

    def __init__(self, world_width, world_height, cell_size=100):
        self.grid = SpatialGrid(cell_size, world_width, world_height)
        self.objects = {}  # obj_id -> (obj, aabb)

    def add_object(self, obj_id, aabb):
        self.objects[obj_id] = aabb
        self.grid.insert(obj_id, aabb)

    def rebuild_grid(self):
        """Rebuild spatial grid (call when objects move)"""
        self.grid.clear()
        for obj_id, aabb in self.objects.items():
            self.grid.insert(obj_id, aabb)

    def get_all_collisions(self):
        """Get all colliding pairs using broad + narrow phase"""
        checked = set()
        collisions = []

        for obj_id, aabb in self.objects.items():
            # Broad phase: get potential collisions from grid
            potential = self.grid.get_potential_collisions(aabb)

            for other_id in potential:
                if other_id == obj_id:
                    continue

                # Avoid checking same pair twice
                pair = tuple(sorted([obj_id, other_id]))
                if pair in checked:
                    continue
                checked.add(pair)

                # Narrow phase: precise AABB check
                other_aabb = self.objects[other_id]
                if aabb.intersects(other_aabb):
                    collisions.append(pair)

        return collisions


# Benchmark: Grid vs Brute Force
import time
import random

# Create many objects
num_objects = 1000
world_size = 1000

print(f"Collision detection with {num_objects} objects:")

# Create random AABBs
objects = []
for i in range(num_objects):
    x = random.uniform(0, world_size - 20)
    y = random.uniform(0, world_size - 20)
    w = random.uniform(10, 30)
    h = random.uniform(10, 30)
    objects.append((i, AABB(x, y, w, h)))

# Brute force
start = time.time()
brute_collisions = []
for i in range(len(objects)):
    for j in range(i + 1, len(objects)):
        if objects[i][1].intersects(objects[j][1]):
            brute_collisions.append((i, j))
brute_time = time.time() - start

# Grid-based
start = time.time()
system = CollisionSystem(world_size, world_size, cell_size=50)
for obj_id, aabb in objects:
    system.add_object(obj_id, aabb)
grid_collisions = system.get_all_collisions()
grid_time = time.time() - start

print(f"\nBrute force: {len(brute_collisions)} collisions in {brute_time*1000:.2f}ms")
print(f"Grid-based:  {len(grid_collisions)} collisions in {grid_time*1000:.2f}ms")
print(f"Speedup: {brute_time/grid_time:.1f}x")
```

---

## Summary (요약)

| Technique | Korean | Use Case |
|-----------|--------|----------|
| OBB | 회전 박스 | Rotated objects |
| SAT | 분리 축 정리 | Convex polygon collision |
| Raycasting | 레이캐스팅 | Line of sight, shooting |
| Spatial Grid | 공간 분할 | Many objects optimization |
| Quadtree | 쿼드트리 | Variable density areas |
| Broad/Narrow | 브로드/내로우 | Two-phase optimization |

---

## Practice Problems (연습 문제)

1. Two OBBs: A at (0,0) size 10x5 rotated 30°, B at (12, 0) size 8x4 rotated -20°. Do they collide?

2. A ray from (0, 0) in direction (1, 1) toward a circle at (10, 10) radius 3. At what distance does it hit?

3. You have 500 objects in a 1000x1000 world. Design a spatial grid. What cell size would you choose?

4. Implement a quadtree for collision detection (conceptual: describe the insert and query algorithms).
