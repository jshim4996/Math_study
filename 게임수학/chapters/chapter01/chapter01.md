# Chapter 01. Coordinate Systems (좌표계)

> Understanding how positions are represented in game worlds - the foundation of all game math.

---

## What is a Coordinate System? (좌표계란?)

### Concept (개념)
A coordinate system is a method for describing positions in space using numbers. In games, every object - characters, enemies, projectiles, UI elements - needs a position. Coordinate systems give us a precise way to define "where" something is.

Think of it like an address system for your game world. Just as street addresses help you find buildings, coordinates help your game engine find and place objects.

### Game Example
When you place a character in a game, you need to tell the engine exactly where. Without coordinates, you couldn't specify "spawn the player at the center of the map" or "place the treasure chest in the corner of the room."

### Code Example
```python
# Every game object has a position
class GameObject:
    def __init__(self, x, y):
        self.x = x  # Horizontal position
        self.y = y  # Vertical position

player = GameObject(100, 200)
enemy = GameObject(300, 200)
```

---

## 2D Coordinate System (2D 좌표계)

### Concept (개념)
A 2D coordinate system uses two axes:
- **X-axis (X축)**: Horizontal direction (left/right)
- **Y-axis (Y축)**: Vertical direction (up/down)

Any point can be described as (x, y), where x is the horizontal distance from the origin and y is the vertical distance.

The **origin (원점)** is the point (0, 0) where both axes meet.

### Game Example
In a 2D platformer like Mario, the character's position is described with just two numbers. If Mario is at position (150, 80), he's 150 units to the right and 80 units up from the origin.

### Code Example
```python
# 2D position representation
class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"({self.x}, {self.y})"

# Character at position (150, 80)
mario_position = Vector2(150, 80)
print(f"Mario is at {mario_position}")
```

---

## Screen Coordinates vs Math Coordinates (스크린 좌표 vs 수학 좌표)

### Concept (개념)
**Math coordinates (수학 좌표)**:
- Y increases upward (↑)
- Origin at center or bottom-left
- What you learned in school

**Screen coordinates (스크린 좌표)**:
- Y increases downward (↓)
- Origin at top-left corner
- Used by most graphics systems and game engines

This difference exists because computer monitors draw pixels from top to bottom, left to right.

### Game Example
If your game world uses math coordinates but the rendering system uses screen coordinates, you need to convert:
- World position (100, 300) with screen height 600
- Screen position = (100, 600 - 300) = (100, 300)

Many 2D game engines like Unity use math coordinates (Y-up), while HTML Canvas and many graphics libraries use screen coordinates (Y-down).

### Code Example
```python
SCREEN_HEIGHT = 600

def world_to_screen(world_x, world_y):
    """Convert math coordinates to screen coordinates"""
    screen_x = world_x
    screen_y = SCREEN_HEIGHT - world_y
    return (screen_x, screen_y)

def screen_to_world(screen_x, screen_y):
    """Convert screen coordinates to math coordinates"""
    world_x = screen_x
    world_y = SCREEN_HEIGHT - screen_y
    return (world_x, world_y)

# Player at world position (100, 400)
world_pos = (100, 400)
screen_pos = world_to_screen(100, 400)  # (100, 200)
print(f"World: {world_pos} -> Screen: {screen_pos}")
```

---

## 3D Coordinate System (3D 좌표계)

### Concept (개념)
3D coordinates add a third axis:
- **X-axis (X축)**: Left/Right
- **Y-axis (Y축)**: Up/Down (usually)
- **Z-axis (Z축)**: Forward/Backward (depth)

A point in 3D space is described as (x, y, z).

### Game Example
In a 3D game like Minecraft, every block has a 3D position. A block at (10, 64, -20) means:
- 10 units in X direction
- 64 units high (Y)
- 20 units in negative Z direction

### Code Example
```python
class Vector3:
    def __init__(self, x=0, y=0, z=0):
        self.x = x
        self.y = y
        self.z = z

    def __repr__(self):
        return f"({self.x}, {self.y}, {self.z})"

# Player position in 3D world
player_pos = Vector3(10, 64, -20)
print(f"Player is at {player_pos}")
```

---

## Left-Handed vs Right-Handed Coordinate Systems (왼손 좌표계 vs 오른손 좌표계)

### Concept (개념)
In 3D, there are two conventions for axis orientation:

**Right-handed system (오른손 좌표계)**:
- Point your right hand's fingers along +X
- Curl them toward +Y
- Your thumb points to +Z
- Used by: OpenGL, Blender, Maya

**Left-handed system (왼손 좌표계)**:
- Same gesture with left hand
- +Z goes into the screen
- Used by: DirectX, Unity, Unreal

### Game Example
This matters when:
- Importing 3D models between tools
- Working with cross-platform engines
- Implementing camera systems

If you import a model from Blender (right-handed) into Unity (left-handed), you might need to flip the Z-axis or the model will appear mirrored.

### Code Example
```python
def convert_right_to_left_handed(x, y, z):
    """Convert from right-handed to left-handed coordinate system"""
    # Typically, we negate the Z axis
    return (x, y, -z)

def convert_left_to_right_handed(x, y, z):
    """Convert from left-handed to right-handed coordinate system"""
    return (x, y, -z)

# Position in Blender (right-handed)
blender_pos = (10, 5, 20)

# Convert for Unity (left-handed)
unity_pos = convert_right_to_left_handed(*blender_pos)
print(f"Blender: {blender_pos} -> Unity: {unity_pos}")  # (10, 5, -20)
```

---

## Practical Application: Setting Up a Game World

### Code Example
```python
class GameWorld:
    """A simple 2D game world with coordinate system handling"""

    def __init__(self, width, height, use_screen_coords=True):
        self.width = width
        self.height = height
        self.use_screen_coords = use_screen_coords
        self.objects = []

    def add_object(self, name, x, y):
        """Add object at given position"""
        self.objects.append({
            'name': name,
            'x': x,
            'y': y
        })

    def get_screen_position(self, obj):
        """Get screen position for rendering"""
        if self.use_screen_coords:
            return (obj['x'], obj['y'])
        else:
            # Convert from math coords to screen coords
            return (obj['x'], self.height - obj['y'])

    def print_all_objects(self):
        for obj in self.objects:
            screen_pos = self.get_screen_position(obj)
            print(f"{obj['name']}: world({obj['x']}, {obj['y']}) -> screen{screen_pos}")

# Create game world (800x600, using math coordinates)
world = GameWorld(800, 600, use_screen_coords=False)
world.add_object("Player", 400, 300)  # Center
world.add_object("Enemy", 100, 500)   # Top-left area
world.add_object("Coin", 700, 100)    # Bottom-right area

world.print_all_objects()
```

---

## Summary (요약)

| Term | Korean | Description |
|------|--------|-------------|
| Coordinate System | 좌표계 | Method to describe positions using numbers |
| Origin | 원점 | The (0,0) point where axes meet |
| X-axis | X축 | Horizontal axis |
| Y-axis | Y축 | Vertical axis |
| Z-axis | Z축 | Depth axis (3D) |
| Screen Coordinates | 스크린 좌표 | Y increases downward |
| Math Coordinates | 수학 좌표 | Y increases upward |
| Right-handed | 오른손 좌표계 | OpenGL convention |
| Left-handed | 왼손 좌표계 | DirectX/Unity convention |

---

## Practice Problems (연습 문제)

1. Convert world position (200, 450) to screen coordinates with screen height 600.
2. A character is at screen position (150, 100). Convert to math coordinates (screen height 480).
3. You're importing a model from Blender at position (5, 10, 15) into Unity. What's the converted position?
