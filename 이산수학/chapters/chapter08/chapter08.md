# Chapter 08. Graph Traversal and Paths (그래프 탐색과 경로)

> Exploring systematic methods to visit all vertices and find paths in graphs, fundamental to countless algorithms.

---

## Paths (경로)

### Concept (개념)

A **path** (경로) in a graph is a sequence of vertices where each adjacent pair is connected by an edge.

**Types:**
- **Simple path** (단순 경로): No repeated vertices
- **Walk** (보행): Vertices and edges may repeat
- **Path length** (경로 길이): Number of edges in the path

**Notation:** Path from v₀ to vₖ: v₀ → v₁ → v₂ → ... → vₖ

### Example

```python
from collections import defaultdict

class Graph:
    def __init__(self):
        self.adj = defaultdict(list)

    def add_edge(self, u, v):
        self.adj[u].append(v)
        self.adj[v].append(u)

    def find_all_paths(self, start, end, path=None):
        """Find all simple paths from start to end"""
        if path is None:
            path = []

        path = path + [start]

        if start == end:
            return [path]

        paths = []
        for neighbor in self.adj[start]:
            if neighbor not in path:  # Avoid cycles (simple path)
                new_paths = self.find_all_paths(neighbor, end, path)
                paths.extend(new_paths)

        return paths

# Example graph
g = Graph()
g.add_edge("A", "B")
g.add_edge("A", "C")
g.add_edge("B", "C")
g.add_edge("B", "D")
g.add_edge("C", "D")

# Find all paths from A to D
paths = g.find_all_paths("A", "D")
print("All paths from A to D:")
for p in paths:
    print(f"  {' -> '.join(p)} (length: {len(p) - 1})")

# Output:
# A -> B -> D (length: 2)
# A -> B -> C -> D (length: 3)
# A -> C -> B -> D (length: 3)
# A -> C -> D (length: 2)
```

---

## Cycles (순환/사이클)

### Concept (개념)

A **cycle** (순환/사이클) is a path that starts and ends at the same vertex with at least one edge.

**Types:**
- **Simple cycle**: No repeated vertices except start/end
- **Self-loop**: Edge from a vertex to itself
- **Acyclic graph** (비순환 그래프): Graph with no cycles

**Applications:**
- Deadlock detection (circular wait)
- Dependency checking (circular dependencies)
- Infinite loop detection

### Example

```python
def has_cycle_undirected(adj):
    """Detect cycle in undirected graph using DFS"""
    visited = set()

    def dfs(node, parent):
        visited.add(node)

        for neighbor in adj[node]:
            if neighbor not in visited:
                if dfs(neighbor, node):
                    return True
            elif neighbor != parent:
                # Found a back edge (cycle!)
                return True

        return False

    # Check all components
    for node in adj:
        if node not in visited:
            if dfs(node, None):
                return True

    return False

def has_cycle_directed(adj):
    """Detect cycle in directed graph using DFS with colors"""
    WHITE, GRAY, BLACK = 0, 1, 2
    color = {node: WHITE for node in adj}

    def dfs(node):
        color[node] = GRAY  # Currently exploring

        for neighbor in adj[node]:
            if color[neighbor] == GRAY:
                # Back edge to ancestor = cycle!
                return True
            if color[neighbor] == WHITE:
                if dfs(neighbor):
                    return True

        color[node] = BLACK  # Done exploring
        return False

    for node in adj:
        if color[node] == WHITE:
            if dfs(node):
                return True

    return False

# Example: Dependency graph
dependencies = defaultdict(list)
dependencies["A"].append("B")  # A depends on B
dependencies["B"].append("C")
dependencies["C"].append("A")  # Circular dependency!

print(f"Has circular dependency: {has_cycle_directed(dependencies)}")  # True
```

---

## Connected Graphs (연결 그래프)

### Concept (개념)

**Undirected graph:**
- **Connected** (연결): Path exists between every pair of vertices
- **Connected component** (연결 요소): Maximal connected subgraph

**Directed graph:**
- **Strongly connected** (강연결): Path exists in both directions between every pair
- **Weakly connected** (약연결): Connected if we ignore edge directions

### Example

```python
def find_connected_components(adj):
    """Find all connected components in undirected graph"""
    visited = set()
    components = []

    def dfs(node, component):
        visited.add(node)
        component.append(node)
        for neighbor in adj[node]:
            if neighbor not in visited:
                dfs(neighbor, component)

    for node in adj:
        if node not in visited:
            component = []
            dfs(node, component)
            components.append(component)

    return components

# Graph with 2 components
g = defaultdict(list)
# Component 1
g["A"].extend(["B", "C"]); g["B"].extend(["A", "C"]); g["C"].extend(["A", "B"])
# Component 2
g["X"].extend(["Y"]); g["Y"].extend(["X", "Z"]); g["Z"].extend(["Y"])

components = find_connected_components(g)
print(f"Number of components: {len(components)}")
for i, comp in enumerate(components):
    print(f"  Component {i + 1}: {comp}")

# Output:
# Number of components: 2
# Component 1: ['A', 'B', 'C']
# Component 2: ['X', 'Y', 'Z']

# Practical: Finding isolated groups in social network
# Each component is a separate community
```

