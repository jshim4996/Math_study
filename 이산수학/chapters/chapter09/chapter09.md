# Chapter 09. Special Graphs (특수 그래프)

> Trees, DAGs, and special graph structures that power fundamental algorithms.

---

## Trees (트리)

### Concept (개념)

A **tree** is a connected graph with no cycles. Properties:

- N vertices → N-1 edges
- Exactly one path between any two vertices
- Removing any edge disconnects the graph
- Adding any edge creates a cycle

**Key terms:**
- **Root**: Top node (in rooted trees)
- **Leaf**: Node with no children
- **Parent/Child**: Hierarchical relationship
- **Depth**: Distance from root
- **Height**: Maximum depth

### Example

```python
from collections import defaultdict

class Tree:
    def __init__(self):
        self.children = defaultdict(list)
        self.parent = {}
        self.root = None

    def add_edge(self, parent, child):
        if self.root is None:
            self.root = parent
        self.children[parent].append(child)
        self.parent[child] = parent

    def get_depth(self, node):
        """Distance from root"""
        depth = 0
        while node != self.root:
            node = self.parent[node]
            depth += 1
        return depth

    def get_height(self, node=None):
        """Height of subtree rooted at node"""
        if node is None:
            node = self.root

        if not self.children[node]:  # Leaf
            return 0

        return 1 + max(self.get_height(c) for c in self.children[node])

    def is_leaf(self, node):
        return len(self.children[node]) == 0

    def get_leaves(self):
        return [n for n in self.children if self.is_leaf(n)]


# Build a tree
#       A
#      /|\
#     B C D
#    /|   |
#   E F   G

tree = Tree()
tree.add_edge("A", "B")
tree.add_edge("A", "C")
tree.add_edge("A", "D")
tree.add_edge("B", "E")
tree.add_edge("B", "F")
tree.add_edge("D", "G")

print("Tree structure:")
print(f"  Root: {tree.root}")
print(f"  Height: {tree.get_height()}")
print(f"  Depth of G: {tree.get_depth('G')}")
print(f"  Children of B: {tree.children['B']}")
print(f"  Leaves: {[n for n in ['A','B','C','D','E','F','G'] if tree.is_leaf(n)]}")

# Practical: File system is a tree
print("\nFile system example:")
fs = Tree()
fs.add_edge("/", "home")
fs.add_edge("/", "etc")
fs.add_edge("/", "var")
fs.add_edge("home", "user1")
fs.add_edge("home", "user2")
fs.add_edge("user1", "documents")
fs.add_edge("user1", "downloads")

print(f"  Path depth of 'documents': {fs.get_depth('documents')}")
```

---

## Binary Trees (이진 트리)

### Concept (개념)

A **binary tree** has at most 2 children per node (left and right).

**Types:**
- **Full binary tree**: Every node has 0 or 2 children
- **Complete binary tree**: All levels filled except possibly last (filled left to right)
- **Perfect binary tree**: All internal nodes have 2 children, all leaves at same level
- **Binary Search Tree (BST)**: Left < Parent < Right

**Properties:**
- Height h perfect tree has 2^(h+1) - 1 nodes
- At level k, at most 2^k nodes

### Example

```python
class BinaryTreeNode:
    def __init__(self, value):
        self.value = value
        self.left = None
        self.right = None


class BinaryTree:
    def __init__(self):
        self.root = None

    def insert_bst(self, value):
        """Insert into Binary Search Tree"""
        if self.root is None:
            self.root = BinaryTreeNode(value)
            return

        self._insert_recursive(self.root, value)

    def _insert_recursive(self, node, value):
        if value < node.value:
            if node.left is None:
                node.left = BinaryTreeNode(value)
            else:
                self._insert_recursive(node.left, value)
        else:
            if node.right is None:
                node.right = BinaryTreeNode(value)
            else:
                self._insert_recursive(node.right, value)

    def inorder(self, node=None, result=None):
        """Inorder traversal: Left, Root, Right (sorted for BST)"""
        if result is None:
            result = []
        if node is None:
            node = self.root

        if node:
            self.inorder(node.left, result)
            result.append(node.value)
            self.inorder(node.right, result)

        return result

    def preorder(self, node=None, result=None):
        """Preorder: Root, Left, Right"""
        if result is None:
            result = []
        if node is None:
            node = self.root

        if node:
            result.append(node.value)
            self.preorder(node.left, result)
            self.preorder(node.right, result)

        return result

    def postorder(self, node=None, result=None):
        """Postorder: Left, Right, Root"""
        if result is None:
            result = []
        if node is None:
            node = self.root

        if node:
            self.postorder(node.left, result)
            self.postorder(node.right, result)
            result.append(node.value)

        return result


# Build a BST
bst = BinaryTree()
values = [5, 3, 7, 2, 4, 6, 8]
for v in values:
    bst.insert_bst(v)

print("Binary Search Tree:")
print(f"  Inserted: {values}")
print(f"  Inorder (sorted):  {bst.inorder()}")
print(f"  Preorder:          {bst.preorder()}")
print(f"  Postorder:         {bst.postorder()}")

#       5
#      / \
#     3   7
#    / \ / \
#   2  4 6  8
```

