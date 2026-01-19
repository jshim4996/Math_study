# Chapter 11. Matrix Basics (행렬 기초)

> The foundation for transforms: what matrices are, how to multiply them, and why they matter for games.

---

## What is a Matrix (행렬이란 무엇인가)

### Concept (개념)

A **matrix** is a rectangular grid of numbers arranged in rows and columns.

```
     columns
       ↓
    [1  2  3]  ← row 0
    [4  5  6]  ← row 1
```

**Notation:**
- An m×n matrix has m rows and n columns
- Element at row i, column j is written as A[i][j] or Aᵢⱼ
- Matrices are typically written with capital letters (A, B, M)

**Common sizes in games:**
- 2×2: 2D rotation/scale
- 3×3: 2D transforms with translation (homogeneous)
- 4×4: 3D transforms (standard in 3D graphics)

### Game Example
A 4×4 transformation matrix stores an object's position, rotation, and scale all in one structure. Game engines use these to transform vertices.

### Code Example
```python
class Matrix:
    def __init__(self, rows, cols, data=None):
        self.rows = rows
        self.cols = cols
        if data:
            self.data = data
        else:
            # Initialize with zeros
            self.data = [[0 for _ in range(cols)] for _ in range(rows)]

    def __getitem__(self, index):
        return self.data[index]

    def __setitem__(self, index, value):
        self.data[index] = value

    def __repr__(self):
        result = []
        for row in self.data:
            formatted = [f"{x:7.2f}" for x in row]
            result.append("[" + " ".join(formatted) + "]")
        return "\n".join(result)


# Create a 2x3 matrix
m = Matrix(2, 3, [
    [1, 2, 3],
    [4, 5, 6]
])

print("2x3 Matrix:")
print(m)
print(f"\nElement at [0][1]: {m[0][1]}")  # 2
print(f"Element at [1][2]: {m[1][2]}")  # 6

# Create a 3x3 matrix
m3 = Matrix(3, 3, [
    [1, 0, 0],
    [0, 1, 0],
    [0, 0, 1]
])
print("\n3x3 Identity Matrix:")
print(m3)
```

---

## Matrix Representation: Rows, Columns (행렬 표현)

### Concept (개념)

**Row-major vs Column-major:**
- **Row-major**: Elements stored row by row (C, Python, DirectX)
- **Column-major**: Elements stored column by column (OpenGL, GLSL)

This affects how you interpret and multiply matrices. Most game engines use one convention consistently.

**Vectors as matrices:**
- Row vector: 1×n matrix [x, y, z]
- Column vector: n×1 matrix

```
Row vector (1×3):     Column vector (3×1):
[x  y  z]             [x]
                      [y]
                      [z]
```

### Code Example
```python
class Matrix:
    def __init__(self, rows, cols, data=None):
        self.rows = rows
        self.cols = cols
        self.data = data if data else [[0]*cols for _ in range(rows)]

    def __repr__(self):
        return "\n".join(["[" + " ".join(f"{x:6.2f}" for x in row) + "]" for row in self.data])

    def get(self, row, col):
        return self.data[row][col]

    def set(self, row, col, value):
        self.data[row][col] = value


# Row vector (1x3)
row_vec = Matrix(1, 3, [[10, 20, 30]])
print("Row vector (1x3):")
print(row_vec)

# Column vector (3x1)
col_vec = Matrix(3, 1, [[10], [20], [30]])
print("\nColumn vector (3x1):")
print(col_vec)

# Accessing elements
print(f"\nRow vector [0][1] = {row_vec.get(0, 1)}")    # 20
print(f"Column vector [1][0] = {col_vec.get(1, 0)}")  # 20
```

---

## Matrix Addition, Subtraction (행렬 덧셈, 뺄셈)

### Concept (개념)

Add or subtract matrices **element by element**. Both matrices must have the same dimensions.

```
[a b]   [e f]   [a+e  b+f]
[c d] + [g h] = [c+g  d+h]
```

**Properties:**
- Commutative: A + B = B + A
- Associative: (A + B) + C = A + (B + C)

### Game Example
Blending two transformation matrices (though multiplication is more common for transforms).

