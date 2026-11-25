# Two-Edge-Connected Components (TECC) & Bridge Tree Usage Guide

A C++ struct (`TECC`) using Tarjan's low-link algorithm to find all **Bridges (cut-edges)**, decompose a **1-indexed** undirected graph (including disconnected graphs and multigraphs with parallel edges) into **Two-Edge-Connected Components**, and optionally build the condensed **Bridge Tree** in linear **$O(V + E)$** time.

---

## Complexity Overview

| Member / Operation | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`TECC(graph)`** | $O(V + E)$ | $O(V + E)$ | Runs Tarjan's DFS (`dfs`) and component assignment (`dfs2`) automatically on construction. |
| **`build_tree()`** | $O(V + E)$ | $O(k)$ | Builds the **0-indexed** condensed Bridge Tree `t` of size `k` (one node per TECC, edges = bridges). |
| **`br`** | — | $O(V)$ | `vector<pair<int, int>>` storing all bridge edges `{u, v}` in the original graph. |
| **`comp[u]`** | $O(1)$ | $O(V)$ | **0-indexed** TECC ID in $[0, k - 1]$ that original vertex `u` ($1 \le u \le n$) belongs to. |
| **`k`** | $O(1)$ | $O(1)$ | Total number of Two-Edge-Connected Components found. |

---

## Implementation

```cpp
struct TECC { // 1-indexed input graph, 0-indexed component IDs [0..k-1]
    int n, k;
    vector<vector<int>> g, t;
    vector<bool> used;
    vector<int> comp, ord, low;

    using edge = pair<int, int>;
    vector<edge> br;

    void dfs(int x, int prv, int &c) {
        used[x] = true;
        ord[x] = c++;
        low[x] = n + 1;

        bool mul = false;

        for (auto y : g[x]) {
            if (used[y]) {
                if (y != prv || mul)
                    low[x] = min(low[x], ord[y]);
                else
                    mul = true;
                continue;
            }

            dfs(y, x, c);
            low[x] = min(low[x], low[y]);
        }
    }

    void dfs2(int x, int num) {
        comp[x] = num;

        for (auto y : g[x]) {
            if (comp[y] != -1)
                continue;

            if (ord[x] < low[y]) {
                br.push_back({x, y});
                ++k;
                dfs2(y, k);
            } else {
                dfs2(y, num);
            }
        }
    }

    TECC(const vector<vector<int>> &graph)
        : g(graph),
          n((int)graph.size() - 1),
          used(n + 1, false),
          comp(n + 1, -1),
          ord(n + 1),
          low(n + 1),
          k(0) {

        int c = 0;

        for (int i = 1; i <= n; i++) {
            if (used[i])
                continue;

            dfs(i, -1, c);
            dfs2(i, k);
            ++k;
        }
    }

    void build_tree() {
        t.assign(k, {});

        for (auto [u, v] : br) {
            int x = comp[u];
            int y = comp[v];
            t[x].push_back(y);
            t[y].push_back(x);
        }
    }
};
```

---

## Usage

### 1. Build Graph (1-Indexed) & Construct `TECC`
Pass a **1-indexed** adjacency list `vector<vector<int>> g(n + 1)` directly to `TECC`:
```cpp
int n = 6;
vector<vector<int>> g(n + 1);

auto addEdge = [&](int u, int v) {
    g[u].push_back(v);
    g[v].push_back(u);
};

// Cycle 1-2-3, bridge 3-4, cycle 4-5-6
addEdge(1, 2); addEdge(2, 3); addEdge(3, 1);
addEdge(3, 4);
addEdge(4, 5); addEdge(5, 6); addEdge(6, 4);

TECC tecc(g);
```

### 2. Inspect Bridges & Component IDs
```cpp
int numComps = tecc.k;          // 2 components: {1, 2, 3} and {4, 5, 6}
int compOf1  = tecc.comp[1];    // 0 (0-indexed)
int compOf5  = tecc.comp[5];    // 1 (0-indexed)

for (auto [u, v] : tecc.br) {
    cout << "Bridge: " << u << " - " << v << "\n"; // 3 - 4
}
```

### 3. Build & Traverse the Bridge Tree (`build_tree`)
Call `tecc.build_tree()` to populate `tecc.t`, which is a **0-indexed** tree (or forest if `g` was disconnected) with `tecc.k` nodes:
```cpp
tecc.build_tree();

// Example: minimum edges needed to make a connected graph 2-edge-connected
// equals ceil(leaves / 2) in the Bridge Tree (when k > 1)
int leaves = 0;
for (int i = 0; i < tecc.k; i++) {
    if ((int)tecc.t[i].size() == 1) leaves++;
}
int minEdgesToAdd = (tecc.k == 1) ? 0 : (leaves + 1) / 2;
```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste TECC struct here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n = 7;
    vector<vector<int>> g(n + 1);

    auto addEdge = [&](int u, int v) {
        g[u].push_back(v);
        g[v].push_back(u);
    };

    // Component 0: {1, 2, 3}
    addEdge(1, 2);
    addEdge(2, 3);
    addEdge(3, 1);

    // Bridges: (3, 4) and (4, 5)
    addEdge(3, 4);
    addEdge(4, 5);

    // Component with parallel edges: {5, 6} (NOT a bridge because of duplicate edge)
    addEdge(5, 6);
    addEdge(5, 6);

    // Bridge: (6, 7)
    addEdge(6, 7);

    TECC tecc(g);
    tecc.build_tree();

    cout << "Total TECCs: " << tecc.k << "\n"; // 4
    cout << "Bridges (" << tecc.br.size() << "):\n";
    for (auto [u, v] : tecc.br) {
        cout << u << " - " << v << "\n";
    }

    for (int u = 1; u <= n; u++) {
        cout << "Node " << u << " -> TECC " << tecc.comp[u] << "\n";
    }

    return 0;
}
```

---

## Important Notes

1. **Indexing Convention (1-Based Input, 0-Based Output):**
   * The input `graph` **must** have size `n + 1` with vertices numbered `1..n` (`n = graph.size() - 1`).
   * The output component IDs `comp[u]` and the condensed Bridge Tree `t` are **0-indexed** in $[0, k - 1]$.
2. **Parallel Edges Handled Automatically:** The `bool mul` flag skips only the **first** edge back to the DFS parent `prv`. Any second parallel edge between `x` and `prv` updates `low[x] = min(low[x], ord[prv])`, so parallel edges are never falsely flagged as bridges.
3. **Self-Loops Safe:** If a self-loop `(x, x)` exists in `g[x]`, `used[x]` is already `true` and `x != prv`, so `low[x] = min(low[x], ord[x])` leaves `low[x]` unchanged without affecting bridges.
4. **Disconnected Graphs:** The outer loop `for (int i = 1; i <= n; i++)` processes every connected component, producing a **Bridge Forest** in `t` when `g` is disconnected.