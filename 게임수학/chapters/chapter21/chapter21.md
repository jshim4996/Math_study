# Chapter 21. Forces and Motion (힘과 운동)

> Applying Newton's laws: forces, acceleration, and simple physics simulations.

---

## Force (힘)

### Concept (개념)

**Force** is a push or pull that causes acceleration. In physics simulations:

```
Force causes acceleration
Acceleration changes velocity
Velocity changes position
```

**Newton's First Law:**
An object at rest stays at rest, an object in motion stays in motion (unless a force acts on it).

**Representing forces:**
- Forces are vectors (magnitude + direction)
- Multiple forces can act on an object
- Forces are summed to get net force

### Game Example
- Pushing objects
- Wind effects
- Explosions
- Engine thrust

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
        return f"({self.x:.2f}, {self.y:.2f})"


class ForceObject:
    def __init__(self, x, y, mass=1.0):
        self.position = Vector2(x, y)
        self.velocity = Vector2(0, 0)
        self.mass = mass
        self.forces = []  # List of forces acting on object

    def add_force(self, force):
        """Add a force to be applied this frame"""
        self.forces.append(force)

    def clear_forces(self):
        """Clear all forces"""
        self.forces = []

    def get_net_force(self):
        """Sum all forces"""
        net = Vector2(0, 0)
        for f in self.forces:
            net = net + f
        return net


# Multiple forces on an object
obj = ForceObject(0, 0, mass=10)

# Apply multiple forces
gravity = Vector2(0, -98)  # Weight = mass * g
thrust = Vector2(150, 0)   # Engine thrust
wind = Vector2(-20, 10)    # Wind resistance

obj.add_force(gravity)
obj.add_force(thrust)
obj.add_force(wind)

net = obj.get_net_force()
print("Forces acting on object:")
print(f"  Gravity: {gravity}")
print(f"  Thrust:  {thrust}")
print(f"  Wind:    {wind}")
print(f"  Net force: {net}")
```

---

## Newton's Second Law: F = ma (뉴턴의 제2법칙)

### Concept (개념)

**F = ma** (Force = mass × acceleration)

Rearranged for games:
```
acceleration = force / mass
a = F / m
```

**Implications:**
- Same force on heavier object → less acceleration
- More force needed to move heavy objects quickly
- Light objects accelerate faster

### Game Example
- Heavy tanks vs light vehicles
- Character strength affecting push force
- Realistic object physics

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

    def __repr__(self):
        return f"({self.x:.2f}, {self.y:.2f})"


class PhysicsBody:
    def __init__(self, x, y, mass):
        self.position = Vector2(x, y)
        self.velocity = Vector2(0, 0)
        self.mass = mass
        self.force_accumulator = Vector2(0, 0)

    def apply_force(self, force):
        """Accumulate force for this frame"""
        self.force_accumulator = self.force_accumulator + force

    def update(self, dt):
        """Update physics using F = ma"""
        # Calculate acceleration (a = F / m)
        if self.mass > 0:
            acceleration = self.force_accumulator * (1.0 / self.mass)
        else:
            acceleration = Vector2(0, 0)

        # Update velocity
        self.velocity = self.velocity + acceleration * dt

        # Update position
        self.position = self.position + self.velocity * dt

        # Clear forces for next frame
        self.force_accumulator = Vector2(0, 0)


# Compare light vs heavy objects with same force
force = Vector2(100, 0)

light_obj = PhysicsBody(0, 0, mass=1)
heavy_obj = PhysicsBody(0, 0, mass=10)

print("Same force (100N) on different masses:")
print(f"  Light object (1 kg):  a = {force.x / light_obj.mass:.1f} m/s²")
print(f"  Heavy object (10 kg): a = {force.x / heavy_obj.mass:.1f} m/s²")

# Simulate for 1 second
dt = 0.1
print("\nPosition after 1 second:")
for i in range(10):
    light_obj.apply_force(force)
    heavy_obj.apply_force(force)
    light_obj.update(dt)
    heavy_obj.update(dt)

print(f"  Light: x = {light_obj.position.x:.1f}")
print(f"  Heavy: x = {heavy_obj.position.x:.1f}")


# Pushing objects with player strength
class PushableObject:
    def __init__(self, x, y, mass, name):
        self.body = PhysicsBody(x, y, mass)
        self.name = name


player_push_force = 50  # Newtons
objects = [
    PushableObject(0, 0, 5, "Box"),
    PushableObject(0, 0, 20, "Crate"),
    PushableObject(0, 0, 100, "Boulder"),
]

print(f"\nPushing objects with {player_push_force}N force:")
for obj in objects:
    obj.body.apply_force(Vector2(player_push_force, 0))
    obj.body.update(1.0)  # 1 second
    print(f"  {obj.name} (mass={obj.body.mass}): moved {obj.body.position.x:.2f} units")
```

---

