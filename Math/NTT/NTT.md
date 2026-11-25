# Number Theoretic Transform (NTT) & CRT Convolution Usage Guide

A KACTL-style C++ implementation of **Number Theoretic Transform (NTT)** for exact polynomial multiplication modulo NTT-friendly primes of the form $c \cdot 2^k + 1$ (defaulting to $998244353 = 119 \cdot 2^{23} + 1$, primitive root $g = 62$), paired with **Chinese Remainder Theorem (CRT)** helpers for arbitrary moduli (e.g., $10^9 + 7$) or exact large-coefficient convolutions without floating-point precision loss.

---

## Complexity Overview

| Function | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`modpow(b, e, m)`** | $O(\log e)$ | $O(1)$ | Binary exponentiation computing $b^e \bmod m$. |
| **`generator()`** | $O(\sqrt{p} + g \log p)$ | $O(\omega(p - 1))$ | Finds the smallest primitive root $g$ modulo $p$. |
| **`ntt(a)`** | $O(N \log N)$ | $O(N)$ | In-place forward NTT ($N$ must be a power of 2). |
| **`conv(a, b)`** | $O((N + M) \log(N + M))$ | $O(N + M)$ | Multiplies polynomials `a` and `b` modulo `mod`, returning a vector of size $\Vert{}a\Vert{} + \Vert{}b\Vert{} - 1$. |
| **`CRT(a, m1, b, m2)`** | $O(\log(\max(m_1, m_2)))$ | $O(1)$ | Reconstructs $x \in [0, m_1 m_2 - 1]$ satisfying $x \equiv a \pmod{m_1}$ and $x \equiv b \pmod{m_2}$. |

---

## Implementation

```cpp
using ll = long long;

const ll mod = (119 << 23) + 1, root = 62; // = 998244353
// Other NTT-friendly primes (p = c * 2^k + 1):
// 5 << 25 + 1 = 167772161  (root = 3)
// 7 << 26 + 1 = 469762049  (root = 3)
// 479 << 21 + 1 = 1004535809 (root = 3)
// 483 << 21 + 1 = 1012924417 (root = 5)

ll modpow(ll b, ll e) {
    ll ans = 1;
    for (; e; b = b * b % mod, e /= 2)
        if (e & 1)
            ans = ans * b % mod;
    return ans;
}

int modpow(ll b, ll e, ll m) {
    ll ans = 1;
    for (b %= m; e; b = b * b % m, e /= 2)
        if (e & 1)
            ans = ans * b % m;
    return (int)ans;
}

// Primitive Root of prime mod of form 2^a * b + 1
int generator() {
    vector<int> fact;
    int phi = mod - 1, n = phi;
    for (int i = 2; i * i <= n; ++i)
        if (n % i == 0) {
            fact.push_back(i);
            while (n % i == 0)
                n /= i;
        }
    if (n > 1)
        fact.push_back(n);
    for (int res = 2; res <= mod; ++res) {
        bool ok = true;
        for (size_t i = 0; i < fact.size() && ok; ++i)
            ok &= modpow(res, phi / fact[i]) != 1;
        if (ok)
            return res;
    }
    return -1;
}

void ntt(vector<int> &a) {
    int n = (int)a.size(), L = 31 - __builtin_clz(n);
    static vector<int> rt(2, 1); // Remove `static` if switching `mod` at runtime
    for (static int k = 2, s = 2; k < n; k *= 2, s++) {
        rt.resize(n);
        int z[] = {1, modpow(root, mod >> s, mod)};
        for (int i = k; i < 2 * k; ++i)
            rt[i] = (ll)rt[i / 2] * z[i & 1] % mod;
    }
    vector<int> rev(n);
    for (int i = 0; i < n; ++i)
        rev[i] = (rev[i / 2] | (i & 1) << L) / 2;
    for (int i = 0; i < n; ++i)
        if (i < rev[i])
            swap(a[i], a[rev[i]]);
    for (int k = 1; k < n; k *= 2) {
        for (int i = 0; i < n; i += 2 * k) {
            for (int j = 0; j < k; ++j) {
                int z = (ll)rt[j + k] * a[i + j + k] % mod, &ai = a[i + j];
                a[i + j + k] = ai - z + (z > ai ? mod : 0);
                ai += (ai + z >= mod ? z - mod : z);
            }
        }
    }
}

vector<int> conv(const vector<int> &a, const vector<int> &b) {
    if (a.empty() || b.empty())
        return {};
    int s = (int)a.size() + (int)b.size() - 1, B = 32 - __builtin_clz(s), n = 1 << B;
    int inv = modpow(n, mod - 2, mod);
    vector<int> L(a), R(b), out(n);
    L.resize(n), R.resize(n);
    ntt(L), ntt(R);
    for (int i = 0; i < n; ++i)
        out[-i & (n - 1)] = (ll)L[i] * R[i] % mod * inv % mod;
    ntt(out);
    return {out.begin(), out.begin() + s};
}

// Reconstructs x mod (m1 * m2) given x ≡ a (mod m1) and x ≡ b (mod m2)
ll CRT(ll a, ll m1, ll b, ll m2) {
    __int128 m = (__int128)m1 * m2;
    __int128 ans = (__int128)a * m2 % m * modpow(m2, m1 - 2, m1) % m
                 + (__int128)m1 * b % m * modpow(m1, m2 - 2, m2) % m;
    return (ll)(ans % m);
}
```

