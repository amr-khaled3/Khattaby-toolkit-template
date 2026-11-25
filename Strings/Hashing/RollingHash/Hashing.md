# Static Double Rolling Hash Usage Guide

A C++ polynomial rolling hash struct (`RollingHash`) for static strings, using double hashing (`MOD1 = 1e9 + 7`, `MOD2 = 1e9 + 9`) with precomputed powers (`pw1`, `pw2`) and modular inverses (`invPw1`, `invPw2`) to answer any **0-based inclusive** substring hash query $s[l \dots r]$ in **$O(1)$** time after **$O(N)$** preprocessing.

---

## Complexity Overview

| Operation | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`RollingHash(maxLen)`** | $O(\text{maxLen})$ | $O(\text{maxLen})$ | Precomputes base powers and modular inverses up to `maxLen` once. |
| **`build(s)`** | $O(\Vert{}s\Vert{})$ | $O(\Vert{}s\Vert{})$ | Computes prefix hashes `pref1` and `pref2` for string `s` ($\Vert{}s\Vert{} \le \text{maxLen}$). |
| **`substringHash(l, r)`** | $O(1)$ | $O(1)$ | Returns the normalized double hash `{h1, h2}` for the **0-based inclusive** range $[l, r]$. |

---

## Implementation

```cpp
struct RollingHash {

    using ll = long long;

    static constexpr ll MOD1  = 1'000'000'007LL, BASE1 = 31LL;
    static constexpr ll MOD2  = 1'000'000'009LL, BASE2 = 53LL;

    using Hash = pair<ll, ll>;

    vector<ll> pw1, pw2, invPw1, invPw2;
    vector<ll> pref1, pref2;

    static ll fastPow(ll base, ll exp, ll mod) {
        ll res = 1;
        for (base %= mod; exp > 0; exp >>= 1) {
            if (exp & 1) res = res * base % mod;
            base = base * base % mod;
        }
        return res;
    }
    static ll inv(ll x, ll mod) { return fastPow(x, mod - 2, mod); }

    static int charVal(char c) { return c - 'a' + 1; }  

    explicit RollingHash(int maxLen) {
        pw1.resize(maxLen + 1);    pw2.resize(maxLen + 1);
        invPw1.resize(maxLen + 1); invPw2.resize(maxLen + 1);

        pw1[0] = pw2[0] = invPw1[0] = invPw2[0] = 1;

        ll ib1 = inv(BASE1, MOD1), ib2 = inv(BASE2, MOD2);

        for (int i = 1; i <= maxLen; i++) {
            pw1[i]    = pw1[i-1]    * BASE1 % MOD1;
            pw2[i]    = pw2[i-1]    * BASE2 % MOD2;
            invPw1[i] = invPw1[i-1] * ib1   % MOD1;
            invPw2[i] = invPw2[i-1] * ib2   % MOD2;
        }
    }

    void build(const string& s) {
        int n = (int)s.size();
        pref1.assign(n, 0);
        pref2.assign(n, 0);

        for (int i = 0; i < n; i++) {
            ll v = charVal(s[i]);
            pref1[i] = v * pw1[i] % MOD1;
            pref2[i] = v * pw2[i] % MOD2;
            if (i) {
                pref1[i] = (pref1[i] + pref1[i-1]) % MOD1;
                pref2[i] = (pref2[i] + pref2[i-1]) % MOD2;
            }
        }
    }

    Hash substringHash(int l, int r) const {
        ll h1 = pref1[r], h2 = pref2[r];
        if (l > 0) {
            h1 = (h1 - pref1[l-1] + MOD1) % MOD1;
            h2 = (h2 - pref2[l-1] + MOD2) % MOD2;
        }
        h1 = h1 * invPw1[l] % MOD1;
        h2 = h2 * invPw2[l] % MOD2;
        return {h1, h2};
    }
};
```

---

## Usage

### 1. Initialize & Build
Construct `RollingHash` with the maximum string length (can be reused across multiple `build(s)` calls), then build on your string:
```cpp
string s = "abracadabra";
RollingHash rh(s.size());
rh.build(s);
```

### 2. Substring Equality Queries (`substringHash(l, r)`)
Uses **0-based inclusive intervals $[l, r]$** (both `l` and `r` are included):
```cpp
// Compare s[0..3] ("abra") and s[7..10] ("abra")
bool same = (rh.substringHash(0, 3) == rh.substringHash(7, 10)); // true
```

### 3. Longest Common Prefix (LCP) of Two Substrings in $O(\log N)$
Binary search the maximum length `len` where the prefixes match:
```cpp
auto lcp = [&](int i, int j) {
    int n = (int)s.size();
    int lo = 1, hi = n - max(i, j), ans = 0;
    while (lo <= hi) {
        int mid = (lo + hi) / 2;
        if (rh.substringHash(i, i + mid - 1) == rh.substringHash(j, j + mid - 1)) {
            ans = mid;
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    return ans;
};
```

### 4. Palindrome Check in $O(1)$
Build a second `RollingHash` on the reversed string `revS` (`revIdx = n - 1 - idx`):
```cpp
string revS(s.rbegin(), s.rend());
RollingHash fwd(s.size()), bwd(s.size());
fwd.build(s);
bwd.build(revS);

int n = (int)s.size();
auto isPalindrome = [&](int l, int r) {
    return fwd.substringHash(l, r) == bwd.substringHash(n - 1 - r, n - 1 - l);
};
```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste RollingHash struct here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    string s = "abacaba";
    int n = (int)s.size();

    RollingHash rh(n);
    rh.build(s);

    // Check if prefix of length 3 ("aba") equals suffix of length 3 ("aba")
    if (rh.substringHash(0, 2) == rh.substringHash(4, 6)) {
        cout << "Matching border of length 3!\n";
    }

    // Count distinct substrings of length 3
    set<RollingHash::Hash> distinctSubstrings;
    for (int i = 0; i + 3 <= n; i++) {
        distinctSubstrings.insert(rh.substringHash(i, i + 2));
    }
    cout << "Distinct substrings of length 3: " << distinctSubstrings.size() << "\n"; // 4

    return 0;
}
```

---

## Important Notes

1. **Inclusive Range $[l, r]$ vs. `HashSegTree` Half-Open $[l, r)$:** Unlike `HashSegTree::substringHash(l, r)` which is half-open $[l, r)$, `RollingHash::substringHash(l, r)` is **0-based inclusive** ($0 \le l \le r < \|s\|$). Always ensure $l \le r$ before calling.
2. **Reuse Across Test Cases:** In multi-testcase problems (`TEST`), instantiate `RollingHash rh(MAXN);` **once** outside the testcase loop and only call `rh.build(s)` inside each testcase to avoid recomputing powers and modular inverses repeatedly.
3. **Character Set Restriction:** `charVal(c)` computes `c - 'a' + 1`, assuming strictly lowercase `'a'..'z'`. For uppercase letters, digits, or general ASCII strings, change `charVal(c)` to `(unsigned char)c + 1` and increase `BASE1` / `BASE2` above `256` (e.g., `311` and `313`).
4. **Static Strings Only:** `RollingHash` does not support character updates after `build(s)`. If characters change dynamically, use `HashSegTree`.