### Code Example
```python
class Matrix:
    def __init__(self, rows, cols, data=None):
        self.rows = rows
        self.cols = cols
        self.data = data if data else [[0]*cols for _ in range(rows)]

    def __repr__(self):
        return "\n".join(["[" + " ".join(f"{x:6.2f}" for x in row) + "]" for row in self.data])

    def __add__(self, other):
        if self.rows != other.rows or self.cols != other.cols:
            raise ValueError("Matrix dimensions must match")
        result = Matrix(self.rows, self.cols)
        for i in range(self.rows):
            for j in range(self.cols):
                result.data[i][j] = self.data[i][j] + other.data[i][j]
        return result

    def __sub__(self, other):
        if self.rows != other.rows or self.cols != other.cols:
            raise ValueError("Matrix dimensions must match")
        result = Matrix(self.rows, self.cols)
        for i in range(self.rows):
            for j in range(self.cols):
                result.data[i][j] = self.data[i][j] - other.data[i][j]
        return result


# Matrix addition
A = Matrix(2, 2, [[1, 2], [3, 4]])
B = Matrix(2, 2, [[5, 6], [7, 8]])

print("A:")
print(A)
print("\nB:")
print(B)
print("\nA + B:")
print(A + B)
print("\nA - B:")
print(A - B)
```

---

## Matrix Multiplication (행렬 곱셈)

### Concept (개념)

Matrix multiplication is **NOT** element-by-element. For A × B:
- A must have the same number of **columns** as B has **rows**
- Result dimensions: (A rows) × (B columns)

**Formula:** For result C = A × B:
```
C[i][j] = sum of A[i][k] * B[k][j] for all k
```

Visually: Row i of A "dot product" with Column j of B.

```
[a b]   [e f]   [ae+bg  af+bh]
[c d] × [g h] = [ce+dg  cf+dh]
```

**Important:** Matrix multiplication is **NOT commutative**: A × B ≠ B × A

### Game Example
Combining transformations: Scale × Rotation × Translation. The order matters!

### Code Example
```python
class Matrix:
    def __init__(self, rows, cols, data=None):
        self.rows = rows
        self.cols = cols
        self.data = data if data else [[0]*cols for _ in range(rows)]

    def __repr__(self):
        return "\n".join(["[" + " ".join(f"{x:7.2f}" for x in row) + "]" for row in self.data])

    def __mul__(self, other):
        # Matrix × Matrix
        if isinstance(other, Matrix):
            if self.cols != other.rows:
                raise ValueError(f"Cannot multiply {self.rows}x{self.cols} by {other.rows}x{other.cols}")

            result = Matrix(self.rows, other.cols)
            for i in range(self.rows):
                for j in range(other.cols):
                    total = 0
                    for k in range(self.cols):
                        total += self.data[i][k] * other.data[k][j]
                    result.data[i][j] = total
            return result

        # Matrix × scalar
        elif isinstance(other, (int, float)):
            result = Matrix(self.rows, self.cols)
            for i in range(self.rows):
                for j in range(self.cols):
                    result.data[i][j] = self.data[i][j] * other
            return result


# 2x2 multiplication
A = Matrix(2, 2, [[1, 2], [3, 4]])
B = Matrix(2, 2, [[5, 6], [7, 8]])

print("A:")
print(A)
print("\nB:")
print(B)
print("\nA × B:")
print(A * B)  # [[19, 22], [43, 50]]
print("\nB × A:")
print(B * A)  # [[23, 34], [31, 46]] - Different!

# 2x3 × 3x2 = 2x2
C = Matrix(2, 3, [[1, 2, 3], [4, 5, 6]])
D = Matrix(3, 2, [[7, 8], [9, 10], [11, 12]])

print("\nC (2x3):")
print(C)
print("\nD (3x2):")
print(D)
print("\nC × D (2x2):")
print(C * D)

# Scalar multiplication
print("\nA × 2:")
print(A * 2)
```

---

## Identity Matrix (단위 행렬)

### Concept (개념)

The **identity matrix** (I) is the matrix equivalent of multiplying by 1:
```
A × I = A
I × A = A
```

For an n×n identity matrix, 1s on the diagonal, 0s elsewhere:

```
2×2 Identity:    3×3 Identity:    4×4 Identity:
[1 0]            [1 0 0]          [1 0 0 0]
[0 1]            [0 1 0]          [0 1 0 0]
                 [0 0 1]          [0 0 1 0]
                                  [0 0 0 1]
```

### Game Example
When no transformation is applied, an object uses the identity matrix. Starting point for building transformation matrices.

