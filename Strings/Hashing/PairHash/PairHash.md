# Custom `PairHash` (Anti-Hash `splitmix64` & Fast XOR) Usage Guide

C++ custom hash functors (`PairHash`) for using `std::pair<long long, long long>` (and `std::pair<int, int>`) as keys in `std::unordered_map` and `std::unordered_set`—which lack a built-in `std::hash` specialization in C++—comparing a lightweight bit-shift hash against a **collision-resistant `splitmix64` hash** safe from Codeforces anti-hash test cases.

---

## Complexity Overview

| Variant | Time Complexity (Average) | Time Complexity (Adversarial) | Space Complexity | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **`FastPairHash` (`p.first ^ (p.second << 32)`)** | $O(1)$ | $O(N)$ worst-case | $O(1)$ | Fast, zero-overhead bit mix; vulnerable to crafted bucket collisions when `p.second` exceeds 32 bits or on adversarial tests. |
| **`PairHash` (`splitmix64` + random salt)** | $O(1)$ | $O(1)$ expected | $O(1)$ | Uses `splitmix64` avalanche mixing (plus optional runtime `FIXED_RANDOM` seed) to defeat Codeforces hash-killer tests. |

---

## Implementation

```cpp
// 1. Recommended: Anti-Hash Safe PairHash using splitmix64 + runtime random seed
struct PairHash {
    static uint64_t splitmix64(uint64_t x) {
        x += 0x9e3779b97f4a7c15ULL;
        x = (x ^ (x >> 30)) * 0xbf58476d1ce4e5b9ULL;
        x = (x ^ (x >> 27)) * 0x94d049bb133111ebULL;
        return x ^ (x >> 31);
    }

    template <typename T1, typename T2>
    size_t operator()(const pair<T1, T2>& p) const {
        static const uint64_t FIXED_RANDOM =
            chrono::steady_clock::now().time_since_epoch().count();
        uint64_t h1 = splitmix64((uint64_t)p.first + FIXED_RANDOM);
        uint64_t h2 = splitmix64((uint64_t)p.second + FIXED_RANDOM);
        return (size_t)(h1 ^ (h2 << 1));
    }
};

// 2. Lightweight Alternative (for non-adversarial inputs / offline tasks)
struct FastPairHash {
    size_t operator()(const pair<long long, long long>& p) const {
        return (size_t)((uint64_t)p.first ^ ((uint64_t)p.second << 32));
    }
};
```

---

## Usage

### 1. Declare `unordered_map` and `unordered_set` with `PairHash`
Pass `PairHash` as the 3rd template argument for `unordered_map` and the 2nd template argument for `unordered_set`:
```cpp
unordered_map<pair<long long, long long>, int, PairHash> freq;
unordered_set<pair<int, int>, PairHash> visited;
```

### 2. Storing Double Rolling Hashes (`pair<ll, ll>`)
Pair `PairHash` directly with `RollingHash::Hash` or `HashSegTree::Hash` to count distinct substring hashes in $O(1)$ average time per insertion:
```cpp
unordered_set<pair<long long, long long>, PairHash> seenHashes;
seenHashes.reserve(n);
seenHashes.insert(rh.substringHash(l, r));
```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste PairHash struct here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    unordered_map<pair<long long, long long>, int, PairHash> edgeCount;
    edgeCount.reserve(1024);

    vector<pair<long long, long long>> pts = {
        {10, 20}, {-5, 1000000007LL}, {10, 20}, {0, 0}
    };

    for (auto& pt : pts) {
        edgeCount[pt]++;
    }

    cout << "Frequency of (10, 20): " << edgeCount[{10, 20}] << "\n"; // 2
    cout << "Unique pairs: " << edgeCount.size() << "\n";              // 3

    return 0;
}
```

---

## Important Notes

1. **Duplicate Struct Name in Original Snippet:** Both structs in the original snippet were named `PairHash`, which causes a redefinition compile error if pasted into the same file. Renamed the lightweight variant to `FastPairHash` and kept the `splitmix64` version as the primary `PairHash`.
2. **Undefined Behavior Fix on Signed Left Shift:** Shifting a signed negative `long long` (`p.second << 32`) is undefined behavior in C++. Casting `(uint64_t)p.second << 32` makes bit-shifting 100% well-defined for negative coordinates.
3. **Why `FIXED_RANDOM` Matters on Codeforces:** Without a runtime random seed (`FIXED_RANDOM`), `splitmix64` is deterministic, meaning a hacker can still reverse `splitmix64` offline to construct colliding keys for GCC's bucket sizes. Adding `chrono::steady_clock::now().time_since_epoch().count()` makes bucket collisions impossible to predict.
4. **Always Call `.reserve(N)`:** When inserting up to $N$ pairs (e.g., $N \ge 10^5$), call `mp.reserve(N);` upfront to avoid expensive mid-execution rehash spikes.