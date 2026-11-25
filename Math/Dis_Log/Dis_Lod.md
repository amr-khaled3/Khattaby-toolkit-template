# Discrete Logarithm (BSGS) & Multiplicative Order Usage Guide

A C++ number theory utility collection implementing **Baby-Step Giant-Step (BSGS)** for solving discrete logarithms $a^x \equiv b \pmod m$ (including non-coprime $\gcd(a, m) > 1$), Euler's Totient $\varphi(n)$, unique prime factorization, and the **multiplicative order** $\operatorname{ord}_m(a)$ (the smallest positive integer $x \ge 1$ such that $a^x \equiv 1 \pmod m$).

---

## Complexity Overview

| Function | Time Complexity | Space Complexity | Description |
| :--- | :--- | :--- | :--- |
| **`discrete_log(a, b, m)`** | $O(\sqrt{m})$ average | $O(\sqrt{m})$ | Finds the smallest integer $x \ge 0$ satisfying $a^x \equiv b \pmod m$, or `-1` if none exists. |
| **`fast_power(a, b, m)`** | $O(\log b)$ | $O(1)$ | Binary exponentiation computing $a^b \bmod m$. |
| **`phi(n)`** | $O(\sqrt{n})$ | $O(1)$ | Computes Euler's Totient $\varphi(n)$ for a single integer $n$. |
| **`factor(n)`** | $O(\sqrt{n})$ | $O(\omega(n))$ | Returns all distinct prime factors of $n$ in ascending order. |
| **`multiplicative_order(a, m)`** | $O(\sqrt{m} + \log^2 m)$ | $O(\omega(\varphi(m)))$ | Computes the smallest $x \ge 1$ such that $a^x \equiv 1 \pmod m$ (requires $\gcd(a, m) = 1$, $m > 1$). |

---

## Implementation

```cpp
using ll = long long;

// Returns minimum x >= 0 for which a^x % m == b % m (or -1 if no solution exists)
ll discrete_log(ll a, ll b, ll m) {
    if (m == 1) return 0;
    a %= m, b %= m;
    if (a == 0) return b == 0 ? 1 : -1;

    ll k = 1, add = 0, g;
    while ((g = gcd(a, m)) > 1) {
        if (b == k) return add;
        if (b % g) return -1;
        b /= g, m /= g, ++add;
        k = (k * (a / g)) % m;
    }

    ll n = sqrt(m) + 1;
    ll an = 1;
    for (ll i = 0; i < n; ++i)
        an = (an * a) % m;

    unordered_map<ll, ll> vals;
    vals.reserve(n + 1);
    for (ll q = 0, cur = b; q <= n; ++q) {
        vals[cur] = q;
        cur = (cur * a) % m;
    }

    for (ll p = 1, cur = k; p <= n; ++p) {
        cur = (cur * an) % m;
        if (vals.count(cur)) {
            return n * p - vals[cur] + add;
        }
    }
    return -1;
}

ll fast_power(ll a, ll b, ll m) {
    a %= m;
    ll res = 1;
    while (b) {
        if (b & 1) res = (res * a) % m;
        a = (a * a) % m;
        b >>= 1;
    }
    return res;
}

ll phi(ll n) {
    ll result = n;
    for (ll i = 2; i * i <= n; i++) {
        if (n % i == 0) {
            while (n % i == 0)
                n /= i;
            result -= result / i;
        }
    }
    if (n > 1)
        result -= result / n;
    return result;
}

// Unique prime factors of n
vector<ll> factor(ll n) {
    vector<ll> f;
    for (ll i = 2; i * i <= n; i++) {
        if (n % i == 0) {
            f.push_back(i);
            while (n % i == 0) n /= i;
        }
    }
    if (n > 1) f.push_back(n);
    return f;
}

// Smallest x >= 1 such that a^x ≡ 1 (mod m), assuming gcd(a, m) == 1 and m > 1
ll multiplicative_order(ll a, ll m) {
    if (m == 1) return 1;
    if (gcd(a, m) != 1) return -1;

    ll x = phi(m);
    vector<ll> primes = factor(x);

    for (ll p : primes) {
        while (x % p == 0 && fast_power(a, x / p, m) == 1) {
            x /= p;
        }
    }
    return x;
}
```

