# Strongly Connected Components (Tarjan's SCC & Condensation DAG) Usage Guide

A C++ struct (`SCC`) implementing **Tarjan's Strongly Connected Components** algorithm for **1-indexed** directed graphs in linear **$O(V + E)$** time, automatically grouping vertices into SCCs (`comp`), assigning component IDs (`id`), and building the condensed **Directed Acyclic Graph (DAG)** (`adjCC`) in reverse topological order.

---

## Complexity Overview

| Member / Operation | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`SCC(n)`** | $O(1)$ | $O(1)$ | Initializes an `SCC` solver for `n` vertices (`1..n`). |
| **`build(_adj)`** | $O(V + E)$ | $O(V + E)$ | Runs Tarjan's DFS across all unvisited vertices (`1..n`) and builds the condensation DAG `adjCC`. |
| **`condens()`** | $O(V + E)$ | $O(V + E)$ | Populates the **0-indexed** condensation DAG `adjCC` of size `sz` (may include parallel edges). |
| **`sz`** | $O(1)$ | $O(1)$ | Total number of Strongly Connected Components found. |
| **`id[u]`** | $O(1)$ | $O(V)$ | **0-indexed** SCC ID in $[0, \text{sz} - 1]$ that original vertex `u` ($1 \le u \le n$) belongs to. |
| **`comp[c]`** | $O(1)$ | $O(V)$ | `vector<int>` containing all original vertices belonging to **0-indexed** SCC `c`. |

---

## Implementation

```cpp
struct SCC { // 1-indexed input graph, 0-indexed component IDs [0..sz-1]
    int n, timer = 1, sz;
    vector<vector<int>> adj, comp, adjCC;
    vector<int> in, low, id, st;
    vector<bool> stacked;

    SCC(int n) : n(n), sz(0) {}

    void build(const vector<vector<int>> &_adj) {
        adj = _adj;
        timer = 1;
        sz = 0;
        comp.clear();
        st.clear();
        in.assign(n + 1, 0);
        low.assign(n + 1, 0);
        id.assign(n + 1, 0);
        stacked.assign(n + 1, 0);

        for (int i = 1; i <= n; i++) {
            if (!in[i]) dfs(i);
        }
        condens();
    }

    void dfs(int u) {
        in[u] = low[u] = timer++;
        stacked[u] = 1;
        st.push_back(u);
        for (int v : adj[u]) {
            if (!in[v]) {
                dfs(v);
                low[u] = min(low[u], low[v]);
            } else if (stacked[v]) {
                low[u] = min(low[u], in[v]);
            }
        }
        if (low[u] == in[u]) {
            comp.push_back({});
            int v = -1;
            while (v != u) {
                v = st.back();
                st.pop_back();
                stacked[v] = 0;
                id[v] = sz;
                comp.back().push_back(v);
            }
            ++sz;
        }
    }

    void condens() {
        adjCC.assign(sz, {});
        for (int u = 1; u <= n; u++) {
            for (int v : adj[u]) {
                if (id[u] != id[v])
                    adjCC[id[u]].emplace_back(id[v]);
            }
        }
    }
};
```

---

## Usage

### 1. Build Directed Graph (1-Indexed) & Run `SCC`
Pass a **1-indexed** directed adjacency list `vector<vector<int>> g(n + 1)` to `scc.build(g)`:
```cpp
int n = 5;
vector<vector<int>> g(n + 1);

// Cycle 1 -> 2 -> 3 -> 1, edge 3 -> 4, cycle 4 <-> 5
g[1].push_back(2);
g[2].push_back(3);
g[3].push_back(1);
g[3].push_back(4);
g[4].push_back(5);
g[5].push_back(4);

SCC scc(n);
scc.build(g);
```

### 2. Inspect Components & Vertex Mappings
```cpp
int totalSCCs = scc.sz;       // 2
int compOf1   = scc.id[1];    // 0-indexed component ID
auto& members = scc.comp[compOf1]; // {3, 2, 1}
```

