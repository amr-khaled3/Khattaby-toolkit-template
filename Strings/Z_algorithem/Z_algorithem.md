# Z-Function & Linear Pattern Matching Usage Guide

A linear-time C++ implementation of the **Z-Algorithm** (`zFunction`) that computes the Longest Common Prefix (LCP) between a string `s` and every suffix `s[i..n-1]` in **$O(N)$** time, paired with a **`search`** helper for finding all **0-based** occurrences of a `pattern` inside a `text` in **$O(N + M)$** time.

---

## Complexity Overview

| Function | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`zFunction(s)`** | $O(N)$ | $O(N)$ | Returns `z` of size $N = \Vert{}s\Vert{}$, where `z[i]` is the length of the longest prefix of `s` starting at `i` (`z[0] = n`). |
| **`search(text, pattern)`** | $O(N + M)$ | $O(N + M)$ | Returns a sorted `vector<int>` of all **0-based starting indices** where `pattern` (size $M$) occurs in `text` (size $N$). |

---

## Implementation

```cpp
vector<int> zFunction(const string &s) {
    int n = (int)s.size();
    if (n == 0) return {};
    vector<int> z(n, 0);
    z[0] = n; // convention: z[0] = n (or set to 0 if counting proper suffixes)

    int l = 0, r = 0;
    for (int i = 1; i < n; i++) {
        if (i < r) {
            z[i] = min(r - i, z[i - l]);
        }
        while (i + z[i] < n && s[z[i]] == s[i + z[i]]) {
            z[i]++;
        }
        if (i + z[i] > r) {
            l = i;
            r = i + z[i];
        }
    }
    return z;
}

vector<int> search(const string &text, const string &pattern) {
    if (pattern.empty()) return {};
    string combined = pattern + "$" + text;
    vector<int> z = zFunction(combined);
    int m = (int)pattern.size();
    vector<int> positions;

    for (int i = m + 1; i < (int)combined.size(); i++) {
        if (z[i] == m) {
            positions.push_back(i - m - 1);
        }
    }
    return positions;
}
```

---

## Usage

### 1. Compute Z-Array (`zFunction`)
`z[i]` gives the exact length of the longest substring starting at index `i` that matches a prefix of `s`:
```cpp
string s = "aabaaab";
vector<int> z = zFunction(s);
// z = {7, 1, 0, 2, 3, 1, 0}
```

### 2. Find All Occurrences of a Pattern in Text (`search`)
Returns all **0-based starting indices** (including overlapping matches):
```cpp
string text = "ababa";
string pat  = "aba";

vector<int> pos = search(text, pat);
// pos = {0, 2}
```

### 3. Common String Applications with `zFunction`
* **Smallest Period of String `s`:** The smallest $p$ ($1 \le p < n$) such that $n \bmod p == 0$ and `z[p] == n - p` (if none exists, the period is `n`).
* **All Borders (Prefixes that are also Suffixes):** Every index $i \ge 1$ where `i + z[i] == n` yields a border of length `z[i]`.

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste zFunction and search here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    string text = "ababacaba";
    string pattern = "aba";

    vector<int> matches = search(text, pattern);
    cout << "Matches at indices: ";
    for (int idx : matches) {
        cout << idx << " "; // 0 2 6
    }
    cout << "\n";

    // Find shortest period of "abcabcabc"
    string periodic = "abcabcabc";
    int n = (int)periodic.size();
    vector<int> z = zFunction(periodic);
    int period = n;
    for (int i = 1; i < n; i++) {
        if (n % i == 0 && z[i] == n - i) {
            period = i;
            break;
        }
    }
    cout << "Shortest period: " << period << "\n"; // 3

    return 0;
}
```

---

## Important Notes

1. **Delimiter Character `'$'` in `search`:** `pattern + "$" + text` assumes `'$'` does **not** appear inside `pattern` or `text`. If the input alphabet can contain arbitrary printable ASCII characters (including `'$'`), use a non-printable sentinel such as `'\0'` or `'\1'`.
2. **Empty String Guard:** Added `if (n == 0) return {};` in `zFunction` so calling `zFunction("")` does not access `z[0]` out of bounds.
3. **Convention for `z[0]`:** This template sets `z[0] = n` (since the entire string matches itself). Some problems define `z[0] = 0`—adjust that single line if a problem explicitly specifies `z[0] = 0`.
4. **Integer / Token Arrays:** `zFunction` works identically on `vector<int>` (using a sentinel like `INT_MIN` between `pattern` and `text`) when matching subarrays instead of strings.