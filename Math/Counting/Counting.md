# Combinatorics & Counting Functions Usage Guide

A C++ combinatorics reference covering modular factorials, permutations ($nPr$), combinations ($nCr$) for both small $N$ (with $O(1)$ queries) and large $N \le 10^{18}$ (with small $r$), Stars and Bars, Catalan numbers, and Derangements ($!n$) modulo a prime `MOD`.

---

## Complexity Overview

| Function | Time Complexity | Space Complexity | Constraints & Use Case |
| :--- | :--- | :--- | :--- |
| **`precomputeFactorials()`** | $O(N)$ | $O(N)$ | Precomputes `factorial[0..N-1]` and `inv_factorial[0..N-1]` in linear time. |
| **`nPr(n, r)` (Precomputed)** | $O(1)$ | $O(1)$ | Permutations $P(n, r) = \frac{n!}{(n - r)!} \bmod \text{MOD}$ for $0 \le r \le n < N$. |
| **`nCr(n, r)` (Precomputed)** | $O(1)$ | $O(1)$ | Binomial coefficient $\binom{n}{r} \bmod \text{MOD}$ for $0 \le r \le n < N$. |
| **`nCrLarge(n, r)`** | $O(\min(r, n - r))$ | $O(1)$ | Computes $\binom{n}{r} \bmod \text{MOD}$ when $n \le 10^{18}$ and $\min(r, n - r) \le 10^6$. |
| **`nPrLarge(n, r)`** | $O(r)$ | $O(1)$ | Computes $P(n, r) \bmod \text{MOD}$ when $n \le 10^{18}$ and $r \le 10^6$. |
| **`starsAndBars(n, k, allowEmpty)`** | $O(1)$ | $O(1)$ | Counts ways to distribute $n$ identical items into $k$ distinct bins. |
| **`catalan(n)`** | $O(1)$ | $O(1)$ | Computes the $n$-th Catalan number $C_n = \frac{1}{n + 1}\binom{2n}{n} \bmod \text{MOD}$ (requires $2n < N$). |
| **`precomputeDerangements()`** | $O(N)$ | $O(N)$ | Precomputes $!n$ (permutations of $n$ elements with no fixed points) for $0 \le n < N$. |

---

## Implementation

```cpp
using ll = long long;
const int N = 2e6 + 5; // Make sure N > 2 * max_n if using catalan(n) or starsAndBars(n, k)
const ll MOD = 1e9 + 7;

ll powMod(ll a, ll b, ll m = MOD) {
    ll res = 1;
    a %= m;
    while (b > 0) {
        if (b & 1) res = (res * a) % m;
        a = (a * a) % m;
        b >>= 1;
    }
    return res;
}

ll modInv(ll a, ll m = MOD) {
    return powMod(a, m - 2, m);
}

// 1. Precomputed Factorials & Inverse Factorials: O(N)
ll factorial[N];
ll inv_factorial[N];

void precomputeFactorials() {
    factorial[0] = 1;
    for (int i = 1; i < N; i++) {
        factorial[i] = (factorial[i - 1] * i) % MOD;
    }
    inv_factorial[N - 1] = powMod(factorial[N - 1], MOD - 2);
    for (int i = N - 2; i >= 0; i--) {
        inv_factorial[i] = (inv_factorial[i + 1] * (i + 1)) % MOD;
    }
}

// 2. Permutations & Combinations (Precomputed): O(1)
ll nPr(int n, int r) {
    if (r < 0 || n < 0 || r > n) return 0;
    return (factorial[n] * inv_factorial[n - r]) % MOD;
}

ll nCr(int n, int r) {
    if (r < 0 || n < 0 || r > n) return 0;
    ll ans = (factorial[n] * inv_factorial[r]) % MOD;
    ans = (ans * inv_factorial[n - r]) % MOD;
    return ans;
}

// 3. Combinations & Permutations for Large N (up to 1e18) and Small r
ll nCrLarge(ll n, ll r) {
    if (r < 0 || n < 0 || r > n) return 0;
    r = min(r, n - r);
    ll ans = 1;
    for (ll i = 1; i <= r; i++) {
        ans = ans * ((n - i + 1) % MOD) % MOD;
        ans = ans * modInv(i) % MOD;
    }
    return ans;
}

ll nPrLarge(ll n, ll r) {
    if (r < 0 || n < 0 || r > n) return 0;
    ll ans = 1;
    for (ll i = 0; i < r; i++) {
        ans = (ans * ((n - i) % MOD)) % MOD;
    }
    return ans;
}

// 4. Stars and Bars: O(1)
ll starsAndBars(ll n, ll k, bool allowEmpty = true) {
    if (k <= 0) return (n == 0 && k == 0) ? 1 : 0;
    if (allowEmpty) {
        if (n < 0) return 0;
        return nCr(n + k - 1, k - 1);
    }
    if (n < k) return 0;
    return nCr(n - 1, k - 1);
}

// 5. Catalan Numbers: O(1)
ll catalan(int n) {
    if (n < 0) return 0;
    ll ans = nCr(2 * n, n);
    return (ans * modInv(n + 1)) % MOD;
}

// 6. Derangements (!n): O(N)
ll derangement[N];

void precomputeDerangements() {
    derangement[0] = 1;
    if (N > 1) derangement[1] = 0;
    for (int i = 2; i < N; i++) {
        derangement[i] = ((i - 1) * ((derangement[i - 1] + derangement[i - 2]) % MOD)) % MOD;
    }
}
```