### 3. DP on the Condensation DAG (`adjCC`)
Because Tarjan's algorithm pops sink components first, every directed edge $u \to v$ in `adjCC` satisfies **$u > v$** (`0, 1, ..., sz - 1` is already a **reverse topological order**!):
```cpp
// Example: compute max vertices reachable from each SCC in O(V + E) without extra topo-sort
vector<int> dp(scc.sz, 0);
for (int c = 0; c < scc.sz; c++) {
    dp[c] = (int)scc.comp[c].size();
    for (int nxt : scc.adjCC[c]) {
        dp[c] = max(dp[c], (int)scc.comp[c].size() + dp[nxt]);
    }
}
```

### 4. Deduplicating Parallel Edges in `adjCC` (If Needed)
If multiple edges exist between two SCCs and your problem requires a simple DAG (e.g., counting edges or paths without multiplicity):
```cpp
for (int c = 0; c < scc.sz; c++) {
    sort(scc.adjCC[c].begin(), scc.adjCC[c].end());
    scc.adjCC[c].erase(unique(scc.adjCC[c].begin(), scc.adjCC[c].end()), scc.adjCC[c].end());
}
```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste SCC struct here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n = 6;
    vector<vector<int>> g(n + 1);

    // SCC A: {1, 2, 3}
    g[1].push_back(2);
    g[2].push_back(3);
    g[3].push_back(1);

    // Edges from SCC A -> SCC B: {4, 5}
    g[2].push_back(4);
    g[4].push_back(5);
    g[5].push_back(4);

    // Edge from SCC B -> SCC C: {6}
    g[5].push_back(6);

    SCC scc(n);
    scc.build(g);

    cout << "Number of SCCs: " << scc.sz << "\n"; // 3
    for (int c = 0; c < scc.sz; c++) {
        cout << "SCC " << c << ": ";
        for (int u : scc.comp[c]) cout << u << " ";
        cout << "\n";
    }

    // Minimum edges to make the entire graph strongly connected (when sz > 1):
    // max(number of sources with in-degree 0, number of sinks with out-degree 0)
    vector<int> indeg(scc.sz, 0), outdeg(scc.sz, 0);
    for (int u = 0; u < scc.sz; u++) {
        for (int v : scc.adjCC[u]) {
            outdeg[u]++;
            indeg[v]++;
        }
    }
    int sources = 0, sinks = 0;
    for (int c = 0; c < scc.sz; c++) {
        if (indeg[c] == 0) sources++;
        if (outdeg[c] == 0) sinks++;
    }
    cout << "Min edges to strongly connect: " << (scc.sz == 1 ? 0 : max(sources, sinks)) << "\n";

    return 0;
}
```

---

## Important Notes

1. **Indexing Convention (1-Based Input, 0-Based Output):**
   * The input graph `_adj` **must** be **1-indexed** with vertices `1..n` (`_adj.size() >= n + 1`).
   * The output component IDs `id[u]`, `comp`, and the condensed graph `adjCC` are **0-indexed** in $[0, \text{sz} - 1]$.
2. **Reverse Topological Order Guarantee:** In `adjCC`, every edge $u \to v$ satisfies $u > v$. Iterating `for (int c = 0; c < scc.sz; c++)` visits sinks first (bottom-up DP), while `for (int c = scc.sz - 1; c >= 0; c--)` visits sources first (top-down topological order).
3. **Parallel Edges in `adjCC`:** `condens()` adds an edge `id[u] -> id[v]` for every original edge `u -> v` crossing between distinct SCCs, so `adjCC` can contain duplicate directed edges. Sort and `unique` each adjacency list (see Usage #4) whenever simple-graph degrees or path counts are needed.
4. **Safe Reuse Across Calls:** Added `timer = 1; sz = 0; comp.clear(); st.clear();` and `const` reference in `build(const vector<vector<int>> &_adj)` so calling `build` multiple times on the same `SCC` instance resets all state cleanly.