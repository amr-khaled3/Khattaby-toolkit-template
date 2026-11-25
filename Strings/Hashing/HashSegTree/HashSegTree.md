# Position-Weighted Double Hash Segment Tree Usage Guide

A C++ Segment Tree (`HashSegTree`) supporting **$O(\log N)$** point character updates and **$O(\log N)$** substring polynomial rolling hash queries over dynamic strings. Uses **position-weighted leaves** ($s[i] \cdot B^i \bmod M$) with double hashing (`MOD1 = 1e9 + 7`, `MOD2 = 1e9 + 9`) so node merging is simple modular addition and substring normalization requires only multiplying by $B^{-l} \bmod M$.

---

## Complexity Overview

| Operation | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`HashSegTree(n)`** | $O(N)$ | $O(N)$ | Precomputes base powers `pw1/pw2`, modular inverses `invPw1/invPw2`, and allocates $2 \cdot 2^{\lceil \log_2 N \rceil}$ nodes. |
| **`init(s)`** | $O(N)$ | $O(\log N)$ stack | Builds the segment tree from string `s` in linear time. |
| **`set(idx, val)`** | $O(\log N)$ | $O(\log N)$ stack | Updates character `s[idx]` to `val` (0-based index). |
| **`substringHash(l, r)`** | $O(\log N)$ | $O(\log N)$ stack | Returns the normalized double hash `{h1, h2}` of half-open substring $s[l \dots r - 1]$. |

---

## Implementation

```cpp
#include <bits/stdc++.h>
using namespace std;

struct HashSegTree {
    using ll = long long;
    using Hash = pair<ll, ll>;

    static constexpr ll MOD1 = 1'000'000'007LL, BASE1 = 31LL;
    static constexpr ll MOD2 = 1'000'000'009LL, BASE2 = 53LL;

    struct Node {
        ll h1, h2;
        Node() : h1(0), h2(0) {}
        Node(ll _h1, ll _h2) : h1(_h1), h2(_h2) {}
    };

    int tree_size;
    vector<Node> SegData;
    vector<ll> pw1, pw2, invPw1, invPw2;

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

    explicit HashSegTree(int n) {
        tree_size = 1;
        while (tree_size < n) tree_size *= 2;
        SegData.assign(2 * tree_size, Node());

        pw1.resize(n + 1);    pw2.resize(n + 1);
        invPw1.resize(n + 1); invPw2.resize(n + 1);
        pw1[0] = pw2[0] = invPw1[0] = invPw2[0] = 1;

        ll ib1 = inv(BASE1, MOD1), ib2 = inv(BASE2, MOD2);
        for (int i = 1; i <= n; i++) {
            pw1[i]    = pw1[i - 1]    * BASE1 % MOD1;
            pw2[i]    = pw2[i - 1]    * BASE2 % MOD2;
            invPw1[i] = invPw1[i - 1] * ib1   % MOD1;
            invPw2[i] = invPw2[i - 1] * ib2   % MOD2;
        }
    }

    Node merge(const Node &lf, const Node &ri) const {
        return Node((lf.h1 + ri.h1) % MOD1, (lf.h2 + ri.h2) % MOD2);
    }

    Node makeLeaf(int idx, char c) const {
        ll v = charVal(c);
        return Node(v * pw1[idx] % MOD1, v * pw2[idx] % MOD2);
    }

    void init(const string &s, int ni, int lx, int rx) {
        if (rx - lx == 1) {
            if (lx < (int)s.size())
                SegData[ni] = makeLeaf(lx, s[lx]);
            return;
        }
        int mid = (rx + lx) / 2;
        init(s, 2 * ni + 1, lx, mid);
        init(s, 2 * ni + 2, mid, rx);
        SegData[ni] = merge(SegData[2 * ni + 1], SegData[2 * ni + 2]);
    }

    void init(const string &s) {
        init(s, 0, 0, tree_size);
    }

    void set(int idx, char val, int ni, int lx, int rx) {
        if (rx - lx == 1) {
            SegData[ni] = makeLeaf(idx, val);
            return;
        }
        int mid = (rx + lx) / 2;
        if (idx < mid) set(idx, val, 2 * ni + 1, lx, mid);
        else           set(idx, val, 2 * ni + 2, mid, rx);
        SegData[ni] = merge(SegData[2 * ni + 1], SegData[2 * ni + 2]);
    }

    void set(int idx, char val) {
        set(idx, val, 0, 0, tree_size);
    }

    Node get(int l, int r, int ni, int lx, int rx) const {
        if (lx >= l && rx <= r) return SegData[ni];
        if (lx >= r || rx <= l) return Node();

        int mid = (rx + lx) / 2;
        Node lf = get(l, r, 2 * ni + 1, lx, mid);
        Node ri = get(l, r, 2 * ni + 2, mid, rx);
        return merge(lf, ri);
    }

    Hash substringHash(int l, int r) const {
        Node res = get(l, r, 0, 0, tree_size);
        ll h1 = res.h1 * invPw1[l] % MOD1;
        ll h2 = res.h2 * invPw2[l] % MOD2;
        return {h1, h2};
    }
};
```

