# Chapter 07. Graph Basics (그래프 기초)

> Understanding graphs as the fundamental data structure for modeling relationships, networks, and connections in computer science.

---

## What is a Graph? (그래프란?)

### Concept (개념)

A **graph** (그래프) G = (V, E) consists of:
- **V** (정점/노드 집합): A set of **vertices** (or nodes)
- **E** (간선 집합): A set of **edges** connecting pairs of vertices

Graphs model relationships: social networks, road maps, web links, dependencies, and more.

**Basic terminology:**
- **Adjacent** (인접): Two vertices connected by an edge
- **Incident** (부속): An edge is incident to its endpoints
- **Neighbor** (이웃): Adjacent vertices

### Example

```python
# Simple graph representation
class Graph:
    def __init__(self):
        self.vertices = set()
        self.edges = set()

    def add_vertex(self, v):
        self.vertices.add(v)

    def add_edge(self, u, v):
        self.vertices.add(u)
        self.vertices.add(v)
        self.edges.add((u, v))
        self.edges.add((v, u))  # Undirected

    def are_adjacent(self, u, v):
        return (u, v) in self.edges

    def neighbors(self, v):
        return {u for (a, u) in self.edges if a == v}

# Create a simple graph
g = Graph()
g.add_edge("A", "B")
g.add_edge("B", "C")
g.add_edge("A", "C")

print(f"Vertices: {g.vertices}")  # {'A', 'B', 'C'}
print(f"A adjacent to B: {g.are_adjacent('A', 'B')}")  # True
print(f"Neighbors of B: {g.neighbors('B')}")  # {'A', 'C'}

# Visual representation:
#    A --- B
#     \   /
#      \ /
#       C
```

---

## Undirected vs Directed Graphs (무향 그래프 vs 유향 그래프)

### Concept (개념)

**Undirected Graph** (무향 그래프):
- Edges have no direction
- Edge {u, v} = {v, u}
- Examples: Facebook friendships, road networks

**Directed Graph (Digraph)** (유향 그래프):
- Edges have direction (arrows)
- Edge (u, v) ≠ (v, u)
- Examples: Twitter follows, web links, dependencies

### Example

```python
# Undirected graph
class UndirectedGraph:
    def __init__(self):
        self.adj = {}

    def add_edge(self, u, v):
        if u not in self.adj: self.adj[u] = set()
        if v not in self.adj: self.adj[v] = set()
        self.adj[u].add(v)
        self.adj[v].add(u)  # Symmetric!

# Directed graph
class DirectedGraph:
    def __init__(self):
        self.adj = {}

    def add_edge(self, u, v):
        if u not in self.adj: self.adj[u] = set()
        if v not in self.adj: self.adj[v] = set()
        self.adj[u].add(v)  # Only u → v, not v → u

# Example: Social media
undirected = UndirectedGraph()  # Facebook
undirected.add_edge("Alice", "Bob")
# Alice friends with Bob = Bob friends with Alice

directed = DirectedGraph()  # Twitter
directed.add_edge("Alice", "Bob")  # Alice follows Bob
# Does NOT mean Bob follows Alice

print(f"Undirected: {undirected.adj}")
# {'Alice': {'Bob'}, 'Bob': {'Alice'}}

print(f"Directed: {directed.adj}")
# {'Alice': {'Bob'}, 'Bob': set()}
```

---

## Weighted Graphs (가중 그래프)

### Concept (개념)

A **weighted graph** (가중 그래프) assigns a numerical value (weight) to each edge.

**Applications:**
- Road networks (distance, travel time)
- Network routing (bandwidth, latency)
- Cost optimization problems
- Similarity/distance measures

### Example

```python
class WeightedGraph:
    def __init__(self, directed=False):
        self.adj = {}
        self.directed = directed

    def add_edge(self, u, v, weight):
        if u not in self.adj: self.adj[u] = {}
        if v not in self.adj: self.adj[v] = {}

        self.adj[u][v] = weight
        if not self.directed:
            self.adj[v][u] = weight

    def get_weight(self, u, v):
        return self.adj.get(u, {}).get(v, float('inf'))

# City distances example
cities = WeightedGraph()
cities.add_edge("Seoul", "Busan", 325)
cities.add_edge("Seoul", "Daejeon", 140)
cities.add_edge("Daejeon", "Busan", 200)

print(f"Seoul to Busan: {cities.get_weight('Seoul', 'Busan')} km")
print(f"Seoul to Daejeon: {cities.get_weight('Seoul', 'Daejeon')} km")

# Visual:
#         140
# Seoul -------- Daejeon
#    \             /
#  325\           /200
#      \         /
#        Busan

# Question: Direct or via Daejeon?
# Direct: 325 km
# Via Daejeon: 140 + 200 = 340 km
```

---

## Adjacency Matrix (인접 행렬)

### Concept (개념)

An **adjacency matrix** (인접 행렬) is a 2D array representation where:
- A[i][j] = 1 if edge exists from vertex i to vertex j
- A[i][j] = 0 otherwise
- For weighted graphs: A[i][j] = weight (or ∞ if no edge)

