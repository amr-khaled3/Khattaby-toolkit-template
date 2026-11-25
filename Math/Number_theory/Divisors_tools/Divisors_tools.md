# Number Theory Utility Functions Guide

A collection of essential C++ number theory functions for finding divisors, generating multiples, and precomputing divisor counts up to a fixed limit.

---

## Complexity Overview

| Function | Time Complexity | Space Complexity | Use Case |
| :--- | :--- | :--- | :--- |
| **`getDivisors(n)`** | $O(\sqrt{n})$ | $O(d(n))$ | Find all divisors of a single number up to $10^{14}$. |
| **`getMultiples(x, n)`** | $O(n / x)$ | $O(n / x)$ | Generate all multiples of $x$ not exceeding $n$. |
| **`divisorCounter()`** | $O(N \log N)$ | $O(N)$ | Precompute the number of divisors for every integer in $[1, N]$. |

---

## Implementation

```cpp
using LL = long long;

vector<LL> getDivisors(LL n) {
    vector<LL> ret;
    LL i = 1;
    for (; i * i < n; i++) {
        if (n % i == 0) {
            ret.push_back(i);
            ret.push_back(n / i);
        }
    }
    if (i * i == n) ret.push_back(i);
    return ret;
}

vector<int> getMultiples(int x, int n) {
    vector<int> ret;
    for (int i = x; i <= n; i += x)
        ret.push_back(i);
    return ret;
}

vector<LL> divisorCounter() {
    const int MAX = 1e6 + 6;
    vector<LL> divisorCount(MAX + 1, 0);
    for (int i = 1; i <= MAX; ++i) {
        for (int j = i; j <= MAX; j += i) {
            divisorCount[j]++;
        }
    }
    return divisorCount;
}
```

---

## Usage

### 1. `getDivisors(n)`
Returns a `vector<LL>` containing all positive divisors of `n`.
```cpp
vector<LL> divs = getDivisors(12);
// divs contains: {1, 12, 2, 6, 3, 4} (unsorted)
```

### 2. `getMultiples(x, n)`
Returns a `vector<int>` containing all positive multiples of `x` that are less than or equal to `n`.
```cpp
vector<int> mults = getMultiples(3, 15);
// mults contains: {3, 6, 9, 12, 15}
```

### 3. `divisorCounter()`
Precomputes a lookup table where `divCount[k]` stores the total number of divisors of `k` for all $1 \le k \le 10^6 + 6$.
```cpp
vector<LL> divCount = divisorCounter();
LL countOf12 = divCount[12]; // 6
```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

using LL = long long;

// Paste function definitions here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    vector<LL> divs = getDivisors(36);
    sort(divs.begin(), divs.end());
    for (LL d : divs) cout << d << " ";
    cout << "\n";

    vector<int> mults = getMultiples(4, 20);
    for (int m : mults) cout << m << " ";
    cout << "\n";

    vector<LL> divCount = divisorCounter();
    cout << divCount[36] << "\n";

    return 0;
}
```

---

## Important Notes

1. **Bug Fix in `getMultiples`:** The original snippet pushed `n` (`ret.push_back(n)`) instead of `i` (`ret.push_back(i)`). The implementation above fixes this to push the actual multiples. Also ensure $x \ge 1$ before calling `getMultiples` to avoid an infinite loop.
2. **Unsorted Output in `getDivisors`:** Because divisors are pushed in pairs `(i, n / i)`, the returned vector is **not sorted**. Call `sort(divs.begin(), divs.end())` if ordered divisors are required.
3. **Precomputation Scope:** Call `divisorCounter()` **once** outside test cases (e.g., at the start of `main()`). Calling it inside a `TEST` loop will cause Time Limit Exceeded (TLE).
4. **Sieve Adaptation:** The nested loop structure in `divisorCounter()` can easily be adapted to precompute the **sum of divisors** (`sumDiv[j] += i`) or **list of all divisors** (`divs[j].push_back(i)`) in $O(N \log N)$.