---

## Spanning Trees (신장 트리)

### Concept (개념)

A **spanning tree** of a graph G includes all vertices with minimum edges (no cycles).

- If G has N vertices, spanning tree has N-1 edges
- Multiple spanning trees may exist
- **Minimum Spanning Tree (MST)**: Spanning tree with minimum total edge weight

**Algorithms:**
- **Kruskal's**: Sort edges by weight, add if no cycle
- **Prim's**: Grow tree from starting vertex

### Example

```python
class UnionFind:
    """Union-Find for cycle detection"""
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n

    def find(self, x):
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]

    def union(self, x, y):
        px, py = self.find(x), self.find(y)
        if px == py:
            return False  # Already in same set (would create cycle)

        if self.rank[px] < self.rank[py]:
            px, py = py, px
        self.parent[py] = px
        if self.rank[px] == self.rank[py]:
            self.rank[px] += 1
        return True


def kruskal_mst(num_vertices, edges):
    """
    Find Minimum Spanning Tree using Kruskal's algorithm.
    edges: list of (weight, u, v)
    """
    edges_sorted = sorted(edges)  # Sort by weight
    uf = UnionFind(num_vertices)
    mst = []
    total_weight = 0

    for weight, u, v in edges_sorted:
        if uf.union(u, v):  # Add edge if no cycle
            mst.append((u, v, weight))
            total_weight += weight

        if len(mst) == num_vertices - 1:
            break

    return mst, total_weight


# Example: Network with weighted edges
#     0
#    /|\
#   2 3 4
#  / \ |
# 1---3-2
#   5   1

edges = [
    (2, 0, 1),  # 0-1, weight 2
    (3, 0, 3),  # 0-3, weight 3
    (4, 0, 2),  # 0-2, weight 4
    (5, 1, 3),  # 1-3, weight 5
    (1, 2, 3),  # 2-3, weight 1
]

mst, total = kruskal_mst(4, edges)

print("Minimum Spanning Tree (Kruskal's):")
print(f"  Edges: {[(u, v) for u, v, w in mst]}")
print(f"  Weights: {[w for u, v, w in mst]}")
print(f"  Total weight: {total}")

# Application: Network design - connect all servers with minimum cable
print("\nApplication: Minimum cost to connect 4 servers")
print(f"  Selected connections: {mst}")
```

---

## DAG - Directed Acyclic Graph

### Concept (개념)

A **DAG** is a directed graph with no cycles.

**Properties:**
- Has at least one topological ordering
- Can represent dependencies
- Used in scheduling, build systems, spreadsheets

**Topological Sort:** Linear ordering where for every edge u→v, u comes before v.

### Example

```python
from collections import defaultdict, deque

def topological_sort(adj, num_vertices):
    """
    Kahn's algorithm for topological sort.
    Returns sorted list or None if cycle exists.
    """
    # Calculate in-degrees
    in_degree = [0] * num_vertices
    for u in adj:
        for v in adj[u]:
            in_degree[v] += 1

    # Start with nodes having no dependencies
    queue = deque([i for i in range(num_vertices) if in_degree[i] == 0])
    result = []

    while queue:
        node = queue.popleft()
        result.append(node)

        for neighbor in adj[node]:
            in_degree[neighbor] -= 1
            if in_degree[neighbor] == 0:
                queue.append(neighbor)

    if len(result) != num_vertices:
        return None  # Cycle detected

    return result


# Build system dependencies
# 0: main.c, 1: utils.c, 2: header.h, 3: config.h, 4: executable
# Dependencies: main.c needs header.h, config.h
#               utils.c needs header.h
#               executable needs main.c, utils.c

adj = defaultdict(list)
adj[2].append(0)  # header.h → main.c
adj[3].append(0)  # config.h → main.c
adj[2].append(1)  # header.h → utils.c
adj[0].append(4)  # main.c → executable
adj[1].append(4)  # utils.c → executable

names = ["main.c", "utils.c", "header.h", "config.h", "executable"]

order = topological_sort(adj, 5)

print("Build Order (Topological Sort):")
print(f"  Order: {[names[i] for i in order]}")
print("\nDependency explanation:")
for i, node in enumerate(order):
    deps = [names[j] for j in range(5) if node in adj[j]]
    print(f"  {i+1}. Build {names[node]}" + (f" (needs: {deps})" if deps else " (no deps)"))


# Spreadsheet calculation order
print("\nSpreadsheet example:")
# A1 = 10, A2 = 20, A3 = A1 + A2, A4 = A3 * 2
cells = {0: "A1=10", 1: "A2=20", 2: "A3=A1+A2", 3: "A4=A3*2"}
cell_deps = defaultdict(list)
cell_deps[0].append(2)  # A1 → A3
cell_deps[1].append(2)  # A2 → A3
cell_deps[2].append(3)  # A3 → A4

calc_order = topological_sort(cell_deps, 4)
print(f"  Calculation order: {[cells[i] for i in calc_order]}")
```