---

## Usage

### 1. Precompute Once at Startup
Call `precomputeFactorials()` (and `precomputeDerangements()` if needed) **once** at the beginning of `main()` before reading test cases:
```cpp
precomputeFactorials();
precomputeDerangements();
```

### 2. Standard $nCr$ & $nPr$ vs. Large-$N$ $nCr$ & $nPr$
```cpp
ll c1 = nCr(10, 3);                     // 120 (O(1) lookup)
ll p1 = nPr(10, 3);                     // 720 (O(1) lookup)

ll c2 = nCrLarge(1000000000000000000LL, 4); // N = 10^18, r = 4 (O(r))
ll p2 = nPrLarge(1000000000000000000LL, 4); // N = 10^18, r = 4 (O(r))
```

### 3. Stars and Bars (Distributing $n$ Identical Objects into $k$ Distinct Bins)
* **Non-negative solutions** ($x_1 + x_2 + \dots + x_k = n$, $x_i \ge 0$):
  ```cpp
  ll waysEmptyAllowed = starsAndBars(5, 3, true);   // C(5 + 3 - 1, 3 - 1) = C(7, 2) = 21
  ```
* **Positive solutions** ($x_1 + x_2 + \dots + x_k = n$, $x_i \ge 1$):
  ```cpp
  ll waysNonEmpty = starsAndBars(5, 3, false);      // C(5 - 1, 3 - 1) = C(4, 2) = 6
  ```

### 4. Catalan Numbers & Derangements
* **Catalan Numbers** ($1, 1, 2, 5, 14, 42, \dots$ — valid bracket sequences of $n$ pairs, full binary trees with $n + 1$ leaves, monotonic grid paths not crossing diagonal):
  ```cpp
  ll cat4 = catalan(4); // 14
  ```
* **Derangements** ($1, 0, 1, 2, 9, 44, \dots$ — permutations where $p_i \ne i$ for all $i$):
  ```cpp
  ll der4 = derangement[4]; // 9
  ```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste Combinatorics functions here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    precomputeFactorials();
    precomputeDerangements();

    cout << "nCr(6, 2) = " << nCr(6, 2) << "\n";                         // 15
    cout << "nPr(6, 2) = " << nPr(6, 2) << "\n";                         // 30
    cout << "nCrLarge(1e18, 3) = " << nCrLarge(1000000000000000000LL, 3) << "\n";
    cout << "Stars & Bars (n=5, k=3, empty): " << starsAndBars(5, 3, true) << "\n";   // 21
    cout << "Stars & Bars (n=5, k=3, non-empty): " << starsAndBars(5, 3, false) << "\n"; // 6
    cout << "Catalan(5) = " << catalan(5) << "\n";                       // 42
    cout << "Derangement(5) = " << derangement[5] << "\n";               // 44

    return 0;
}
```

---

## Important Notes

1. **Improvements & Fixes Applied from Original Snippets:**
   * **$O(N)$ Factorial Inversion:** The original loop called `powMod(factorial[i], MOD - 2)` on every iteration ($O(N \log \text{MOD})$). Computing `inv_factorial[N - 1]` once and propagating backwards (`inv_factorial[i] = inv_factorial[i + 1] * (i + 1) % MOD`) cuts precomputation to **$O(N)$**.
   * **Resolved Overload Ambiguity:** Renamed the $O(r)$ versions from `nCr(ll, ll)` / `nPr(ll, ll)` to `nCrLarge(ll, ll)` / `nPrLarge(ll, ll)` so passing `long long` variables for standard small queries does not accidentally trigger the slow $O(r)$ loop or cause ambiguous overload errors.
   * **Negative `r` Guards:** Added `r < 0 || n < 0` checks to the precomputed `nPr` and `nCr` to prevent negative out-of-bounds array indexing (`inv_factorial[r]`).
2. **Array Size `N` for `catalan` & `starsAndBars`:** Because `catalan(n)` queries `nCr(2 * n, n)` and `starsAndBars(n, k)` queries `nCr(n + k - 1, k - 1)`, make sure `N` is strictly greater than $\max(2n, n + k)$.
3. **Prime Modulus Requirement:** All modular inverse divisions (`inv_factorial`, `modInv(i)`, `modInv(n + 1)`) require `MOD` to be prime (e.g., $10^9 + 7$ or $998244353$).