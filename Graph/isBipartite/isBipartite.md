# Bipartite Graph Check (2-Coloring BFS) Usage Guide

A linear-time C++ implementation of **BFS 2-Coloring** (`isBipartite`) that determines whether an undirected graph (including disconnected graphs with multiple components) is **bipartite** (contains no odd-length cycles) in **$O(V + E)$** time.

---

## Complexity Overview

| Function | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`isBipartite()`** | $O(V + E)$ | $O(V)$ | Assigns colors `0` and `1` across all connected components (`0..n-1`); returns `false` on any same-color edge conflict. |

---

## Implementation

```cpp
int n;
vector<vector<int>> adj;

bool isBipartite() {
    vector<int> color(n, -1);

    for (int i = 0; i < n; i++) {
        if (color[i] != -1) continue;

        queue<int> q;
        q.push(i);
        color[i] = 0;

        while (!q.empty()) {
            int u = q.front();
            q.pop();

            for (int v : adj[u]) {
                if (color[v] == -1) {
                    color[v] = color[u] ^ 1;
                    q.push(v);
                }
                else if (color[v] == color[u]) {
                    return false;
                }
            }
        }
    }

    return true;
}
```

---

## Usage

### 1. Initialize 0-Indexed Graph & Check Bipartiteness
Set global `n` and build the **0-indexed** undirected adjacency list `adj` (`0..n-1`), then call `isBipartite()`:
```cpp
n = 4;
adj.assign(n, {});

auto addEdge = [&](int u, int v) {
    adj[u].push_back(v);
    adj[v].push_back(u);
};

// Even cycle: 0 - 1 - 2 - 3 - 0 (Bipartite)
addEdge(0, 1);
addEdge(1, 2);
addEdge(2, 3);
addEdge(3, 0);

bool ok = isBipartite(); // true
```

### 2. Extracting Partition Sizes or Valid 2-Coloring
When a problem requires the actual vertex partition (`color[u] == 0` vs. `color[u] == 1`) or the maximum/minimum partition size per connected component:
```cpp
vector<int> color(n, -1);
for (int i = 0; i < n; i++) {
    if (color[i] != -1) continue;
    int cnt[2] = {0, 0};
    queue<int> q;
    q.push(i);
    color[i] = 0;
    cnt[0]++;

    while (!q.empty()) {
        int u = q.front();
        q.pop();
        for (int v : adj[u]) {
            if (color[v] == -1) {
                color[v] = color[u] ^ 1;
                cnt[color[v]]++;
                q.push(v);
            } else if (color[v] == color[u]) {
                // Not bipartite
            }
        }
    }
    // Component partition sizes are cnt[0] and cnt[1]
}
```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste n, adj, and isBipartite here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int m;
    cin >> n >> m;
    adj.assign(n, {});

    for (int i = 0; i < m; i++) {
        int u, v;
        cin >> u >> v;
        --u; --v; // Convert 1-based input to 0-based
        adj[u].push_back(v);
        adj[v].push_back(u);
    }

    cout << (isBipartite() ? "YES\n" : "NO\n");

    return 0;
}
```

---

## Important Notes

1. **0-Indexed Vertices (`0..n-1`):** `isBipartite()` iterates `for (int i = 0; i < n; i++)`. If your input vertices are 1-indexed (`1..n`), subtract `1` when reading edges (or allocate `color(n + 1, -1)` and loop `1..n`).
2. **Self-Loops Handled Automatically:** If a self-loop `(u, u)` exists in `adj[u]`, `color[u] == color[u]` triggers immediately and returns `false` (a self-loop is a cycle of length 1, which is odd).
3. **Disconnected Components & Isolated Vertices:** The outer loop checks every unvisited vertex `0..n-1`, so disconnected components and isolated vertices (`adj[i].empty()`) are colored cleanly without missing any subgraph.
4. **Resetting `adj` Between Test Cases:** Always call `adj.assign(n, {});` at the beginning of each test case so stale edges from earlier test cases do not corrupt the graph.