---

## Euler Path, Hamiltonian Path (오일러 경로, 해밀턴 경로)

### Concept (개념)

**Euler Path/Circuit:**
- Visits every **edge** exactly once
- Euler Path: Exists if 0 or 2 vertices have odd degree
- Euler Circuit: Exists if all vertices have even degree

**Hamiltonian Path/Circuit:**
- Visits every **vertex** exactly once
- No simple condition to check (NP-complete)
- Must try paths or use heuristics

### Example

```python
from collections import defaultdict

def has_euler_path(adj, num_vertices):
    """Check if undirected graph has Euler path"""
    odd_degree_count = 0

    for v in range(num_vertices):
        degree = len(adj[v])
        if degree % 2 == 1:
            odd_degree_count += 1

    # Euler path: 0 or 2 odd-degree vertices
    # Euler circuit: 0 odd-degree vertices
    return odd_degree_count == 0 or odd_degree_count == 2, odd_degree_count


def find_euler_path(adj, num_vertices):
    """Find Euler path using Hierholzer's algorithm"""
    # Make a copy of adjacency list
    adj_copy = defaultdict(list)
    for v in adj:
        adj_copy[v] = list(adj[v])

    # Find starting vertex (odd degree if exists, else any)
    start = 0
    for v in range(num_vertices):
        if len(adj_copy[v]) % 2 == 1:
            start = v
            break

    stack = [start]
    path = []

    while stack:
        v = stack[-1]
        if adj_copy[v]:
            u = adj_copy[v].pop()
            adj_copy[u].remove(v)  # Remove both directions
            stack.append(u)
        else:
            path.append(stack.pop())

    return path[::-1]


# Königsberg bridges problem (simplified)
#    A---B
#    |\ /|
#    | X |
#    |/ \|
#    C---D

adj = defaultdict(list)
# A-B, A-C, A-D, B-C, B-D, C-D (all connected)
edges = [(0,1), (0,2), (0,3), (1,2), (1,3), (2,3)]
for u, v in edges:
    adj[u].append(v)
    adj[v].append(u)

has_path, odd_count = has_euler_path(adj, 4)
print("Euler Path Check:")
print(f"  Odd degree vertices: {odd_count}")
print(f"  Has Euler path: {has_path}")
if has_path:
    path = find_euler_path(adj, 4)
    names = ['A', 'B', 'C', 'D']
    print(f"  Path: {' → '.join(names[v] for v in path)}")


# Hamiltonian path (brute force for small graphs)
def find_hamiltonian_path(adj, num_vertices):
    """Find Hamiltonian path (brute force)"""
    def backtrack(path, visited):
        if len(path) == num_vertices:
            return path[:]

        current = path[-1]
        for neighbor in adj[current]:
            if neighbor not in visited:
                visited.add(neighbor)
                path.append(neighbor)

                result = backtrack(path, visited)
                if result:
                    return result

                path.pop()
                visited.remove(neighbor)

        return None

    # Try starting from each vertex
    for start in range(num_vertices):
        path = backtrack([start], {start})
        if path:
            return path

    return None


print("\nHamiltonian Path:")
ham_path = find_hamiltonian_path(adj, 4)
if ham_path:
    names = ['A', 'B', 'C', 'D']
    print(f"  Path: {' → '.join(names[v] for v in ham_path)}")
```

---

## Summary

| Graph Type | Korean | Key Property |
|------------|--------|--------------|
| Tree | 트리 | Connected, no cycles, N-1 edges |
| Binary Tree | 이진 트리 | At most 2 children per node |
| Spanning Tree | 신장 트리 | All vertices, minimum edges |
| DAG | 방향 비순환 그래프 | Directed, no cycles |
| Euler Path | 오일러 경로 | Visit every edge once |
| Hamiltonian Path | 해밀턴 경로 | Visit every vertex once |

---

## Practice Problems

1. Given a tree with 10 nodes, how many edges does it have?

2. Determine if a graph with degrees [2, 2, 2, 3, 3] has an Euler path.

3. Write topological sort for: A→B, A→C, B→D, C→D.

4. Find the MST for a graph with edges: (1,2,3), (1,3,5), (2,3,2), (2,4,4), (3,4,1).
