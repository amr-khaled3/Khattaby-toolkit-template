# Fast Fourier Transform (FFT) Polynomial Multiplication Usage Guide

An iterative in-place C++ implementation of **Cooley-Tukey Fast Fourier Transform (FFT)** for multiplying two polynomials (or computing linear convolutions / sumset frequencies) in **$O(N \log N)$** time instead of naive $O(N^2)$ multiplication.

---

## Complexity Overview

| Function | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`fft(a, invert)`** | $O(N \log N)$ | $O(1)$ auxiliary | In-place iterative bit-reversal permutation + butterfly stages ($N$ must be a power of 2). |
| **`multiply(a, b)`** | $O((N + M) \log(N + M))$ | $O(N + M)$ | Multiplies polynomials of sizes $N = \Vert{}a\Vert{}$ and $M = \Vert{}b\Vert{}$, returning a vector of size $N + M - 1$. |

---

## Implementation

```cpp
using ll = long long;
using cd = complex<double>;
const double PI = acos(-1.0);

void fft(vector<cd>& a, bool invert) {
    int n = (int)a.size();

    for (int i = 1, j = 0; i < n; i++) {
        int bit = n >> 1;
        while (j & bit) {
            j ^= bit;
            bit >>= 1;
        }
        j ^= bit;

        if (i < j)
            swap(a[i], a[j]);
    }

    for (int len = 2; len <= n; len <<= 1) {
        double ang = 2 * PI / len * (invert ? -1 : 1);
        cd wlen(cos(ang), sin(ang));

        for (int i = 0; i < n; i += len) {
            cd w(1);

            for (int j = 0; j < len / 2; j++) {
                cd u = a[i + j];
                cd v = a[i + j + len / 2] * w;

                a[i + j] = u + v;
                a[i + j + len / 2] = u - v;

                w *= wlen;
            }
        }
    }

    if (invert) {
        for (cd& x : a)
            x /= n;
    }
}

vector<ll> multiply(const vector<ll>& a, const vector<ll>& b) {
    if (a.empty() || b.empty()) return {};

    vector<cd> fa(a.begin(), a.end());
    vector<cd> fb(b.begin(), b.end());

    int n = 1;
    while (n < (int)a.size() + (int)b.size())
        n <<= 1;

    fa.resize(n);
    fb.resize(n);

    fft(fa, false);
    fft(fb, false);

    for (int i = 0; i < n; i++)
        fa[i] *= fb[i];

    fft(fa, true);

    vector<ll> result((int)a.size() + (int)b.size() - 1);

    for (int i = 0; i < (int)result.size(); i++)
        result[i] = llround(fa[i].real());

    return result;
}
```

---

## Usage

### 1. Polynomial Multiplication
Represent polynomial $P(x) = c_0 + c_1 x + c_2 x^2 + \dots + c_d x^d$ as a `vector<ll>` where `a[i]` is the coefficient of $x^i$:
```cpp
// A(x) = 1 + 2x + 3x^2  -> {1, 2, 3}
// B(x) = 4 + 5x         -> {4, 5}
vector<ll> A = {1, 2, 3};
vector<ll> B = {4, 5};

vector<ll> C = multiply(A, B);
// C(x) = (1 + 2x + 3x^2)(4 + 5x) = 4 + 13x + 22x^2 + 15x^3 -> {4, 13, 22, 15}
```

### 2. Counting All Pairwise Sums ($a_i + b_j$)
Build frequency arrays `freqA` and `freqB` over value ranges $[0, V]$; then `ways[s]` gives the exact number of pairs $(i, j)$ with $a_i + b_j = s$:
```cpp
vector<ll> freqA(MAXV + 1, 0), freqB(MAXV + 1, 0);
for (int x : arrA) freqA[x]++;
for (int y : arrB) freqB[y]++;

vector<ll> ways = multiply(freqA, freqB);
// ways[s] = count of pairs (x, y) such that x + y == s
```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste FFT & multiply implementation here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    vector<ll> poly1 = {2, 0, 3, 1}; // 2 + 3x^2 + x^3
    vector<ll> poly2 = {1, 4, 2};    // 1 + 4x + 2x^2

    vector<ll> prod = multiply(poly1, poly2);
    // Expected: 2 + 8x + 7x^2 + 13x^3 + 10x^4 + 2x^5
    for (int i = 0; i < (int)prod.size(); i++) {
        cout << prod[i] << (i + 1 == (int)prod.size() ? "\n" : " ");
    }

    return 0;
}
```

---

## Important Notes

1. **Required Type Aliases & Constants:** Ensure `using cd = complex<double>;` and `const double PI = acos(-1.0);` are defined above `fft`.
2. **Floating-Point Precision Limits:** IEEE-754 `double` has 53 bits of significand precision ($\approx 9 \times 10^{15}$). Because FFT accumulates rounding errors across $\log_2 N$ stages, `llround` stays exact as long as output coefficients satisfy $\max |c_k| \lesssim 10^{13}$ (e.g., $N \le 2 \times 10^5$ with input values $\le 10^4$).
   * If coefficients exceed $10^{13}$, switch `cd` to `complex<long double>`.
   * If a problem asks for polynomial multiplication **modulo a prime** like `998244353` with large coefficients ($10^9$), use **Number Theoretic Transform (NTT)** instead to eliminate all floating-point error.
3. **Power-of-Two Requirement on Direct `fft` Calls:** `multiply` automatically pads `fa` and `fb` to the next power of two. If calling `fft(a, invert)` directly, `a.size()` **must** be a power of two.
4. **Empty Vector Guard:** Added `if (a.empty() || b.empty()) return {};` to `multiply` so empty inputs do not create a `vector<ll>` of size `-1`.