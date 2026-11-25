# Modular Arithmetic & Binary Exponentiation Usage Guide

A C++ utility collection for safe modular arithmetic under a configurable modulus `MOD` (defaulting to $10^9 + 7$, with optional custom modulus `m` per call). Handles negative operands, prevents intermediate 64-bit overflows before multiplication, and supports fast modular exponentiation, modular inverses, and modular division.

---

## Complexity Overview

| Function | Time Complexity | Space Complexity | Description |
| :--- | :--- | :--- | :--- |
| **`mod(a, m)`** | $O(1)$ | $O(1)$ | Normalizes any signed 64-bit integer `a` into $[0, m - 1]$. |
| **`addMod(a, b, m)`** | $O(1)$ | $O(1)$ | Computes $(a + b) \bmod m$ safely for negative/large inputs. |
| **`subMod(a, b, m)`** | $O(1)$ | $O(1)$ | Computes $(a - b) \bmod m$ without returning negative remainders. |
| **`mulMod(a, b, m)`** | $O(1)$ | $O(1)$ | Computes $(a \times b) \bmod m$ after reducing both operands mod $m$. |
| **`powMod(a, b, m)`** | $O(\log b)$ | $O(1)$ | Binary exponentiation computing $a^b \bmod m$ for $b \ge 0$. |
| **`modInverse(a, m)`** | $O(\log m)$ | $O(1)$ | Computes $a^{-1} \bmod m = a^{m - 2} \bmod m$ via Fermat's Little Theorem (**prime `m` only**). |
| **`divMod(a, b, m)`** | $O(\log m)$ | $O(1)$ | Computes $(a \cdot b^{-1}) \bmod m$ (**prime `m` only**, $b \not\equiv 0 \pmod m$). |

---

## Implementation

```cpp
using ll = long long;
const ll MOD = 1e9 + 7; // Change to 998244353 or pass as a parameter if needed

// 1. Safe Modulo (handles negative numbers properly)
ll mod(ll a, ll m = MOD) {
    return ((a % m) + m) % m;
}

// 2. Addition
ll addMod(ll a, ll b, ll m = MOD) {
    return mod(mod(a, m) + mod(b, m), m);
}

// 3. Subtraction
ll subMod(ll a, ll b, ll m = MOD) {
    return mod(mod(a, m) - mod(b, m), m);
}

// 4. Multiplication
ll mulMod(ll a, ll b, ll m = MOD) {
    return mod(mod(a, m) * mod(b, m), m);
}

// 5. Binary Exponentiation (a^b % m)
ll powMod(ll a, ll b, ll m = MOD) {
    ll res = 1;
    a = mod(a, m); // Prevents overflow on first (a * a) if a >= 2^32
    while (b > 0) {
        if (b & 1) res = mulMod(res, a, m);
        a = mulMod(a, a, m);
        b >>= 1;
    }
    return res;
}

// 6. Modular Inverse (Fermat's Little Theorem - requires 'm' to be prime)
ll modInverse(ll a, ll m = MOD) {
    return powMod(a, m - 2, m);
}

// 7. Division (a / b % m)
ll divMod(ll a, ll b, ll m = MOD) {
    return mulMod(a, modInverse(b, m), m);
}
```

---

## Usage

### 1. Basic Arithmetic (`mod`, `addMod`, `subMod`, `mulMod`)
Omit the last argument to use the global `MOD` ($10^9 + 7$), or pass a custom modulus (such as `998244353`) as the final argument:
```cpp
ll x = mod(-17);                         // 999999990
ll s = subMod(3, 10);                    // 1000000000
ll p = mulMod(1000000000, 1000000000);   // 49
ll custom = addMod(500000000, 600000000, 998244353); // 101755647
```

### 2. Fast Power, Inverse & Division (`powMod`, `modInverse`, `divMod`)
```cpp
ll p2  = powMod(2, 60);          // 2^60 % (1e9 + 7)
ll inv = modInverse(2);          // 500000004 (since 2 * 500000004 ≡ 1 mod 1e9+7)
ll q   = divMod(10, 4);          // 500000006 (equivalent to 5 / 2 mod 1e9+7)
```



## Important Notes

1. **Prime Modulus Requirement for `modInverse` & `divMod`:** Fermat's Little Theorem ($a^{m - 2} \equiv a^{-1} \pmod m$) works **only when $m$ is prime** (such as $10^9 + 7$ or $998244353$) and $a \not\equiv 0 \pmod m$. If $m$ is composite, use the **Extended Euclidean Algorithm** instead.
2. **Large Modulus Overflow ($m > 3 \times 10^9$):** `mulMod` multiplies two values up to $m - 1$ inside signed 64-bit `long long` (`LLONG_MAX` $\approx 9.22 \times 10^{18}$), which is completely safe for $m \le 3 \times 10^9$. If a problem uses a large modulus like $m \approx 10^{18}$, cast to `__int128` inside `mulMod`: `return (ll)((__int128)mod(a, m) * mod(b, m) % m);`.
3. **Edge Case $0^0 \bmod m$:** `powMod(0, 0, m)` returns `1 % m` (so `1` for $m > 1$, and `0` if $m = 1$).
4. **Loop Performance Tip:** Inside tight $O(N^2)$ DP loops, repeated function calls with modulo divisions (`%`) add constant-factor overhead; inline `if (x >= MOD) x -= MOD;` when maximum speed is needed.