### Code Example
```python
class Matrix:
    def __init__(self, rows, cols, data=None):
        self.rows = rows
        self.cols = cols
        self.data = data if data else [[0]*cols for _ in range(rows)]

    def __repr__(self):
        return "\n".join(["[" + " ".join(f"{x:6.2f}" for x in row) + "]" for row in self.data])

    def __mul__(self, other):
        if isinstance(other, Matrix):
            if self.cols != other.rows:
                raise ValueError("Dimension mismatch")
            result = Matrix(self.rows, other.cols)
            for i in range(self.rows):
                for j in range(other.cols):
                    total = 0
                    for k in range(self.cols):
                        total += self.data[i][k] * other.data[k][j]
                    result.data[i][j] = total
            return result
        return NotImplemented

    @staticmethod
    def identity(size):
        """Create an identity matrix of given size"""
        m = Matrix(size, size)
        for i in range(size):
            m.data[i][i] = 1
        return m


# Create identity matrices
I2 = Matrix.identity(2)
I3 = Matrix.identity(3)
I4 = Matrix.identity(4)

print("2×2 Identity:")
print(I2)
print("\n3×3 Identity:")
print(I3)
print("\n4×4 Identity:")
print(I4)

# Verify: A × I = A
A = Matrix(3, 3, [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
])

print("\nA:")
print(A)
print("\nA × I3:")
print(A * I3)
print("\nI3 × A:")
print(I3 * A)
# Both equal A
```

---

## Transforming Vectors with Matrices (행렬로 벡터 변환)

### Concept (개념)

To transform a vector using a matrix, treat the vector as a matrix and multiply:

**Column vector convention (OpenGL):**
```
[a b] × [x] = [ax + by]
[c d]   [y]   [cx + dy]
```

**Row vector convention (DirectX):**
```
[x y] × [a b] = [xa + yc, xb + yd]
        [c d]
```

### Game Example
Apply a rotation matrix to a vertex position to rotate it around the origin.

### Code Example
```python
import math

class Vector2:
    def __init__(self, x=0, y=0):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"({self.x:.2f}, {self.y:.2f})"


class Matrix2x2:
    def __init__(self, data):
        self.data = data  # [[a, b], [c, d]]

    def __repr__(self):
        return f"[{self.data[0][0]:6.2f} {self.data[0][1]:6.2f}]\n[{self.data[1][0]:6.2f} {self.data[1][1]:6.2f}]"

    def transform(self, v):
        """Transform a Vector2 using this matrix"""
        x = self.data[0][0] * v.x + self.data[0][1] * v.y
        y = self.data[1][0] * v.x + self.data[1][1] * v.y
        return Vector2(x, y)

    @staticmethod
    def identity():
        return Matrix2x2([[1, 0], [0, 1]])

    @staticmethod
    def rotation(degrees):
        rad = math.radians(degrees)
        c, s = math.cos(rad), math.sin(rad)
        return Matrix2x2([[c, -s], [s, c]])

    @staticmethod
    def scale(sx, sy):
        return Matrix2x2([[sx, 0], [0, sy]])


# Transform a point with different matrices
point = Vector2(1, 0)

# Identity (no change)
I = Matrix2x2.identity()
print(f"Original point: {point}")
print(f"After identity: {I.transform(point)}")

# Rotation by 90°
R = Matrix2x2.rotation(90)
print(f"\n90° Rotation matrix:")
print(R)
print(f"Point after 90° rotation: {R.transform(point)}")  # (0, 1)

# Scale by 2x in X, 3x in Y
S = Matrix2x2.scale(2, 3)
print(f"\nScale matrix (2, 3):")
print(S)
point2 = Vector2(1, 1)
print(f"Point {point2} after scale: {S.transform(point2)}")  # (2, 3)
```

---

## Summary (요약)

| Concept | Korean | Key Point |
|---------|--------|-----------|
| Matrix | 행렬 | Grid of numbers (rows × columns) |
| Addition | 덧셈 | Element by element, same dimensions |
| Multiplication | 곱셈 | Row dot Column, A.cols = B.rows |
| Identity | 단위 행렬 | 1s on diagonal, A × I = A |
| Transform | 변환 | Matrix × Vector = transformed vector |

**Key Rules:**
- A × B ≠ B × A (not commutative)
- (A × B) × C = A × (B × C) (associative)
- A × I = I × A = A

---

## Practice Problems (연습 문제)

1. Multiply these matrices:
   ```
   [1 2]   [5 6]
   [3 4] × [7 8]
   ```

2. Create a 90° rotation matrix and verify it rotates (1, 0) to (0, 1).

3. What are the dimensions of the result when multiplying a 3×4 matrix by a 4×2 matrix?

4. Create a scale matrix that doubles X and halves Y. Transform the point (10, 20).
