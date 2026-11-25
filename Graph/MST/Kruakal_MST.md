# Kruskal's Minimum Spanning Tree (MST) Usage Guide

A templated C++ implementation of **Kruskal's Algorithm** for finding the Minimum Spanning Tree (or Minimum Spanning Forest if the graph is disconnected) of an undirected weighted graph represented as an adjacency list. Uses a Disjoint Set Union (`DSU`) structure to greedily select minimum-weight edges without forming cycles.

---

## Complexity Overview

| Metric | Complexity | Notes |
| :--- | :--- | :--- |
| **Time Complexity** | $O(E \log E + E \cdot \alpha(V))$ | Dominated by sorting all $E$ edges ($V = n$). |
| **Space Complexity** | $O(V + E)$ | Stores the extracted edge list, DSU arrays, and output MST edges. |

---

## Implementation

```cpp
using ll = long long;

template <typename W>
pair<ll, vector<pair<int, int>>> kruskalMST(int n, const vector<pair<int, W>> adj[]) {
    vector<tuple<W, int, int>> edges;

    for (int u = 1; u <= n; ++u) {
        for (const auto& [v, w] : adj[u]) {
            if (u < v) {
                edges.emplace_back(w, u, v);
            }
        }
    }

    sort(edges.begin(), edges.end());

    DSU dsu(n);
    ll total_weight = 0;
    vector<pair<int, int>> mst_edges;
    mst_edges.reserve(n - 1);

    for (const auto& [w, u, v] : edges) {
        if (dsu.merge(u, v)) {
            total_weight += w;
            mst_edges.emplace_back(u, v);

            if ((int)mst_edges.size() == n - 1) {
                break;
            }
        }
    }

    return {total_weight, mst_edges};
}
```

---

## Usage

### 1. Prerequisites (`DSU` Struct)
Ensure your `DSU` struct is defined above `kruskalMST` and provides:
* A constructor `DSU(int n)` supporting **1-based** nodes $[1, n]$.
* A boolean method `merge(int u, int v)` that returns `true` if `u` and `v` were in separate components and merged, or `false` if they were already connected.

### 2. Build the Undirected Adjacency List
Populate `adj` with `{neighbor, weight}` pairs for both directions (1-based indexing):
```cpp
int n = 4, m = 5;
vector<pair<int, int>> adj[n + 1];

auto addEdge = [&](int u, int v, int w) {
    adj[u].emplace_back(v, w);
    adj[v].emplace_back(u, w);
};

addEdge(1, 2, 10);
addEdge(1, 3, 6);
addEdge(1, 4, 5);
addEdge(2, 4, 15);
addEdge(3, 4, 4);
```

### 3. Compute the MST
Use structured bindings to unpack the minimum total weight and the selected edges:
```cpp
auto [mst_cost, mst_edges] = kruskalMST(n, adj);

// Check if the graph is connected (valid single spanning tree)
if ((int)mst_edges.size() != n - 1) {
    cout << "IMPOSSIBLE\n";
} else {
    cout << mst_cost << "\n";
}
```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

using ll = long long;

struct DSU {
    vector<int> parent, sz;
    DSU(int n) : parent(n + 1), sz(n + 1, 1) {
        iota(parent.begin(), parent.end(), 0);
    }
    int find(int v) {
        return parent[v] == v ? v : parent[v] = find(parent[v]);
    }
    bool merge(int a, int b) {
        a = find(a), b = find(b);
        if (a == b) return false;
        if (sz[a] < sz[b]) swap(a, b);
        parent[b] = a;
        sz[a] += sz[b];
        return true;
    }
};

// Paste kruskalMST template here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n = 4;
    vector<pair<int, ll>> adj[5];

    auto addEdge = [&](int u, int v, ll w) {
        adj[u].emplace_back(v, w);
        adj[v].emplace_back(u, w);
    };

    addEdge(1, 2, 10);
    addEdge(1, 3, 6);
    addEdge(1, 4, 5);
    addEdge(2, 4, 15);
    addEdge(3, 4, 4);

    auto [total_weight, mst_edges] = kruskalMST(n, adj);

    cout << "Total MST Weight: " << total_weight << "\n";
    for (auto [u, v] : mst_edges) {
        cout << u << " - " << v << "\n";
    }

    return 0;
}
```

---

## Important Notes

1. **Deduplication via `u < v`:** Because an undirected graph stores every edge twice (`u -> v` and `v -> u`), `if (u < v)` filters out duplicate reverse edges and self-loops (`u == v`) automatically.
2. **Disconnected Graph Detection:** If the input graph is not connected, `kruskalMST` returns the Minimum Spanning Forest. Always verify `(int)mst_edges.size() == n - 1` (for $n \ge 1$) when the problem requires a single spanning tree covering all vertices.
3. **0-Based Indexing Adaptation:** To use 0-based vertices $[0, n - 1]$, change the outer loop to `for (int u = 0; u < n; ++u)` and ensure `DSU(n)` allocates indices $0 \dots n - 1$.
4. **Maximum Spanning Tree:** To compute a **Maximum Spanning Tree**, replace `sort(edges.begin(), edges.end());` with `sort(edges.rbegin(), edges.rend());` (or negate all edge weights).
5. **Template Weight Flexibility:** `typename W` automatically deduces `int`, `long long`, or `double` from `adj`, while `total_weight` always accumulates into `ll` (`long long`) to avoid integer overflow.