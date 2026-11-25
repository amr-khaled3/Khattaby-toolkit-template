# DFS Topological Sort Usage Guide

A linear-time C++ implementation of **DFS-based Topological Sorting** for Directed Acyclic Graphs (DAGs). It traverses unvisited vertices (`0..n-1`), pushes each vertex onto `topo` upon finishing all its outgoing neighbors (post-order), and reverses `topo` so every directed edge $u \to v$ satisfies `pos[u] < pos[v]`.

---

## Complexity Overview

| Function | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`dfs(u)`** | $O(V + E)$ total | $O(V)$ recursion stack | Visits reachable unvisited vertices and records post-order finish times. |
| **`topoSort()`** | $O(V + E)$ | $O(V)$ | Resets `topo` and `vis`, processes all components (`0..n-1`), and reverses `topo`. |

---

## Implementation

```cpp
int n;
vector<vector<int>> adj;
vector<int> topo;
vector<bool> vis;

void dfs(int u) {
    vis[u] = true;

    for (int v : adj[u]) {
        if (!vis[v])
            dfs(v);
    }

    topo.push_back(u);
}

void topoSort() {
    topo.clear();
    vis.assign(n, false);

    for (int i = 0; i < n; i++) {
        if (!vis[i])
            dfs(i);
    }
    reverse(topo.begin(), topo.end());
}
```

---

## Usage

### 1. Initialize 0-Indexed Graph & Run `topoSort()`
Set global `n` and populate the **0-indexed** directed adjacency list `adj` (`0..n-1`), then call `topoSort()`:
```cpp
n = 6;
adj.assign(n, {});

// Directed edges u -> v (u must come before v)
adj[5].push_back(2);
adj[5].push_back(0);
adj[4].push_back(0);
adj[4].push_back(1);
adj[2].push_back(3);
adj[3].push_back(1);

topoSort();
// Example valid order in `topo`: {5, 4, 2, 3, 1, 0}
```

### 2. DAG Dynamic Programming Along `topo`
* **Forward DP (Sources $\to$ Sinks):** Iterate `for (int u : topo)` to propagate values along `u -> v` (e.g., longest path ending at `v`, number of paths from sources).
* **Backward DP (Sinks $\to$ Sources):** Iterate from `i = n - 1` down to `0` (`int u = topo[i]`) to compute values depending on all outgoing neighbors `v` (e.g., longest path starting at `u`).

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste n, adj, topo, vis, dfs, and topoSort here

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
    }

    topoSort();

    for (int i = 0; i < n; i++) {
        cout << topo[i] + 1 << (i + 1 == n ? "\n" : " ");
    }

    return 0;
}
```

---

## Important Notes

1. **Signature Mismatch Fix from Original Snippet:** `dfs` was declared as `void dfs(int u)` (using global `adj`), but called as `dfs(v, adj)` and `dfs(i, adj)`. Fixed the calls to `dfs(v)` and `dfs(i)` so it compiles cleanly.
2. **DAG Requirement (No Cycle Detection):** This implementation assumes the graph is guaranteed to be a **DAG**. If the graph might contain a directed cycle, DFS topological sort will still produce a permutation without warning; either verify afterward that `pos[u] < pos[v]` for all edges `(u, v)`, use a 3-color DFS (`0 = unvisited`, `1 = visiting`, `2 = visited`), or use **Kahn's Algorithm** (`topo.size() < n` indicates a cycle).
3. **Lexicographically Smallest Topological Order:** DFS produces an arbitrary valid topological order. If a problem asks for the **lexicographically smallest** topological ordering, use **Kahn's Algorithm** with a min-heap (`priority_queue<int, vector<int>, greater<int>>`).
4. **1-Indexed Graphs:** If your vertices are numbered `1..n` without subtracting `1`, resize `adj` and `vis` to `n + 1` and loop `for (int i = 1; i <= n; i++)` inside `topoSort()`.