## Combining Multiple Forces (여러 힘의 합성)

### Concept (개념)

When multiple forces act on an object, add them as vectors:

```
net_force = force1 + force2 + force3 + ...
```

**Common force combinations:**
- Gravity + Air resistance
- Thrust + Drag
- Push forces from multiple sources
- Constraint forces

### Game Example
- Space games (thrust in any direction vs gravity)
- Wind + Movement
- Tug of war
- Physics puzzles

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


class RigidBody:
    def __init__(self, x, y, mass):
        self.position = Vector2(x, y)
        self.velocity = Vector2(0, 0)
        self.mass = mass
        self.forces = []

    def add_force(self, force, name=""):
        self.forces.append((force, name))

    def clear_forces(self):
        self.forces = []

    def update(self, dt):
        # Sum all forces
        net_force = Vector2(0, 0)
        for force, _ in self.forces:
            net_force = net_force + force

        # F = ma → a = F/m
        acceleration = net_force * (1.0 / self.mass) if self.mass > 0 else Vector2(0, 0)

        # Integrate
        self.velocity = self.velocity + acceleration * dt
        self.position = self.position + self.velocity * dt

        self.clear_forces()
        return net_force


# Rocket with multiple forces
rocket = RigidBody(0, 100, mass=1000)

def simulate_rocket_frame(rocket, thrust_on, dt):
    # Gravity (always present)
    gravity = Vector2(0, -9800)  # Weight = mg
    rocket.add_force(gravity, "Gravity")

    # Thrust (when engine on)
    if thrust_on:
        thrust = Vector2(0, 15000)  # Upward thrust
        rocket.add_force(thrust, "Thrust")

    # Air resistance (proportional to velocity squared, simplified)
    if rocket.velocity.magnitude() > 0:
        drag_coefficient = 0.5
        drag_mag = drag_coefficient * rocket.velocity.magnitude()
        drag_dir = rocket.velocity.normalized() * -1
        drag = drag_dir * drag_mag
        rocket.add_force(drag, "Drag")

    net = rocket.update(dt)
    return net


print("Rocket launch simulation:")
print("(Thrust: 15000N, Weight: 9800N, Net: ~5200N up)")

dt = 0.1
thrust_on = True

for frame in range(20):
    # Turn off thrust after 1 second
    if frame * dt > 1.0:
        thrust_on = False

    net = simulate_rocket_frame(rocket, thrust_on, dt)
    thrust_str = "ON " if thrust_on else "OFF"
    print(f"  t={frame*dt:.1f}s: y={rocket.position.y:7.1f}, vy={rocket.velocity.y:7.1f}, thrust={thrust_str}")

    if rocket.position.y < 0:
        print("  (Crashed)")
        break
```

---

## Basic Friction (마찰력 기초)

### Concept (개념)

**Friction** opposes motion:

```
friction_force = -friction_coefficient × velocity
```

Or for ground friction:
```
friction_magnitude = friction_coefficient × normal_force
Direction: opposite to velocity
```

**Types:**
- **Static friction**: Prevents object from starting to move
- **Kinetic friction**: Slows moving objects
- **Drag/Air resistance**: Proportional to velocity (or velocity²)

### Game Example
- Cars slowing down
- Characters stopping
- Objects sliding to a halt
- Different surfaces (ice, mud, concrete)

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


class FrictionObject:
    def __init__(self, x, y, mass):
        self.position = Vector2(x, y)
        self.velocity = Vector2(0, 0)
        self.mass = mass

    def apply_friction(self, friction_coefficient, dt):
        """Apply friction force (simplified model)"""
        if self.velocity.magnitude() < 0.01:
            self.velocity = Vector2(0, 0)
            return

        # Friction opposes motion
        friction_dir = self.velocity.normalized() * -1
        friction_mag = friction_coefficient * self.mass * 9.8  # μ × N (N = mg)

        # Don't let friction reverse direction
        max_friction = self.velocity.magnitude() * self.mass / dt
        friction_mag = min(friction_mag, max_friction)

        friction_force = friction_dir * friction_mag

        # Apply friction (a = F/m)
        acceleration = friction_force * (1.0 / self.mass)
        self.velocity = self.velocity + acceleration * dt

    def update(self, dt):
        self.position = self.position + self.velocity * dt


# Compare different surfaces
surfaces = [
    ("Ice", 0.03),
    ("Wood", 0.3),
    ("Concrete", 0.6),
    ("Rubber", 0.9),
]

print("Object sliding on different surfaces:")
print("(Initial velocity: 10 m/s)")

for surface_name, friction_coef in surfaces:
    obj = FrictionObject(0, 0, mass=1)
    obj.velocity = Vector2(10, 0)

    dt = 0.1
    time = 0
    while obj.velocity.magnitude() > 0.01 and time < 10:
        obj.apply_friction(friction_coef, dt)
        obj.update(dt)
        time += dt

    print(f"  {surface_name:10} (μ={friction_coef}): stopped after {time:.1f}s, traveled {obj.position.x:.1f}m")


# Car with engine and friction
class Car:
    def __init__(self):
        self.position = Vector2(0, 0)
        self.velocity = Vector2(0, 0)
        self.mass = 1000
        self.engine_force = 0
        self.friction_coef = 0.7  # Road friction

    def set_throttle(self, throttle):
        """Throttle 0-1"""
        max_engine_force = 5000
        self.engine_force = throttle * max_engine_force

    def update(self, dt):
        # Engine force (forward)
        force = Vector2(self.engine_force, 0)

        # Rolling resistance (simplified)
        if self.velocity.magnitude() > 0:
            drag = self.velocity.normalized() * -1 * (self.velocity.magnitude() * 10)
            force = force + drag

        # Apply force
        acceleration = force * (1.0 / self.mass)
        self.velocity = self.velocity + acceleration * dt
        self.position = self.position + self.velocity * dt


print("\nCar acceleration and deceleration:")
car = Car()
car.set_throttle(1.0)  # Full throttle

for i in range(20):
    # Release throttle after 1 second
    if i * 0.1 > 1.0:
        car.set_throttle(0)

    print(f"  t={i*0.1:.1f}s: pos={car.position.x:6.1f}, vel={car.velocity.x:5.1f}")
    car.update(0.1)
```

