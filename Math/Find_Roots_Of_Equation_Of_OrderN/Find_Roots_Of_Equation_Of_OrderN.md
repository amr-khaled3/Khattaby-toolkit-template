# Polynomial Evaluation & Bisection Root Finder Usage Guide

A C++ numerical methods utility for evaluating real-valued polynomials $P(x) = \sum_{i=0}^{d} c_i x^i$ and locating a real root inside a bracketed interval $[l, r]$ using the **Bisection Method** in **$O(K \cdot d)$** time ($K = 100$ fixed iterations).

---

## Complexity Overview

| Function | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`evalPoly(coeffs, x)`** | $O(d)$ | $O(1)$ | Evaluates $P(x)$ of degree $d = \Vert{}\text{coeffs}\Vert{} - 1$ (`coeffs[i]` = coefficient of $x^i$). |
| **`findRoot(coeffs, l, r)`** | $O(K \cdot d)$ | $O(1)$ | Runs up to $K = 100$ bisection steps, shrinking interval width by $2^{-100} \approx 7.8 \times 10^{-31}$ (`NAN` if $f(l)$ and $f(r)$ share the same sign). |

---

## Implementation

```cpp
const double EPS = 1e-12;

double evalPoly(const vector<double> &coeffs, double x) {
    double res = 0;
    double xn = 1;
    for (auto c : coeffs) {
        res += c * xn;
        xn *= x;
    }
    return res;
}

double findRoot(const vector<double> &coeffs, double l, double r) {
    double fl = evalPoly(coeffs, l);
    double fr = evalPoly(coeffs, r);
    if (fl * fr > 0) return NAN; 

    for (int iter = 0; iter < 100; iter++) {
        double mid = (l + r) / 2;
        double fmid = evalPoly(coeffs, mid);
        if (fabs(fmid) < EPS) return mid;
        if (fl * fmid <= 0) {
            r = mid;
            fr = fmid;
        } else {
            l = mid;
            fl = fmid;
        }
    }
    return (l + r) / 2;
}
```

---

## Usage

### 1. Polynomial Representation & Evaluation (`evalPoly`)
Store coefficients from **lowest degree ($x^0$) to highest degree ($x^d$)**:
```cpp
// P(x) = -6 + 11x - 6x^2 + x^3 = (x - 1)(x - 2)(x - 3)
vector<double> poly = {-6.0, 11.0, -6.0, 1.0};

double val = evalPoly(poly, 2.0); // 0.0
```

### 2. Finding a Bracketed Root (`findRoot`)
Pass an interval $[l, r]$ where $P(l)$ and $P(r)$ have opposite signs ($P(l) \cdot P(r) \le 0$):
```cpp
double root = findRoot(poly, 1.5, 2.5); // ~2.0

if (isnan(root)) {
    cout << "No sign change in [l, r]\n";
} else {
    cout << fixed << setprecision(9) << root << "\n";
}
```

### 3. Finding All Real Roots of a Degree-$d$ Polynomial
Differentiate $P(x)$ recursively down to degree $1$ (linear): the $k$ real roots of $P'(x)$ partition $(-\infty, +\infty)$ into $k + 1$ monotonic intervals, each containing at most one root of $P(x)$ that can be isolated via `findRoot`.

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste EPS, evalPoly, and findRoot here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    // Solve x^3 - 2x - 5 = 0 on [2, 3]
    vector<double> coeffs = {-5.0, -2.0, 0.0, 1.0};

    double root = findRoot(coeffs, 2.0, 3.0);

    cout << fixed << setprecision(10);
    if (!isnan(root)) {
        cout << "Root: " << root << "\n";                 // 2.0945514815
        cout << "P(root): " << evalPoly(coeffs, root) << "\n";
    }

    return 0;
}
```

---

## Important Notes

1. **Required Constant `EPS`:** `findRoot` references `EPS` (e.g., `const double EPS = 1e-12;`). Ensure `EPS` is declared globally before `findRoot`.
2. **Even-Multiplicity Roots & Same-Sign Endpoints:** Bisection requires $P(l) \cdot P(r) \le 0$. If an interval contains an even number of roots (e.g., $P(x) = (x - 2)^2$ on $[1, 3]$), $P(l)$ and $P(r)$ have the same sign and `findRoot` returns `NAN`.
3. **Horner's Method for Better Numerical Stability:** For high-degree polynomials ($d \ge 15$) or large $\vert{}x\vert{}$, `xn *= x` can overflow or lose precision. Rewriting `evalPoly` using Horner's rule avoids computing high powers directly:
   ```cpp
   double evalPoly(const vector<double> &coeffs, double x) {
       double res = 0;
       for (int i = (int)coeffs.size() - 1; i >= 0; i--)
           res = res * x + coeffs[i];
       return res;
   }
   ```
4. **Overflow in `fl * fr > 0`:** When $|P(l)|$ and $|P(r)|$ are very large ($\approx 10^{200}$), `fl * fr` can overflow to `+inf`. Comparing signs directly via `(fl > 0) == (fr > 0) && fl != 0 && fr != 0` avoids intermediate product overflow.