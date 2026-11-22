# Coordinate Compression Usage Guide

A C++ utility struct that maps large, sparse, or negative values (up to `long long`) into compact **0-based** contiguous indices $[0, K - 1]$, where $K$ is the number of unique elements. Essential for using Segment Trees, Fenwick Trees (BIT), or prefix sums over large value ranges (e.g., $[-10^{18}, 10^{18}]$).

---

## Complexity Overview

| Operation | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`CoordinateCompression(v)`** | $O(N \log N)$ | $O(N)$ | Direct constructor that adds all elements of `v` and builds immediately. |
| **`add(x)`** | $O(1)$ amortized | $O(1)$ | Appends a single value before sorting. |
| **`add(v)`** | $O(\Vert{}v\Vert{})$ | $O(\Vert{}v\Vert{})$ | Appends all elements of a vector. |
| **`build()`** | $O(N \log N)$ | $O(1)$ auxiliary | Sorts and removes duplicates ($N$ = total added items). |
| **`get(x)`** | $O(\log K)$ | $O(1)$ | Binary searches for the 0-based compressed index ($K$ = unique items). |
| **`get(v)`** | $O(\Vert{}v\Vert{} \log K)$ | $O(\Vert{}v\Vert{})$ | Compresses an entire vector into a new `vector<int>`. |
| **`rev(idx)`** | $O(1)$ | $O(1)$ | Retrieves the original `long long` value from its compressed index. |
| **`size()`** | $O(1)$ | $O(1)$ | Returns $K$, the number of unique compressed values. |

---

## Implementation

```cpp
struct CoordinateCompression {
    vector<long long> vals;

    CoordinateCompression() = default;

    template<typename T>
    CoordinateCompression(const vector<T>& v) {
        add(v);
        build();
    }

    template<typename T>
    void add(const vector<T>& v) {
        for (auto x : v)
            vals.push_back(x);
    }

    void build() {
        sort(vals.begin(), vals.end());
        vals.erase(unique(vals.begin(), vals.end()), vals.end());
    }

    int get(long long x) const {
        return lower_bound(vals.begin(), vals.end(), x) - vals.begin();
    }

    template<typename T>
    vector<int> get(const vector<T>& v) const {
        vector<int> res(v.size());
        for (int i = 0; i < (int)v.size(); i++) {
            res[i] = get(v[i]);
        }
        return res;
    }

    long long rev(int idx) const {
        return vals[idx];
    }

    int size() const {
        return vals.size();
    }
};
```

---

## Usage

### 1. Initialization & Build

#### Option A: Direct Initialization (Single Vector)
Pass a vector directly to the constructor to add and build in one step:
```cpp
vector<long long> a = {1000000000000LL, -50, 42, -50};
CoordinateCompression cc(a);
```

#### Option B: Incremental Initialization (Multiple Sources / Queries)
Use the default constructor when collecting values from multiple arrays or offline queries, then call `build()` once:
```cpp
CoordinateCompression cc;
cc.add(a);
cc.add(100);
cc.build();
```

### 2. Compress Values (`get`)
Convert single values or entire vectors into **0-based** indices:
```cpp
int idx = cc.get(42);              // Returns 1 (sorted unique vals: {-50, 42, 100, 10^12})
vector<int> comp = cc.get(a);      // Returns {3, 0, 1, 0}
```

### 3. Decompress / Reverse Lookup (`rev`)
Map a compressed index back to its original value:
```cpp
long long original = cc.rev(3);    // Returns 1000000000000LL
int uniqueCount = cc.size();       // Returns 4 (use this as the size for SegTree / Fenwick)
```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste CoordinateCompression struct here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    vector<long long> arr = {5000000000LL, 10, -200, 10, 999};

    CoordinateCompression cc(arr);

    vector<int> compressed = cc.get(arr);
    vector<int> freq(cc.size(), 0);

    for (int id : compressed) {
        freq[id]++;
    }

    for (int id = 0; id < cc.size(); id++) {
        cout << cc.rev(id) << " -> index " << id << " (count: " << freq[id] << ")\n";
    }

    return 0;
}
```

---

## Important Notes

1. **Offline Requirement:** All values that will ever be queried with `get(x)` must be added before calling `build()`. If `x` was not added, `get(x)` returns the index of the first element $\ge x$ (standard `lower_bound` behavior).
2. **0-Based Indexing:** `get()` returns indices in $[0, \text{size}() - 1]$, which aligns directly with 0-based structures like `SegTree(cc.size())`. For a 1-based Fenwick Tree, use `cc.get(x) + 1`.
3. **Include Query Coordinates:** When answering range queries $[L, R]$ over large coordinates, use the default constructor `CoordinateCompression cc;`, add array elements alongside $L$ and $R$ (and $R + 1$ if using difference arrays or half-open intervals), and then call `cc.build()`.