---

## Usage

### 1. Solve $a^x \equiv b \pmod m$ (`discrete_log`)
Works for both prime and composite moduli (even when $\gcd(a, m) > 1$):
```cpp
ll x1 = discrete_log(2, 8, 13);   // 2^3 ≡ 8 (mod 13) -> returns 3
ll x2 = discrete_log(2, 3, 13);   // 2^4 = 16 ≡ 3 (mod 13) -> returns 4
ll x3 = discrete_log(6, 4, 10);   // 6^2 = 36 ≡ 6, 6^1 = 6; no x gives 4 -> returns -1
ll x4 = discrete_log(6, 6, 10);   // 6^1 ≡ 6 (mod 10) -> returns 1
```

### 2. Find the Period / Multiplicative Order (`multiplicative_order`)
Computes the smallest positive exponent $x \ge 1$ where $a^x \equiv 1 \pmod m$:
```cpp
ll ord1 = multiplicative_order(2, 7);   // 2^1=2, 2^2=4, 2^3=1 (mod 7) -> returns 3
ll ord2 = multiplicative_order(3, 7);   // 3 is a primitive root mod 7 -> returns 6
ll ord3 = multiplicative_order(2, 6);   // gcd(2, 6) != 1 -> returns -1
```

### 3. Check If `a` Is a Primitive Root Modulo `m`
`a` is a primitive root modulo `m` if and only if $\gcd(a, m) = 1$ and $\operatorname{ord}_m(a) = \varphi(m)$:
```cpp
bool isPrimitiveRoot(ll a, ll m) {
    return gcd(a, m) == 1 && multiplicative_order(a, m) == phi(m);
}
```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste Discrete Log & Multiplicative Order functions here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    // Solve 3^x ≡ 13 (mod 17)
    ll x = discrete_log(3, 13, 17);
    cout << "3^x ≡ 13 (mod 17) -> x = " << x << "\n"; // 4 (3^4 = 81 ≡ 13 mod 17)

    // Non-coprime discrete log: 4^x ≡ 16 (mod 24)
    ll xNonCoprime = discrete_log(4, 16, 24);
    cout << "4^x ≡ 16 (mod 24) -> x = " << xNonCoprime << "\n"; // 2

    // Multiplicative order of 10 modulo 7 (length of repeating decimal 1/7)
    cout << "Order of 10 mod 7: " << multiplicative_order(10, 7) << "\n"; // 6

    return 0;
}
```

---

## Important Notes

1. **Bug Fixes Applied from Original Snippets:**
   * **32-Bit Truncation in `discrete_log`:** The original Baby-Step and Giant-Step loops declared `for (int q = 0, cur = b; ...)` and `for (int p = 1, cur = k; ...)`, truncating `cur` to 32-bit `int` when $m > 2 \times 10^9$. Changed `cur`, `q`, and `p` to `ll`.
   * **32-Bit Types in `phi`, `factor`, and `multiplicative_order`:** Upgraded `int` parameters/variables (`n`, `i`, `x`, `p`) to `ll` so $m > 2 \times 10^9$ (up to $\sim 10^{12}$) does not overflow.
   * **Coprime Guard in `multiplicative_order`:** Added `if (gcd(a, m) != 1) return -1;` so non-coprime inputs (where no $a^x \equiv 1 \pmod m$ exists) return `-1` instead of a bogus divisor of $\varphi(m)$.
   * **Edge Case $m = 1$ & $a = 0$:** Added `if (m == 1) return 0;` before `if (a == 0)` in `discrete_log` (since $0^0 \equiv 0 \pmod 1$ gives $x = 0$).
2. **Minimum $x \ge 0$ vs. $x \ge 1$ in `discrete_log`:** `discrete_log(a, 1, m)` returns `0` (since $a^0 = 1$). If a problem strictly requires $x \ge 1$ when $b \equiv 1 \pmod m$, return `multiplicative_order(a, m)` instead.
3. **Modulus Limit for `discrete_log`:** BSGS runs in $O(\sqrt{m})$ time and memory, making it ideal for $m \le 10^{12}$ (or up to $10^{14}$ for single queries). For larger moduli, use **Pohlig–Hellman** when $\varphi(m)$ is smooth.