**Properties:**
- Space: O(V²)
- Edge lookup: O(1)
- All neighbors: O(V)
- Best for dense graphs (many edges)
- Symmetric matrix for undirected graphs

### Example

```python
import numpy as np

class AdjacencyMatrix:
    def __init__(self, vertices):
        self.vertices = list(vertices)
        self.index = {v: i for i, v in enumerate(vertices)}
        n = len(vertices)
        self.matrix = np.zeros((n, n), dtype=int)

    def add_edge(self, u, v, directed=False):
        i, j = self.index[u], self.index[v]
        self.matrix[i][j] = 1
        if not directed:
            self.matrix[j][i] = 1

    def has_edge(self, u, v):
        i, j = self.index[u], self.index[v]
        return self.matrix[i][j] == 1

    def display(self):
        print("   " + " ".join(self.vertices))
        for v in self.vertices:
            i = self.index[v]
            row = " ".join(str(x) for x in self.matrix[i])
            print(f"{v}  {row}")

# Example
g = AdjacencyMatrix(["A", "B", "C", "D"])
g.add_edge("A", "B")
g.add_edge("A", "C")
g.add_edge("B", "C")
g.add_edge("C", "D")

g.display()
#    A B C D
# A  0 1 1 0
# B  1 0 1 0
# C  1 1 0 1
# D  0 0 1 0

print(f"Edge A-B exists: {g.has_edge('A', 'B')}")  # True
print(f"Edge A-D exists: {g.has_edge('A', 'D')}")  # False
```

---

## Adjacency List (인접 리스트)

### Concept (개념)

An **adjacency list** (인접 리스트) stores, for each vertex, a list of its neighbors.

**Properties:**
- Space: O(V + E)
- Edge lookup: O(degree of vertex)
- All neighbors: O(1) to access list
- Best for sparse graphs (few edges)
- More memory efficient for most real-world graphs

### Example

```python
from collections import defaultdict

class AdjacencyList:
    def __init__(self, directed=False):
        self.adj = defaultdict(list)
        self.directed = directed

    def add_edge(self, u, v):
        self.adj[u].append(v)
        if not self.directed:
            self.adj[v].append(u)

    def has_edge(self, u, v):
        return v in self.adj[u]

    def neighbors(self, v):
        return self.adj[v]

    def display(self):
        for v in sorted(self.adj.keys()):
            print(f"{v}: {self.adj[v]}")

# Same graph as before
g = AdjacencyList()
g.add_edge("A", "B")
g.add_edge("A", "C")
g.add_edge("B", "C")
g.add_edge("C", "D")

g.display()
# A: ['B', 'C']
# B: ['A', 'C']
# C: ['A', 'B', 'D']
# D: ['C']

# Space comparison for sparse graph:
# Matrix: 4×4 = 16 cells
# List: 4 + 4 + 4 + 4 + 1 = 17 entries (but for larger sparse graphs, much better)
```

---

## Degree (차수)

### Concept (개념)

The **degree** (차수) of a vertex is the number of edges incident to it.

**Undirected graphs:**
- degree(v) = number of edges connected to v
- Self-loop contributes 2 to degree
- Sum of all degrees = 2 × |E| (Handshaking lemma)

**Directed graphs:**
- **In-degree** (진입차수): Number of incoming edges
- **Out-degree** (진출차수): Number of outgoing edges
- Sum of in-degrees = Sum of out-degrees = |E|

### Example

```python
class DegreeAnalysis:
    def __init__(self, directed=False):
        self.adj = defaultdict(list)
        self.directed = directed

    def add_edge(self, u, v):
        self.adj[u].append(v)
        if not self.directed:
            self.adj[v].append(u)
        elif v not in self.adj:
            self.adj[v] = []  # Ensure v exists

    def degree(self, v):
        return len(self.adj[v])

    def in_degree(self, v):
        """Only for directed graphs"""
        return sum(1 for u in self.adj for neighbor in self.adj[u] if neighbor == v)

    def out_degree(self, v):
        """Only for directed graphs"""
        return len(self.adj[v])

# Undirected example
g = DegreeAnalysis(directed=False)
g.add_edge("A", "B")
g.add_edge("A", "C")
g.add_edge("B", "C")
g.add_edge("C", "D")

print("Undirected degrees:")
for v in ["A", "B", "C", "D"]:
    print(f"  degree({v}) = {g.degree(v)}")
# A: 2, B: 2, C: 3, D: 1

# Verify handshaking lemma
total_degree = sum(g.degree(v) for v in g.adj)
num_edges = 4
print(f"Sum of degrees: {total_degree}, 2×|E|: {2 * num_edges}")  # Both 8

# Directed example (Twitter follow graph)
d = DegreeAnalysis(directed=True)
d.add_edge("Alice", "Bob")
d.add_edge("Alice", "Charlie")
d.add_edge("Bob", "Charlie")
d.add_edge("Charlie", "Alice")

print("\nDirected degrees:")
for v in ["Alice", "Bob", "Charlie"]:
    print(f"  {v}: in={d.in_degree(v)}, out={d.out_degree(v)}")
# Alice: in=1, out=2 (1 follower, follows 2)
# Bob: in=1, out=1
# Charlie: in=2, out=1 (2 followers, follows 1)
```

