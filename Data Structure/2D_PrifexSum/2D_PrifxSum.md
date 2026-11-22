# 2D Prefix Sum Usage Guide

A C++ utility struct (`Prefix2D`) for computing submatrix rectangle sum queries in **$O(1)$** time over a static 2D grid after an **$O(N \times M)$** preprocessing step. Uses **1-based indexing** for both grid coordinates and queries.

---

## Complexity Overview

| Operation | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`Prefix2D(n, m)`** | $O(N \times M)$ | $O(N \times M)$ | Allocates an $(n + 1) \times (m + 1)$ zero-initialized table. |
| **`build(a)`** | $O(N \times M)$ | $O(1)$ auxiliary | Populates the 2D prefix sum table from a **1-based** matrix `a`. |
| **`query(x1, y1, x2, y2)`** | $O(1)$ | $O(1)$ | Returns the sum of elements inside the inclusive rectangle $[x_1, x_2] \times [y_1, y_2]$. |

---

## Implementation

```cpp
struct Prefix2D {
    int n, m;
    vector<vector<long long>> pref;

    // constructor
    Prefix2D(int n, int m) : n(n), m(m) {
        pref.assign(n + 1, vector<long long>(m + 1, 0));
    }

    // build from given matrix (1-based)
    void build(const vector<vector<int>>& a) {
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                pref[i][j] = a[i][j]
                           + pref[i - 1][j]
                           + pref[i][j - 1]
                           - pref[i - 1][j - 1];
            }
        }
    }

    // query sum of rectangle (x1, y1) -> (x2, y2)
    long long query(int x1, int y1, int x2, int y2) {
        return pref[x2][y2]
             - pref[x1 - 1][y2]
             - pref[x2][y1 - 1]
             + pref[x1 - 1][y1 - 1];
    }
};
```

---

## Usage

### 1. Initialize & Build (1-Based Input Matrix)
Ensure the input matrix `a` is sized `(n + 1) x (m + 1)` and populated starting at row `1`, column `1`:
```cpp
int n = 3, m = 4;
vector<vector<int>> a(n + 1, vector<int>(m + 1, 0));

for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= m; j++) {
        cin >> a[i][j];
    }
}

Prefix2D pref(n, m);
pref.build(a);
```

### 2. Query Submatrix Sum
Pass the top-left corner `(x1, y1)` and bottom-right corner `(x2, y2)` using **1-based inclusive** coordinates ($1 \le x_1 \le x_2 \le n$, $1 \le y_1 \le y_2 \le m$):
```cpp
long long rectSum = pref.query(2, 2, 3, 4);
```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste Prefix2D struct here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n = 3, m = 3;
    vector<vector<int>> grid = {
        {0, 0, 0, 0},
        {0, 1, 2, 3},
        {0, 4, 5, 6},
        {0, 7, 8, 9}
    };

    Prefix2D p2d(n, m);
    p2d.build(grid);

    // Sum of submatrix from (1, 1) to (2, 2): 1 + 2 + 4 + 5 = 12
    cout << p2d.query(1, 1, 2, 2) << "\n";

    // Sum of submatrix from (2, 2) to (3, 3): 5 + 6 + 8 + 9 = 28
    cout << p2d.query(2, 2, 3, 3) << "\n";

    // Sum of entire 3x3 grid: 45
    cout << p2d.query(1, 1, 3, 3) << "\n";

    return 0;
}
```

---

## Important Notes

1. **Strict 1-Based Indexing:** `build(a)` accesses `a[i][j]` for $1 \le i \le n$ and $1 \le j \le m$. Passing a 0-based `n x m` vector will cause an **out-of-bounds runtime error**. If your input matrix is 0-based, change `a[i][j]` inside `build` to `a[i - 1][j - 1]`.
2. **Coordinate Order:** Always ensure $x_1 \le x_2$ and $y_1 \le y_2$ when calling `query(x1, y1, x2, y2)`. If `x1 > x2` or `y1 > y2`, return `0` before computing the formula.
3. **Large Cell Values:** `pref` already uses `long long` to prevent overflow during summation, while `build` accepts `vector<vector<int>>`. If individual grid cells can exceed $2 \times 10^9$, change the parameter type of `build` to `const vector<vector<long long>>& a`.
4. **Static Grid Only:** Like 1D prefix sums, `Prefix2D` does not support dynamic cell updates after `build()`. For dynamic 2D point updates and rectangle queries, use a **2D Fenwick Tree (BIT)**.