---

## Usage

### 1. Initialize & Build
Pass the string length `n` to the constructor, then call `init(s)`:
```cpp
string s = "abacaba";
int n = s.size();

HashSegTree hst(n);
hst.init(s);
```

### 2. Substring Equality Queries (`substringHash(l, r)`)
Uses **0-based half-open intervals $[l, r)$** (`r` is excluded, so length is `r - l`):
```cpp
// Compare s[0..2] ("aba") and s[4..6] ("aba")
bool equal = (hst.substringHash(0, 3) == hst.substringHash(4, 7)); // true
```

### 3. Point Character Updates (`set(idx, val)`)
Modify any 0-based index `idx` and re-query in $O(\log N)$:
```cpp
hst.set(2, 'z'); // s becomes "abzcaba"
bool stillEqual = (hst.substringHash(0, 3) == hst.substringHash(4, 7)); // false
```

### 4. Dynamic Palindrome Queries (Forward + Reversed Tree)
Maintain a second `HashSegTree` built on the reversed string `revS` (`revIdx = n - 1 - idx`):
```cpp
string revS(s.rbegin(), s.rend());
HashSegTree fwd(n), bwd(n);
fwd.init(s);
bwd.init(revS);

// Update index idx to char c in both trees:
auto updateBoth = [&](int idx, char c) {
    fwd.set(idx, c);
    bwd.set(n - 1 - idx, c);
};

// Check if inclusive range [L, R] (0-based) is a palindrome:
auto isPalindrome = [&](int L, int R) {
    return fwd.substringHash(L, R + 1) == bwd.substringHash(n - 1 - R, n - L);
};
```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste HashSegTree struct here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    string s = "racecar";
    int n = s.size();

    string revS(s.rbegin(), s.rend());
    HashSegTree fwd(n), bwd(n);
    fwd.init(s);
    bwd.init(revS);

    // Check if s[0..6] ("racecar") is a palindrome -> YES
    cout << (fwd.substringHash(0, 7) == bwd.substringHash(0, 7) ? "YES\n" : "NO\n");

    // Update index 1 ('a' -> 'x'): s becomes "rxcecar"
    fwd.set(1, 'x');
    bwd.set(n - 1 - 1, 'x');

    // Check s[0..6] again -> NO, while s[2..4] ("cec") -> YES
    cout << (fwd.substringHash(0, 7) == bwd.substringHash(0, 7) ? "YES\n" : "NO\n");
    cout << (fwd.substringHash(2, 5) == bwd.substringHash(n - 1 - 4, n - 2) ? "YES\n" : "NO\n");

    return 0;
}
```

---

## Important Notes

1. **Half-Open Interval $[l, r)$:** `substringHash(l, r)` covers indices $l, l + 1, \dots, r - 1$. For an inclusive 0-based query $[L, R]$, call `substringHash(L, R + 1)`.
2. **Lowercase Alphabet Assumption:** `charVal(c)` computes `c - 'a' + 1`, which maps `'a'..'z'` to `1..26`. If the string contains uppercase letters, digits, or arbitrary ASCII characters, change `charVal(c)` to `(unsigned char)c + 1` and increase `BASE1` / `BASE2` above `256` (e.g., `311` and `313`).
3. **Why Non-Zero Mapping Matters:** Mapping characters to $\ge 1$ (`+ 1`) prevents leading zeros from colliding (e.g., distinguishing `"a"` from `"aa"`).
4. **Anti-Hash Safety:** Using two independent prime moduli (`1e9 + 7` and `1e9 + 9`) with distinct bases (`31` and `53`) makes collision probability negligible ($\approx 10^{-18}$). For extra safety against hacked tests on Codeforces, randomize `BASE1` and `BASE2` at runtime using `mt19937_64`.