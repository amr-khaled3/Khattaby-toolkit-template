# Disjoint Set Union (DSU) Usage Guide

A C++ implementation of **Disjoint Set Union (Union-Find)** with **path halving** and **union by size**, supporting dynamic connectivity queries, component merging, and component size lookups in near-constant **$O(\alpha(N))$** amortized time. Pre-allocated for **1-based indexing** $[1, n]$ (while safely supporting 0-based $[0, n]$ as well).

---

## Complexity Overview

| Operation | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`DSU(n)` / `init(n + 1)`** | $O(N)$ | $O(N)$ | Initializes $n + 1$ singleton sets ($0 \dots n$). |
| **`leader(x)`** | $O(\alpha(N))$ amortized | $O(1)$ | Iterative path halving to find the representative root of `x`. |
| **`same(x, y)`** | $O(\alpha(N))$ amortized | $O(1)$ | Checks whether `x` and `y` belong to the same connected component. |
| **`merge(x, y)`** | $O(\alpha(N))$ amortized | $O(1)$ | Merges the sets containing `x` and `y`; returns `false` if already connected. |
| **`size(x)`** | $O(\alpha(N))$ amortized | $O(1)$ | Returns the number of elements in the component containing `x`. |

---

## Implementation

```cpp
struct DSU {
    vector<int> f, siz;
    
    DSU() {}
    DSU(int n) {
        init(n + 1);
    }
    
    void init(int n) {
        f.resize(n);
        iota(f.begin(), f.end(), 0);
        siz.assign(n, 1);
    }
    
    int leader(int x) {
        while (x != f[x]) {
            x = f[x] = f[f[x]];
        }
        return x;
    }
    
    bool same(int x, int y) {
        return leader(x) == leader(y);
    }
    
    bool merge(int x, int y) {
        x = leader(x);
        y = leader(y);
        if (x == y) {
            return false;
        }
 
	    if (siz[x] > siz[y])
        {
            swap(x , y);
        }
 
        siz[x] += siz[y];
        f[y] = x;
        return true;
    }
    
    int size(int x) {
        return siz[leader(x)];
    }
};
```

---

## Usage

### 1. Initialization
Pass `n` to the constructor to support nodes $1 \dots n$ (or $0 \dots n$):
```cpp
int n = 5;
DSU dsu(n);
```

### 2. Merging Components (`merge`)
Unite two elements and check whether a new connection was actually formed:
```cpp
bool merged = dsu.merge(1, 2); // true (1 and 2 were separate)
bool again  = dsu.merge(1, 2); // false (already in the same component)
```

### 3. Connectivity & Component Queries (`same`, `leader`, `size`)
```cpp
dsu.merge(2, 3);

bool connected = dsu.same(1, 3); // true
int root       = dsu.leader(3);  // representative root of {1, 2, 3}
int compSize   = dsu.size(1);    // 3
```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste DSU struct here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n = 5, m = 3;
    DSU dsu(n);

    vector<pair<int, int>> edges = {{1, 2}, {2, 3}, {4, 5}};
    int components = n;

    for (auto [u, v] : edges) {
        if (dsu.merge(u, v)) {
            components--;
        }
    }

    cout << "Connected components: " << components << "\n"; // 2
    cout << "Size of node 1's component: " << dsu.size(1) << "\n"; // 3
    cout << "Are 1 and 4 connected? " << (dsu.same(1, 4) ? "YES" : "NO") << "\n"; // NO

    return 0;
}
```

---

## Important Notes

1. **Both 0-Based and 1-Based Safe:** Because `DSU(n)` calls `init(n + 1)`, valid node indices span $[0, n]$, making it compatible out-of-the-box with both 1-based problems ($1 \dots n$) and 0-based problems ($0 \dots n - 1$).
2. **Note on Union Direction in `merge`:** In the snippet above, `if (siz[x] > siz[y]) swap(x, y);` followed by `f[y] = x` attaches the **larger** tree (`y`) under the **smaller** tree (`x`), though `siz[x]` still correctly tracks the combined size. For optimal worst-case tree height (attaching smaller under larger), change `>` to `<` (`if (siz[x] < siz[y]) swap(x, y);`).
3. **Iterative Path Halving:** `leader(x)` uses a `while` loop (`x = f[x] = f[f[x]]`) instead of recursion, completely eliminating any risk of stack overflow on deep trees when $N \ge 10^5$.
4. **Seamless `kruskalMST` Integration:** This struct matches the exact interface (`DSU(n)` and `bool merge(u, v)`) expected by the `kruskalMST` template.