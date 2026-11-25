# Manacher's Algorithm Usage Guide

A linear-time C++ implementation of **Manacher's Algorithm** (`Manacher`) that computes the maximum palindrome radius centered at every index of a string (for both **odd-length** and **even-length** palindromes) in **$O(N)$** time, enabling **$O(1)$** substring palindrome checks `is_palindrome(l, r)` without hashing collisions.

---

## Complexity Overview

| Member / Operation | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`Manacher(s)`** | $O(N)$ | $O(N)$ | Computes `p[0]` (even-length radii) and `p[1]` (odd-length radii) across all centers in linear time. |
| **`is_palindrome(l, r)`** | $O(1)$ | $O(1)$ | Checks whether the **0-based inclusive** substring $s[l \dots r]$ is a palindrome. |
| **`p[1][i]`** | $O(1)$ | $O(N)$ | Half-length (radius) of the longest **odd** palindrome centered at `i`; actual length = $2 \cdot p[1][i] + 1$. |
| **`p[0][i]`** | $O(1)$ | $O(N)$ | Half-length (radius) of the longest **even** palindrome with right-center `i` (between `i - 1` and `i`); actual length = $2 \cdot p[0][i]$. |

---

## Implementation

```cpp
struct Manacher {
  vector<int> p[2];

  // p[1][i] = radius of longest odd palindrome centered at i (length = 2 * p[1][i] + 1)
  // p[0][i] = radius of longest even palindrome with right center i (length = 2 * p[0][i])
  Manacher(string s) {
    int n = (int)s.size();

    p[0].resize(n + 1);
    p[1].resize(n);

    // z = 0 -> even, z = 1 -> odd
    for (int z = 0; z < 2; z++) {
      for (int i = 0, l = 0, r = 0; i < n; i++) {
        int t = r - i + !z;

        if (i < r)
          p[z][i] = min(t, p[z][l + t]);

        int L = i - p[z][i];
        int R = i + p[z][i] - !z;

        while (L >= 1 && R + 1 < n &&
               s[L - 1] == s[R + 1]) {
          p[z][i]++;
          L--, R++;
        }

        if (R > r)
          l = L, r = R;
      }
    }
  }

  // Checks whether 0-based inclusive substring s[l..r] is a palindrome in O(1)
  bool is_palindrome(int l, int r) {
    int mid = (l + r + 1) / 2;
    int len = r - l + 1;
    return 2 * p[len % 2][mid] + len % 2 >= len;
  }
};
```

---

## Usage

### 1. Initialize
Pass your string `s` directly to the `Manacher` constructor:
```cpp
string s = "abbabba";
Manacher man(s);
```

### 2. Query Substring Palindromes in $O(1)$ (`is_palindrome`)
Uses **0-based inclusive indices $[l, r]$**:
```cpp
bool ok1 = man.is_palindrome(0, 6); // true  ("abbabba", length 7)
bool ok2 = man.is_palindrome(1, 4); // true  ("bbab"? No: s[1..4] = "bbab" -> false; s[0..3] = "abba" -> true)
bool ok3 = man.is_palindrome(0, 3); // true  ("abba", length 4)
```

### 3. Extract Longest Palindromic Substring or Count All Palindromes
* **Total number of palindromic substrings in `s`:**
  ```cpp
  long long totalPalindromes = 0;
  for (int i = 0; i < (int)s.size(); i++) {
      totalPalindromes += man.p[1][i] + 1; // Odd palindromes centered at i
      totalPalindromes += man.p[0][i];     // Even palindromes with right-center i
  }
  ```
* **Longest palindromic substring `(start, maxLen)`:**
  ```cpp
  int bestLen = 1, start = 0;
  for (int i = 0; i < (int)s.size(); i++) {
      int oddLen = 2 * man.p[1][i] + 1;
      if (oddLen > bestLen) {
          bestLen = oddLen;
          start = i - man.p[1][i];
      }
      int evenLen = 2 * man.p[0][i];
      if (evenLen > bestLen) {
          bestLen = evenLen;
          start = i - man.p[0][i];
      }
  }
  string longestPal = s.substr(start, bestLen);
  ```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste Manacher struct here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    string s = "babadab";
    int n = (int)s.size();

    Manacher man(s);

    // 1. O(1) substring queries
    cout << "s[0..4] ('babad'): " << (man.is_palindrome(0, 4) ? "YES" : "NO") << "\n"; // NO
    cout << "s[2..6] ('badab'): " << (man.is_palindrome(2, 6) ? "YES" : "NO") << "\n"; // YES

    // 2. Count all palindromic substrings in O(N)
    long long count = 0;
    for (int i = 0; i < n; i++) {
        count += man.p[1][i] + 1 + man.p[0][i];
    }
    cout << "Total palindromic substrings: " << count << "\n";

    return 0;
}
```

---

## Important Notes

1. **0-Based Inclusive Range $[l, r]$:** `is_palindrome(l, r)` expects $0 \le l \le r < \|s\|$.
2. **Center Convention (`p[0]` vs. `p[1]`):**
   * For **odd** length `len`: `mid = (l + r) / 2`, radius is `p[1][mid]`, spanning `[mid - p[1][mid], mid + p[1][mid]]`.
   * For **even** length `len`: `mid = (l + r + 1) / 2` (the **right** element of the central pair), radius is `p[0][mid]`, spanning `[mid - p[0][mid], mid + p[0][mid] - 1]`. Note that `p[0][0]` is always `0` because there is no character to the left of index `0`.
3. **Zero Collision Risk:** Unlike Rolling Hash, Manacher's algorithm is 100% deterministic with zero hash-collision probability and $O(N)$ worst-case time, making it ideal when answering up to $10^6$ palindrome queries or finding longest palindromic extensions around every center.