---

## Depth-First Search (깊이 우선 탐색, DFS)

### Concept (개념)

**DFS** explores as far as possible along each branch before backtracking.

**Properties:**
- Uses a **stack** (implicit via recursion or explicit)
- Space: O(V) for the stack
- Time: O(V + E)
- Finds **any** path, not necessarily shortest

**Applications:**
- Cycle detection
- Topological sorting
- Finding connected components
- Maze solving

### Example

```python
def dfs_recursive(adj, start, visited=None):
    """Recursive DFS implementation"""
    if visited is None:
        visited = set()

    visited.add(start)
    print(f"Visiting: {start}")

    for neighbor in adj[start]:
        if neighbor not in visited:
            dfs_recursive(adj, neighbor, visited)

    return visited

def dfs_iterative(adj, start):
    """Iterative DFS using explicit stack"""
    visited = set()
    stack = [start]
    order = []

    while stack:
        node = stack.pop()  # LIFO

        if node not in visited:
            visited.add(node)
            order.append(node)

            # Add neighbors (reverse for consistent order)
            for neighbor in reversed(adj[node]):
                if neighbor not in visited:
                    stack.append(neighbor)

    return order

# Example graph
g = defaultdict(list)
g["A"].extend(["B", "C"])
g["B"].extend(["A", "D", "E"])
g["C"].extend(["A", "F"])
g["D"].extend(["B"])
g["E"].extend(["B", "F"])
g["F"].extend(["C", "E"])

print("DFS order:", dfs_iterative(g, "A"))
# Possible output: ['A', 'B', 'D', 'E', 'F', 'C']

# Visual:
#      A
#     / \
#    B   C
#   / \   \
#  D   E - F
```

---

## Breadth-First Search (너비 우선 탐색, BFS)

### Concept (개념)

**BFS** explores all neighbors at the current depth before moving deeper.

**Properties:**
- Uses a **queue**
- Space: O(V) for the queue
- Time: O(V + E)
- Finds **shortest path** in unweighted graphs

**Applications:**
- Shortest path (unweighted)
- Level-order traversal
- Finding nearest neighbors
- Web crawling

### Example

```python
from collections import deque

def bfs(adj, start):
    """BFS implementation"""
    visited = set([start])
    queue = deque([start])
    order = []

    while queue:
        node = queue.popleft()  # FIFO
        order.append(node)

        for neighbor in adj[node]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)

    return order

def bfs_shortest_path(adj, start, end):
    """Find shortest path using BFS"""
    if start == end:
        return [start]

    visited = set([start])
    queue = deque([(start, [start])])  # (node, path)

    while queue:
        node, path = queue.popleft()

        for neighbor in adj[node]:
            if neighbor == end:
                return path + [neighbor]

            if neighbor not in visited:
                visited.add(neighbor)
                queue.append((neighbor, path + [neighbor]))

    return None  # No path exists

# Same graph
g = defaultdict(list)
g["A"].extend(["B", "C"])
g["B"].extend(["A", "D", "E"])
g["C"].extend(["A", "F"])
g["D"].extend(["B"])
g["E"].extend(["B", "F"])
g["F"].extend(["C", "E"])

print("BFS order:", bfs(g, "A"))
# Output: ['A', 'B', 'C', 'D', 'E', 'F']

path = bfs_shortest_path(g, "A", "F")
print(f"Shortest path A to F: {path}")  # ['A', 'C', 'F'] (length 2)

# Compare: DFS might find ['A', 'B', 'E', 'F'] (length 3)
```

---

## DFS vs BFS Comparison (DFS vs BFS 비교)

### Concept (개념)

| Aspect | DFS | BFS |
|--------|-----|-----|
| Data Structure | Stack (LIFO) | Queue (FIFO) |
| Space | O(h) height | O(w) width |
| Shortest Path | No | Yes (unweighted) |
| Complete | Yes | Yes |
| Memory | Less for deep/sparse | Less for wide/shallow |
| Use Case | Maze, topology | Shortest path, levels |

### Example

