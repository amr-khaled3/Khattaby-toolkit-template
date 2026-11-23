# Multi Ordered Set (`__gnu_pbds` Policy-Based Tree) Usage Guide

A templated C++ wrapper (`multi_ordered_set<T>`) built on GNU PBDS `tree` with `tree_order_statistics_node_update`, pairing each element `x` with a monotonically increasing unique `int id` so duplicate values are preserved while supporting **$O(\log N)$** $k$-th smallest element lookup (`find_by_order`) and rank counting (`order_of_key`).

---

## Complexity Overview

| Operation | Time Complexity | Space Complexity | Description |
| :--- | :--- | :--- | :--- |
| **`insert(x)`** | $O(\log N)$ | $O(1)$ per element | Inserts `x` with unique tag `id++` (allows duplicates). |
| **`erase_one(x)`** | $O(\log N)$ | $O(1)$ | Removes a single occurrence of `x` (no-op if not found). |
| **`erase_all(x)`** | $O(c \log N)$ | $O(1)$ | Removes all $c$ occurrences of `x` from the set. |
| **`order_of_key(x)`** | $O(\log N)$ | $O(1)$ | Returns the number of elements **strictly less** than `x` (`< x`). |
| **`count(x)`** | $O(\log N)$ | $O(1)$ | Returns the exact frequency of `x` in the multiset. |
| **`find_by_order(k)`** | $O(\log N)$ | $O(1)$ | Returns the $k$-th smallest element (**0-indexed**, $0 \le k < \text{size}()$). |
| **`lower_bound(x)`** | $O(\log N)$ | $O(1)$ | Returns the smallest element $\ge x$ (asserts existence). |
| **`upper_bound(x)`** | $O(\log N)$ | $O(1)$ | Returns the smallest element $> x$ (asserts existence). |
| **`size()` / `empty()`** | $O(1)$ | $O(1)$ | Returns total element count (including duplicates) / emptiness check. |
| **`clear()`** | $O(N)$ | $O(1)$ | Empties the tree and resets `id` to `0`. |

---

## Implementation

```cpp
#include <bits/stdc++.h>
#include <ext/pb_ds/assoc_container.hpp>
#include <ext/pb_ds/tree_policy.hpp>

using namespace std;
using namespace __gnu_pbds;

template<typename T>
struct multi_ordered_set {
    tree<pair<T, int>, null_type, less<pair<T, int>>,
         rb_tree_tag, tree_order_statistics_node_update> ost;

    int id = 0;

    void insert(T x) {
        ost.insert({x, id++});
    }

    void erase_one(T x) {
        auto it = ost.lower_bound({x, -1});
        if (it != ost.end() && it->first == x) {
            ost.erase(it);
        }
    }

    void erase_all(T x) {
        while (true) {
            auto it = ost.lower_bound({x, -1});
            if (it != ost.end() && it->first == x) {
                ost.erase(it);
            } else {
                break;
            }
        }
    }

    int order_of_key(T x) const {
        return ost.order_of_key({x, -1});
    }

    int count(T x) const {
        return ost.order_of_key({x, id}) - ost.order_of_key({x, -1});
    }

    T find_by_order(int k) const {
        assert(k >= 0 && k < (int)ost.size());
        return ost.find_by_order(k)->first;
    }

    T lower_bound(T x) const {
        auto it = ost.lower_bound({x, -1});
        assert(it != ost.end());
        return it->first;
    }

    T upper_bound(T x) const {
        auto it = ost.upper_bound({x, id});
        assert(it != ost.end());
        return it->first;
    }

    int size() const {
        return ost.size();
    }

    bool empty() const {
        return ost.empty();
    }

    void clear() {
        ost.clear();
        id = 0;
    }
};
```

---

## Usage

### 1. Initialize & Insert Duplicates
```cpp
multi_ordered_set<long long> ms;
ms.insert(10);
ms.insert(5);
ms.insert(10);
ms.insert(20);
// Sorted state: [5, 10, 10, 20]
```

### 2. Order Statistics & Frequency Queries
```cpp
int lessThan10 = ms.order_of_key(10); // 1 (only {5} is < 10)
int lessOrEq10 = ms.order_of_key(11); // 3 ({5, 10, 10})
int freq10     = ms.count(10);        // 2
long long kth  = ms.find_by_order(2); // 10 (0-indexed: idx 0=5, 1=10, 2=10, 3=20)
```

### 3. Bounds & Single vs. Full Deletion
```cpp
long long lb = ms.lower_bound(10); // 10
long long ub = ms.upper_bound(10); // 20

ms.erase_one(10); // Removes one 10 -> [5, 10, 20]
ms.erase_all(10); // Removes all remaining 10s -> [5, 20]
```

---

## Complete Example (Sliding Window Median)

```cpp
#include <bits/stdc++.h>
#include <ext/pb_ds/assoc_container.hpp>
#include <ext/pb_ds/tree_policy.hpp>

using namespace std;
using namespace __gnu_pbds;

// Paste multi_ordered_set template here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    vector<int> a = {1, 3, -1, -3, 5, 3, 6, 7};
    int k = 3;

    multi_ordered_set<int> window;
    for (int i = 0; i < (int)a.size(); i++) {
        window.insert(a[i]);
        if (i >= k) {
            window.erase_one(a[i - k]);
        }
        if (i >= k - 1) {
            cout << window.find_by_order((k - 1) / 2) << " ";
        }
    }
    cout << "\n"; // Outputs: 1 -1 -1 3 5 6

    return 0;
}
```

---

## Important Notes

1. **Compiler Requirement (GCC Only):** `<ext/pb_ds/...>` is a GNU `libstdc++` extension. It works on Codeforces, CSES, AtCoder, and standard MinGW/GCC setups, but will fail to compile under MSVC or Clang with `libc++`.
2. **Avoid `#define int long long` Conflict:** Never use `#define int long long` above `<ext/pb_ds/assoc_container.hpp>`, as PBDS internal templates explicitly expect 32-bit `int` in several specializations and will fail to compile. Use `using ll = long long;` and `multi_ordered_set<ll>` instead.
3. **Assertions on Out-of-Range Queries:** `find_by_order(k)`, `lower_bound(x)`, and `upper_bound(x)` trigger `assert(...)` (Runtime Error / SIGABRT) if `k` is out of bounds or no element $\ge x$ / $> x$ exists. Always verify `x <= ms.find_by_order(ms.size() - 1)` (for `lower_bound`) or `x < ms.find_by_order(ms.size() - 1)` (for `upper_bound`) when an upper neighbor is not guaranteed.
4. **Why `less_equal` Is Avoided:** Using `{x, id++}` with `less<pair<T, int>>` avoids the notorious PBDS `less_equal` bug where `find` and `erase` break because `less_equal` violates strict weak ordering.