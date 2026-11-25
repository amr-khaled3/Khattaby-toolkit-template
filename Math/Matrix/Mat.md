# Modular Matrix & Matrix Exponentiation Usage Guide

A C++ matrix struct (`Mat`) supporting modular addition, subtraction, multiplication, and binary matrix exponentiation (`pow(k)`) under a global modulus `MOD`—essential for solving linear recurrences (such as $N$-th Fibonacci for $N \le 10^{18}$) and counting fixed-length walks in graphs in **$O(D^3 \log K)$** time.

---

## Complexity Overview

| Operation | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`Mat(n, m)` / `Mat(v)`** | $O(N \times M)$ | $O(N \times M)$ | Initializes a 0-based $n \times m$ matrix (zero-filled or from a 2D vector). |
| **`make_unit()`** | $O(N^2)$ | $O(1)$ | Converts an $N \times N$ square matrix into the identity matrix $I_N$. |
| **`A + B` / `A - B`** | $O(N \times M)$ | $O(N \times M)$ | Element-wise modular addition / subtraction (requires matching dimensions). |
| **`A * B`** | $O(N \times M \times P)$ | $O(N \times P)$ | Standard matrix multiplication of $(N \times M)$ by $(M \times P)$ modulo `MOD`. |
| **`A.pow(k)`** | $O(N^3 \log k)$ | $O(N^2)$ | Binary exponentiation computing $A^k \bmod \text{MOD}$ for $k \ge 0$ (requires $N = M$). |
| **`==` / `!=`** | $O(N \times M)$ | $O(1)$ | Compares two matrices element-by-element. |

---

## Implementation

```cpp
const int MOD = 1e9 + 7;

struct Mat
{
    int n, m;
    vector<vector<int>> a;
    Mat() {}
    Mat(int _n, int _m)
    {
        n = _n;
        m = _m;
        a.assign(n, vector<int>(m, 0));
    }
    Mat(vector<vector<int>> v)
    {
        n = v.size();
        m = n ? v[0].size() : 0;
        a = v;
    }
    inline void make_unit()
    {
        assert(n == m);
        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < n; j++)
                a[i][j] = i == j;
        }
    }
    inline Mat operator+(const Mat &b)
    {
        assert(n == b.n && m == b.m);
        Mat ans = Mat(n, m);
        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < m; j++)
            {
                ans.a[i][j] = (a[i][j] + b.a[i][j]) % MOD;
            }
        }
        return ans;
    }
    inline Mat operator-(const Mat &b)
    {
        assert(n == b.n && m == b.m);
        Mat ans = Mat(n, m);
        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < m; j++)
            {
                ans.a[i][j] = (a[i][j] - b.a[i][j] + MOD) % MOD;
            }
        }
        return ans;
    }
    inline Mat operator*(const Mat &b)
    {
        assert(m == b.n);
        Mat ans = Mat(n, b.m);
        for (int i = 0; i < n; i++)
        {
            for (int j = 0; j < b.m; j++)
            {
                for (int k = 0; k < m; k++)
                {
                    ans.a[i][j] = (ans.a[i][j] + 1LL * a[i][k] * b.a[k][j] % MOD) % MOD;
                }
            }
        }
        return ans;
    }
    inline Mat pow(long long k)
    {
        assert(n == m);
        Mat ans(n, n), t = a;
        ans.make_unit();
        while (k)
        {
            if (k & 1)
                ans = ans * t;
            t = t * t;
            k >>= 1;
        }
        return ans;
    }
    inline Mat &operator+=(const Mat &b) { return *this = (*this) + b; }
    inline Mat &operator-=(const Mat &b) { return *this = (*this) - b; }
    inline Mat &operator*=(const Mat &b) { return *this = (*this) * b; }
    inline bool operator==(const Mat &b) { return a == b.a; }
    inline bool operator!=(const Mat &b) { return a != b.a; }
};
```

---

## Usage

### 1. Initialize & Access Elements (0-Based Indexing)
Create an empty $n \times m$ matrix or construct directly from a 2D initializer list:
```cpp
Mat T({
    {1, 1},
    {1, 0}
});

Mat base(2, 1);
base.a[0][0] = 1; // F(1) = 1
base.a[1][0] = 0; // F(0) = 0
```

### 2. Linear Recurrence via Matrix Exponentiation ($N$-th Fibonacci)
Since $\begin{pmatrix} 1 & 1 \\ 1 & 0 \end{pmatrix}^{n - 1} \begin{pmatrix} F_1 \\ F_0 \end{pmatrix} = \begin{pmatrix} F_n \\ F_{n - 1} \end{pmatrix}$:
```cpp
long long n = 1000000000000000000LL; // 10^18
Mat res = T.pow(n - 1) * base;
int fibN = res.a[0][0];
```

### 3. Number of Walks of Exact Length $K$ in a Graph
If `adj` is an $N \times N$ 0-based adjacency matrix (`adj.a[u][v] = 1` if there is a directed edge $u \to v$), then `walks = adj.pow(K)` gives the number of walks of length $K$ from $u$ to $v$ in `walks.a[u][v]`.

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste MOD and Mat struct here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    // Compute 10th Fibonacci number (F_0 = 0, F_1 = 1, ..., F_10 = 55)
    long long n = 10;

    Mat T({
        {1, 1},
        {1, 0}
    });

    Mat base({
        {1},
        {0}
    });

    Mat ans = T.pow(n - 1) * base;
    cout << "F(" << n << ") = " << ans.a[0][0] << "\n"; // 55

    return 0;
}
```

---

## Important Notes

1. **Required Constant `MOD`:** Ensure `MOD` is defined globally above `struct Mat` (e.g., `const int MOD = 1e9 + 7;` or `998244353`).
2. **Inputs Must Be Normalized in $[0, \text{MOD} - 1]$:** `operator+` and `operator-` assume entries in `a` and `b` are already in $[0, \text{MOD} - 1]$ (`a[i][j] - b.a[i][j] + MOD` would stay negative if `b.a[i][j] >= 2 * MOD`). Normalize initial matrix entries with `(x % MOD + MOD) % MOD` before performing operations.
3. **Zero Exponent (`A.pow(0)`):** `pow(0)` safely returns the $N \times N$ identity matrix $I_N$ via `ans.make_unit()`.
4. **Loop Order Optimization for Large Matrices ($N \ge 100$):** In `operator*`, swapping the inner two loops to `i -> k -> j` (with `if (a[i][k] == 0) continue;`) improves CPU cache locality and skips zero entries, making multiplication roughly 2–3× faster on sparse transition matrices.