```python
# Visual comparison on a tree
#        1
#       /|\
#      2 3 4
#     /|   |
#    5 6   7

tree = defaultdict(list)
tree[1].extend([2, 3, 4])
tree[2].extend([5, 6])
tree[4].extend([7])

print("DFS:", dfs_iterative(tree, 1))
# Goes deep first: [1, 2, 5, 6, 3, 4, 7]

print("BFS:", bfs(tree, 1))
# Goes level by level: [1, 2, 3, 4, 5, 6, 7]

# When to use which:
#
# Use DFS when:
# - Finding any path is okay
# - Exploring all possibilities (backtracking)
# - Memory is limited and graph is deep
#
# Use BFS when:
# - Need shortest path (unweighted)
# - Finding nearest neighbors
# - Level-order traversal needed
```

---

## Shortest Path Concept (최단 경로 개념)

### Concept (개념)

Finding the **shortest path** (최단 경로) between vertices is a fundamental problem.

**Unweighted graphs:**
- BFS finds the shortest path
- Path length = number of edges

**Weighted graphs:**
- Dijkstra's algorithm (non-negative weights)
- Bellman-Ford (negative weights allowed)
- Floyd-Warshall (all pairs)

### Example

```python
import heapq

def dijkstra(adj_weighted, start, end):
    """
    Dijkstra's algorithm for shortest path in weighted graph.
    adj_weighted[u] = [(v, weight), ...]
    """
    distances = {node: float('inf') for node in adj_weighted}
    distances[start] = 0
    previous = {node: None for node in adj_weighted}

    # Min-heap: (distance, node)
    heap = [(0, start)]

    while heap:
        current_dist, current = heapq.heappop(heap)

        if current == end:
            break

        if current_dist > distances[current]:
            continue

        for neighbor, weight in adj_weighted[current]:
            distance = current_dist + weight

            if distance < distances[neighbor]:
                distances[neighbor] = distance
                previous[neighbor] = current
                heapq.heappush(heap, (distance, neighbor))

    # Reconstruct path
    path = []
    node = end
    while node is not None:
        path.append(node)
        node = previous[node]
    path.reverse()

    return distances[end], path

# Weighted graph example
g = defaultdict(list)
g["A"].extend([("B", 4), ("C", 2)])
g["B"].extend([("A", 4), ("C", 1), ("D", 5)])
g["C"].extend([("A", 2), ("B", 1), ("D", 8)])
g["D"].extend([("B", 5), ("C", 8)])

distance, path = dijkstra(g, "A", "D")
print(f"Shortest distance A to D: {distance}")
print(f"Path: {' -> '.join(path)}")

# Output:
# Shortest distance A to D: 9
# Path: A -> C -> B -> D

# Visual:
#    A --(4)-- B
#    |         | \
#   (2)       (1) (5)
#    |         |   \
#    C --(8)-- D
```

---

## Applications (응용)

### Concept (개념)

Graph traversal and path algorithms solve real-world problems:

1. **Social Networks**: Friend suggestions (BFS for close connections)
2. **Navigation**: GPS routing (Dijkstra/A*)
3. **Web Crawling**: Discovering pages (BFS for breadth)
4. **Dependency Resolution**: Build order (DFS for topological sort)
5. **Game AI**: Pathfinding (A* algorithm)

### Example

```python
# Application 1: Six Degrees of Separation
def degrees_of_separation(social_graph, person1, person2):
    """Find the shortest connection path between two people"""
    path = bfs_shortest_path(social_graph, person1, person2)
    if path:
        return len(path) - 1, path
    return -1, []

# Application 2: Web crawler structure
def web_crawler_bfs(start_url, max_pages):
    """BFS-based web crawler (conceptual)"""
    visited = set()
    queue = deque([start_url])
    pages_crawled = []

    while queue and len(pages_crawled) < max_pages:
        url = queue.popleft()
        if url in visited:
            continue

        visited.add(url)
        pages_crawled.append(url)

        # In real crawler: fetch page and extract links
        # links = fetch_and_extract_links(url)
        # for link in links:
        #     if link not in visited:
        #         queue.append(link)

    return pages_crawled

# Application 3: Finding reachable states
def find_reachable_states(initial_state, get_next_states):
    """Find all states reachable from initial state"""
    visited = set()
    stack = [initial_state]

    while stack:
        state = stack.pop()
        if state in visited:
            continue
        visited.add(state)

        for next_state in get_next_states(state):
            if next_state not in visited:
                stack.append(next_state)

    return visited
```

---

## Summary

| Concept | Korean | Key Point |
|---------|--------|-----------|
| Path | 경로 | Sequence of connected vertices |
| Simple Path | 단순 경로 | No repeated vertices |
| Cycle | 순환/사이클 | Path that returns to start |
| Connected | 연결 | Path exists between all pairs |
| DFS | 깊이 우선 탐색 | Stack, go deep first |
| BFS | 너비 우선 탐색 | Queue, level by level |
| Shortest Path | 최단 경로 | Minimum edges/weight |
| Dijkstra | 다익스트라 | Weighted shortest path |