---

## Simple Physics Simulation (간단한 물리 시뮬레이션)

### Concept (개념)

Putting it all together - a complete physics simulation loop:

```
1. Clear forces
2. Apply all forces (gravity, input, friction, etc.)
3. Calculate net force
4. Calculate acceleration (F/m)
5. Update velocity
6. Update position
7. Handle collisions
8. Repeat
```

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


class PhysicsWorld:
    def __init__(self):
        self.bodies = []
        self.gravity = Vector2(0, -200)
        self.ground_y = 0

    def add_body(self, body):
        self.bodies.append(body)

    def update(self, dt):
        for body in self.bodies:
            # 1. Apply gravity
            weight = self.gravity * body.mass
            body.apply_force(weight)

            # 2. Apply air resistance
            if body.velocity.magnitude() > 0:
                drag_coef = 0.1
                drag = body.velocity.normalized() * -1 * (body.velocity.magnitude() * drag_coef)
                body.apply_force(drag)

            # 3. Update physics
            body.update(dt)

            # 4. Ground collision
            if body.position.y < self.ground_y:
                body.position.y = self.ground_y
                body.velocity.y = -body.velocity.y * body.bounciness


class SimulatedBody:
    def __init__(self, x, y, mass, bounciness=0.5):
        self.position = Vector2(x, y)
        self.velocity = Vector2(0, 0)
        self.mass = mass
        self.bounciness = bounciness
        self.force_accumulator = Vector2(0, 0)

    def apply_force(self, force):
        self.force_accumulator = self.force_accumulator + force

    def update(self, dt):
        # a = F/m
        acceleration = self.force_accumulator * (1.0 / self.mass)

        # Update velocity and position
        self.velocity = self.velocity + acceleration * dt
        self.position = self.position + self.velocity * dt

        # Clear forces
        self.force_accumulator = Vector2(0, 0)


# Simulation
world = PhysicsWorld()

# Add a bouncy ball
ball = SimulatedBody(0, 100, mass=1, bounciness=0.7)
ball.velocity = Vector2(50, 0)  # Initial horizontal velocity
world.add_body(ball)

print("Bouncing ball simulation:")
dt = 0.033  # ~30 FPS

for frame in range(60):
    if frame % 5 == 0:
        print(f"  t={frame*dt:.2f}s: pos={ball.position}, vel=({ball.velocity.x:.1f}, {ball.velocity.y:.1f})")

    world.update(dt)

    # Stop when ball settles
    if ball.position.y <= 0 and abs(ball.velocity.y) < 1:
        print(f"  Ball settled at x={ball.position.x:.1f}")
        break
```

---

## Summary (요약)

| Concept | Korean | Formula |
|---------|--------|---------|
| Force | 힘 | F = ma |
| Acceleration | 가속도 | a = F/m |
| Net Force | 합력 | F_net = ΣF |
| Friction | 마찰력 | F_f = μN (opposite to velocity) |

**Physics Loop:**
1. Apply forces → 2. Calculate acceleration → 3. Update velocity → 4. Update position → 5. Handle collisions

---

## Practice Problems (연습 문제)

1. A 5kg object has forces of (10, 0) and (0, 15) applied. What's the acceleration?

2. A car with mass 1500kg needs to accelerate at 3 m/s². What force does the engine need?

3. An object moving at 20 m/s on a surface with friction coefficient 0.4. How long until it stops?

4. Create a simple projectile simulation with gravity and air resistance.
