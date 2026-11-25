# Tree Diameter (Double BFS) Usage Guide

A C++ implementation of the **Double BFS** algorithm for finding the **diameter** (longest shortest-path distance in edges) and the two **diameter endpoints** $(A, B)$ of an unweighted tree in linear **$O(V)$** time.

---

## Complexity Overview

| Function | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`bfs(src)`** | $O(V)$ | $O(V)$ | Runs BFS from `src` and returns `{farthestNode, distance}`. |
| **`diameter()`** | $O(V)$ | $O(V)$ | Runs two BFS passes: `0 -> A` (first endpoint), then `A -> B` (second endpoint & diameter length). |

---

## Implementation

```cpp
vector<vector<int>> adj;

pair<int, int> bfs(int src) {
    int n = (int)adj.size();
    vector<int> dist(n, -1);

    queue<int> q;
    q.push(src);
    dist[src] = 0;

    while (!q.empty()) {
        int u = q.front();
        q.pop();

        for (int v : adj[u]) {
            if (dist[v] == -1) {
                dist[v] = dist[u] + 1;
                q.push(v);
            }
        }
    }

    int far = src;
    for (int i = 0; i < n; i++)
        if (dist[i] > dist[far])
            far = i;

    return {far, dist[far]};
}

int diameter() {
    auto [A, _] = bfs(0);
    auto [B, diam] = bfs(A);
    return diam;
}
```

---

## Usage

### 1. Build 0-Indexed Tree & Compute Diameter
Initialize the global adjacency list `adj` with size `n` (vertices `0..n-1`):
```cpp
int n = 5;
adj.assign(n, {});

auto addEdge = [&](int u, int v) {
    adj[u].push_back(v);
    adj[v].push_back(u);
};

addEdge(0, 1);
addEdge(1, 2);
addEdge(1, 3);
addEdge(3, 4);

int d = diameter(); // 3 (path: 2 - 1 - 3 - 4)
```

### 2. Retrieve Diameter Endpoints $(A, B)$ & Tree Centers
Many tree problems require the actual endpoints $A$ and $B$ (e.g., to find the eccentricity $\max(\text{dist}(u, A), \text{dist}(u, B))$ of every node $u$):
```cpp
auto [A, _]    = bfs(0);
auto [B, diam] = bfs(A);
// A and B are guaranteed to be the two ends of a longest path in the tree
```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste adj, bfs, and diameter here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;
    adj.assign(n, {});

    for (int i = 0; i < n - 1; i++) {
        int u, v;
        cin >> u >> v;
        --u; --v; // Convert 1-based input to 0-based
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    cout << diameter() << "\n";

    return 0;
}
```

---

## Important Notes

1. **0-Indexed Vertices (`0..n-1`):** `diameter()` starts from node `0` (`bfs(0)`). If your input graph is 1-indexed (`1..n`), subtract `1` from each vertex when reading edges (or resize `adj` to `n + 1` and start at `bfs(1)`).
2. **Resetting `adj` Between Test Cases:** Always call `adj.assign(n, {});` at the start of every test case so edges from previous test cases do not persist.
3. **Single-Node Tree (`n = 1`):** Works safely—`bfs(0)` returns `{0, 0}` and `diameter()` returns `0`.
4. **Trees Only (Unweighted):** Double BFS is exact for **unweighted trees** (and weighted trees with **non-negative** edge weights using Dijkstra/DFS instead of BFS), but does **not** find the diameter of general cyclic graphs.