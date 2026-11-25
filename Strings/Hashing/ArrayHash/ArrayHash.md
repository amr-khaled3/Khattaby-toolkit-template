# Subarray Polynomial Rolling Hash (`ArrayHash`) Usage Guide

A C++ polynomial rolling hash struct (`ArrayHash`) for integer arrays (`vector<int>`), using a large prime base (`BASE = 911382323`, `MOD = 1e9 + 7`) and a $+10^6$ value offset to safely support negative and zero elements while answering **0-based inclusive** subarray hash queries $a[l \dots r]$ in **$O(1)$** time after **$O(N)$** preprocessing.

---

## Complexity Overview

| Operation | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`ArrayHash(a)`** | $O(N)$ | $O(N)$ | Precomputes base powers `pw[0..n]` and 1-based prefix hashes `pref[0..n]` in linear time. |
| **`getHash(l, r)`** | $O(1)$ | $O(1)$ | Returns the normalized hash in $[0, \text{MOD} - 1]$ for the **0-based inclusive** subarray $a[l \dots r]$. |

---

## Implementation

```cpp
using LL = long long;

struct ArrayHash {
    static const LL MOD  = 1e9 + 7;
    static const LL BASE = 911382323;

    vector<LL> pref, pw;

    ArrayHash(const vector<int>& a) {
        int n = (int)a.size();

        pref.resize(n + 1, 0);
        pw.resize(n + 1, 1);

        for (int i = 1; i <= n; i++) {
            pw[i] = (pw[i - 1] * BASE) % MOD;
        }

        for (int i = 0; i < n; i++) {
            pref[i + 1] =
                ((pref[i] * BASE + a[i] + 1000000) % MOD + MOD) % MOD;
        }
    }

    LL getHash(int l, int r) const {
        LL h =
            (pref[r + 1] - (pref[l] * pw[r - l + 1]) % MOD) % MOD;

        if (h < 0) h += MOD;
        return h;
    }
};
```

---

## Usage

### 1. Initialize & Build
Pass your `vector<int>` directly to the `ArrayHash` constructor:
```cpp
vector<int> arr = {5, -3, 10, 5, -3, 10};
ArrayHash ah(arr);
```

### 2. Subarray Equality Queries (`getHash(l, r)`)
Uses **0-based inclusive intervals $[l, r]$** (both `l` and `r` are included):
```cpp
// Compare arr[0..2] ({5, -3, 10}) and arr[3..5] ({5, -3, 10})
bool equal = (ah.getHash(0, 2) == ah.getHash(3, 5)); // true
```

### 3. Comparing Subarrays Across Two Different Arrays
Because `BASE` and `MOD` are static constants, hashes from two separate `ArrayHash` instances are directly comparable:
```cpp
ArrayHash ahA(arrA), ahB(arrB);
bool match = (ahA.getHash(l1, r1) == ahB.getHash(l2, r2));
```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste LL and ArrayHash struct here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    vector<int> a = {10, -5, 0, 10, -5, 0, 7};
    int n = (int)a.size();

    ArrayHash ah(a);

    // Check if a[0..2] == a[3..5]
    cout << (ah.getHash(0, 2) == ah.getHash(3, 5) ? "Equal\n" : "Different\n"); // Equal

    // Count distinct subarrays of length 3
    set<LL> distinct;
    for (int i = 0; i + 3 <= n; i++) {
        distinct.insert(ah.getHash(i, i + 2));
    }
    cout << "Distinct subarrays of length 3: " << distinct.size() << "\n"; // 4

    return 0;
}
```

---

## Important Notes

1. **Element Value Range & Offset (`+ 1000000`):** Adding `1000000` keeps mapped values strictly positive for all elements $\ge -999,999$. If your array contains elements $< -10^6$ (e.g., $-10^9$), normalize negative values safely (as done in the `((... % MOD) + MOD) % MOD` fix above) or apply **Coordinate Compression** first.
2. **64-Bit Input Arrays (`vector<long long>`):** If elements in `a` can exceed $2 \times 10^9$, change the constructor parameter to `const vector<long long>& a` and reduce `a[i] % MOD` before adding `1000000` so `pref[i] * BASE + a[i]` never overflows signed 64-bit `LL`.
3. **Single-Hash Collision Risk on Codeforces:** A single 32-bit prime modulus (`1e9 + 7`) has an $O(K^2 / 10^9)$ birthday-paradox collision probability when comparing $K \gtrsim 3 \times 10^4$ distinct subarrays (or against anti-hash tests). For high-collision-count problems, pair it with a second modulus (`MOD2 = 1e9 + 9`, `BASE2 = 972663749`) or randomize `BASE` at runtime via `mt19937_64`.
4. **0-Based Inclusive Range $[l, r]$:** Always ensure $0 \le l \le r < \|a\|$ when calling `getHash(l, r)`.