---

## Usage

### 1. Polynomial Multiplication Modulo $998244353$ (`conv`)
Represent $A(x) = \sum a_i x^i$ and $B(x) = \sum b_i x^i$ with coefficients normalized in $[0, \text{mod} - 1]$:
```cpp
vector<int> A = {1, 2, 3}; // 1 + 2x + 3x^2
vector<int> B = {4, 5};    // 4 + 5x

vector<int> C = conv(A, B);
// C = {4, 13, 22, 15} (all coefficients reduced mod 998244353)
```

### 2. Arbitrary Modulus Convolution (e.g., $10^9 + 7$) via 3-Prime CRT
When a problem requires convolution modulo `1000000007` (which is not NTT-friendly because $10^9 + 6 = 2 \cdot 500000003$ has only one factor of 2):
1. Make `mod` and `root` mutable variables and remove `static` inside `ntt` (or clear `rt` when switching primes).
2. Run `conv(a, b)` independently under three NTT primes whose product $M = m_1 m_2 m_3 \approx 5.95 \times 10^{25} > N \cdot \text{desired\_mod}^2$:
   * $m_1 = 167772161$ (`root1 = 3`)
   * $m_2 = 469762049$ (`root2 = 3`)
   * $m_3 = 754974721$ (`root3 = 11`)
3. Combine each coefficient triplet `(c1[i], c2[i], c3[i])` using the 3-prime `CRT` helper to recover the exact value before taking `% desired_mod`.

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste NTT & conv implementation here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    vector<int> a = {1, 2, 3, 4};
    vector<int> b = {5, 6, 7};

    vector<int> res = conv(a, b);
    // (1 + 2x + 3x^2 + 4x^3)(5 + 6x + 7x^2) = 5 + 16x + 34x^2 + 52x^3 + 45x^4 + 28x^5
    for (int i = 0; i < (int)res.size(); i++) {
        cout << res[i] << (i + 1 == (int)res.size() ? "\n" : " ");
    }

    return 0;
}
```

---

## Important Notes

1. **Syntax & Overflow Fixes Applied from Original Snippet:**
   * **Broken Line-Wrap Comment:** Fixed `// erase the static if you want \n to use two moduli;` which had an uncommented second line causing a syntax error.
   * **32-Bit Overflow in `CRT`:** In `CRT(ll a, ll m1, ll b, ll m2)`, `a * m2 % m` overflowed 64-bit `ll` before `% m` when $a, m_2 \approx 10^9$. Cast operands to `__int128` before multiplying.
   * **Negative Input Coefficients:** Ensure all elements in `a` and `b` are already normalized into $[0, \text{mod} - 1]$ (`(x % mod + mod) % mod`) before calling `conv`, as negative values in `a[i]` break the butterfly subtraction `ai - z + (z > ai ? mod : 0)`.
2. **Maximum Supported Polynomial Size:** For $998244353 = 119 \cdot 2^{23} + 1$, the maximum transform length is $n \le 2^{23} = 8,388,608$ (so $\|a\| + \|b\| - 1 \le 2^{23}$).
3. **Why `out[-i & (n - 1)]` Works Without a Separate Inverse NTT:** Reversing indices $1 \dots n - 1$ after pointwise multiplication (`out[-i & (n - 1)]`) evaluates the forward NTT at $\omega^{-k}$, which performs the inverse transform using the exact same `ntt()` function.