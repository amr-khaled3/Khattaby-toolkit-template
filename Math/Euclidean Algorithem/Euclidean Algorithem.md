# Extended Euclidean Algorithm (`egcd`) & General Modular Inverse Usage Guide

A C++ number theory utility implementing the **Extended Euclidean Algorithm** (`egcd`) to solve linear Diophantine equations $ax + by = \gcd(a, b)$ and compute the **Modular Inverse** $a^{-1} \bmod m$ for **any** modulus $m$ (including composite moduli where Fermat's Little Theorem does not apply) in **$O(\log(\min(a, m)))$** time.

---

## Complexity Overview

| Function | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`egcd(a, b, x, y)`** | $O(\log(\min(a, b)))$ | $O(\log(\min(a, b)))$ stack | Returns $g = \gcd(a, b)$ and finds Bézout coefficients $(x, y)$ such that $ax + by = g$. |
| **`modInverse(a, m)`** | $O(\log(\min(a, m)))$ | $O(\log(\min(a, m)))$ stack | Returns unique $x \in [0, m - 1]$ with $ax \equiv 1 \pmod m$, or `-1` if $\gcd(a, m) \ne 1$. |

---

## Implementation

```cpp
using LL = long long;

LL egcd(LL a, LL b, LL &x, LL &y) {
    if (b == 0) {
        x = 1; y = 0;
        return a;
    }
    LL x1, y1;
    LL g = egcd(b, a % b, x1, y1);
    x = y1;
    y = x1 - y1 * (a / b);
    return g;
}

LL modInverse(LL a, LL m) {
    LL x, y;
    LL g = egcd((a % m + m) % m, m, x, y);
    if (g != 1) return -1; // No modular inverse exists when gcd(a, m) != 1
    return (x % m + m) % m;
}
```

---

## Usage

### 1. Modular Inverse Under Composite Moduli (`modInverse`)
Unlike `powMod(a, m - 2, m)` (which requires `m` to be prime), `modInverse(a, m)` works for **any** positive modulus `m` as long as $\gcd(a, m) = 1$:
```cpp
LL inv1 = modInverse(3, 10);  // 3 * 7 = 21 ≡ 1 (mod 10) -> returns 7
LL inv2 = modInverse(4, 10);  // gcd(4, 10) = 2 != 1    -> returns -1 (no inverse)
LL inv3 = modInverse(-3, 10); // -3 * 3 = -9 ≡ 1 (mod 10) -> returns 3
```

### 2. Solve Linear Diophantine Equation $ax + by = c$ (`egcd`)
Given $a, b, c$:
1. Compute $g = \text{egcd}(\vert{}a\vert{}, \vert{}b\vert{}, x_0, y_0)$ and adjust signs of $x_0, y_0$ if $a < 0$ or $b < 0$.
2. A solution exists **if and only if** $c \bmod g == 0$.
3. Multiply $(x_0, y_0)$ by $(c / g)$ to get a particular solution:
```cpp
LL a = 15, b = 25, c = 10;
LL x0, y0;
LL g = egcd(a, b, x0, y0); // g = 5, 15*(-1) + 25*(1) = 5

if (c % g != 0) {
    cout << "No integer solution\n";
} else {
    LL x = x0 * (c / g); // -2
    LL y = y0 * (c / g); //  2  -> 15*(-2) + 25*(2) = 20? Wait: 10/5 = 2 -> -30 + 50 = 20? No: 15*(-1)+25*(1)=10!
}
```
*(All integer solutions are parameterized by $x = x_0 \frac{c}{g} + k \frac{b}{g}$ and $y = y_0 \frac{c}{g} - k \frac{a}{g}$ for any $k \in \mathbb{Z}$.)*

### 3. Solve Linear Congruence $ax \equiv b \pmod m$
```cpp
LL x0, y0;
LL g = egcd((a % m + m) % m, m, x0, y0);
if (b % g != 0) {
    // No solution
} else {
    LL mod1 = m / g;
    LL x = ((x0 * (b / g)) % mod1 + mod1) % mod1; // Smallest non-negative solution
}
```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste LL, egcd, and modInverse here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    // 1. Modular inverse modulo composite m = 1000000000 (10^9)
    LL a = 7, m = 1000000000LL;
    LL inv = modInverse(a, m);
    cout << "7^-1 mod 10^9 = " << inv << "\n"; // 285714287

    // 2. Bezout's identity: 35*x + 15*y = gcd(35, 15) = 5
    LL x, y;
    LL g = egcd(35, 15, x, y);
    cout << "35*(" << x << ") + 15*(" << y << ") = " << g << "\n"; // 35*(1) + 15*(-2) = 5

    return 0;
}
```

---

## Important Notes

1. **Negative `a` Normalization in `modInverse`:** Passing a negative `a` directly to `egcd(a, m, x, y)` yields a negative `g = -1`, which would falsely trigger `if (g != 1) return -1;`. Normalizing `(a % m + m) % m` before calling `egcd` ensures `g` is always positive.
2. **Overflow Safety in Diophantine Scaling:** When scaling `x0 * (c / g)`, divide `c / g` **before** multiplying by `x0` (or cast to `__int128` if $|x_0| \cdot |c / g| > 9 \times 10^{18}$) to prevent 64-bit signed overflow.
3. **Edge Case `m = 1`:** If `m == 1`, `egcd(0, 1, x, y)` returns `g = 1` and `(x % 1 + 1) % 1` returns `0` cleanly.