---

## Graph Types (그래프의 종류)

### Concept (개념)

| Graph Type | Korean | Description |
|------------|--------|-------------|
| **Simple** | 단순 그래프 | No self-loops, no multiple edges |
| **Multigraph** | 다중 그래프 | Multiple edges between same vertices |
| **Complete** | 완전 그래프 | Every pair connected (Kₙ) |
| **Bipartite** | 이분 그래프 | Vertices split into two disjoint sets |
| **Regular** | 정규 그래프 | All vertices have same degree |
| **Planar** | 평면 그래프 | Can be drawn without edge crossings |

### Example

```python
def create_complete_graph(n):
    """Create complete graph Kₙ with n vertices"""
    g = AdjacencyList()
    vertices = list(range(n))
    for i in vertices:
        for j in vertices:
            if i < j:
                g.add_edge(i, j)
    return g

# K₄: Complete graph with 4 vertices
k4 = create_complete_graph(4)
k4.display()
# Every vertex connected to every other
# Number of edges in Kₙ = n(n-1)/2

def is_bipartite(adj):
    """Check if graph is bipartite using BFS coloring"""
    color = {}

    for start in adj:
        if start in color:
            continue

        queue = [start]
        color[start] = 0

        while queue:
            node = queue.pop(0)
            for neighbor in adj[node]:
                if neighbor not in color:
                    color[neighbor] = 1 - color[node]
                    queue.append(neighbor)
                elif color[neighbor] == color[node]:
                    return False

    return True

# Bipartite example (students and courses)
bipartite = defaultdict(list)
bipartite["Alice"].append("Math")
bipartite["Math"].append("Alice")
bipartite["Alice"].append("CS")
bipartite["CS"].append("Alice")
bipartite["Bob"].append("Math")
bipartite["Math"].append("Bob")

print(f"Is bipartite: {is_bipartite(bipartite)}")  # True
```

---

## Graph Representation Comparison (표현 방식 비교)

### Concept (개념)

| Operation | Adjacency Matrix | Adjacency List |
|-----------|------------------|----------------|
| Space | O(V²) | O(V + E) |
| Add edge | O(1) | O(1) |
| Remove edge | O(1) | O(degree) |
| Check edge | O(1) | O(degree) |
| Get neighbors | O(V) | O(1) |
| Iterate all edges | O(V²) | O(E) |

**Choose Matrix when:**
- Graph is dense (E ≈ V²)
- Frequent edge existence queries
- Graph modifications are rare

**Choose List when:**
- Graph is sparse (E << V²)
- Need to iterate neighbors often
- Memory is a concern

### Example

```python
import time
import random

def benchmark_representations(num_vertices, edge_probability):
    """Compare performance of matrix vs list"""

    # Generate random edges
    edges = []
    for i in range(num_vertices):
        for j in range(i + 1, num_vertices):
            if random.random() < edge_probability:
                edges.append((i, j))

    print(f"Vertices: {num_vertices}, Edges: {len(edges)}")
    print(f"Density: {len(edges) / (num_vertices * (num_vertices-1) / 2):.2%}")

    # Build adjacency matrix
    import numpy as np
    start = time.time()
    matrix = np.zeros((num_vertices, num_vertices), dtype=int)
    for u, v in edges:
        matrix[u][v] = matrix[v][u] = 1
    matrix_build_time = time.time() - start

    # Build adjacency list
    start = time.time()
    adj_list = defaultdict(list)
    for u, v in edges:
        adj_list[u].append(v)
        adj_list[v].append(u)
    list_build_time = time.time() - start

    print(f"Matrix build: {matrix_build_time:.4f}s")
    print(f"List build: {list_build_time:.4f}s")

    # Memory comparison (approximate)
    matrix_memory = num_vertices * num_vertices * 8  # bytes
    list_memory = sum(len(v) * 8 for v in adj_list.values()) + num_vertices * 8
    print(f"Matrix memory: ~{matrix_memory // 1024} KB")
    print(f"List memory: ~{list_memory // 1024} KB")

# Sparse graph
benchmark_representations(1000, 0.01)
# List is much better for sparse graphs
```

---

## Summary

| Concept | Korean | Key Point |
|---------|--------|-----------|
| Graph | 그래프 | G = (V, E) |
| Vertex/Node | 정점/노드 | Points in graph |
| Edge | 간선 | Connections between vertices |
| Undirected | 무향 | Edges have no direction |
| Directed | 유향 | Edges have direction (arrows) |
| Weighted | 가중 | Edges have numerical values |
| Adjacency Matrix | 인접 행렬 | 2D array, O(V²) space |
| Adjacency List | 인접 리스트 | List of neighbors, O(V+E) space |
| Degree | 차수 | Number of edges on vertex |
| In-degree/Out-degree | 진입/진출차수 | For directed graphs |
