# Suffix Automaton (`SuffixAutomaton`) Usage Guide

A compact, linear-size C++ implementation of a **Suffix Automaton (SAM)** (`SuffixAutomaton`) for lowercase English strings (`'a'`–`'z'`). Constructs the minimal deterministic finite automaton recognizing all substrings of a string `s` in **$O(N)$** time and **$O(N \cdot \Sigma)$** space (at most $2N - 1$ states and $3N - 4$ transitions), supporting substring existence checks, substring occurrence counting (`preCount`), distinct substring counting, and Longest Common Substring (LCS) queries.

---

## Complexity Overview

| Member / Operation | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`SuffixAutomaton(s)`** | $O(N)$ | $O(N \cdot \Sigma)$ | Builds the SAM online character-by-character ($\Sigma = 26$, at most $2N - 1$ states). |
| **`insert(ch)`** | $O(1)$ amortized | $O(\Sigma)$ per state | Appends character `ch` (`'a'`–`'z'`) and updates suffix links (`lnk`) and transitions (`nxt`). |
| **`move(v, c)`** | $O(1)$ | $O(1)$ | Advances from state `v` along character `c`, returning the next state ID or `-1` if no transition exists. |
| **`preCount()`** | $O(N \log N)$ | $O(N)$ | Propagates end-position counts (`cnt`) up the suffix-link tree in decreasing order of `len`. |
| **`t[v].len` / `t[v].lnk`** | $O(1)$ | $O(N)$ | Longest substring length accepted by state `v` / suffix link parent in the link tree. |

---

## Implementation

```cpp
struct SuffixAutomaton
{
    static const int A = 26;
    struct State
    {
        int len = 0, lnk = -1, cnt = 0;
        array<int, A> nxt;
        State() { nxt.fill(-1); }
    };
    vector<State> t{ { } };
    int lst = 0;

    SuffixAutomaton() = default;
    SuffixAutomaton(const string &s)
    { for(char ch: s) insert(ch); }

    void insert(int ch)
    {
        int c = ch - 'a', me = (int)t.size(), p = lst;
        t.push_back({ });
        t[me].len = t[p].len + 1;
        t[me].cnt = 1;
        t[me].lnk = 0;
        lst = me;
        while(~p && t[p].nxt[c] == -1)
            t[p].nxt[c] = me, p = t[p].lnk;
        if(p == -1) return;
        int q = t[p].nxt[c];
        if(t[q].len == t[p].len + 1)
        { t[me].lnk = q; return; }
        int clone = (int)t.size(); t.push_back(t[q]);
        t[clone].len = t[p].len + 1, t[clone].cnt = 0;
        while (~p && t[p].nxt[c] == q)
            t[p].nxt[c] = clone, p = t[p].lnk;
        t[q].lnk = t[me].lnk = clone;
    }

    int move(int v, char c) const
    { return ~v ? t[v].nxt[c - 'a'] : -1; }

    void preCount()
    {
        vector<array<int, 2>> v;
        v.reserve(t.size());
        for(int i = 0; i < (int)t.size(); ++i)
            v.push_back({ t[i].len, i });
        sort(v.rbegin(), v.rend());
        for(auto &[_, i]: v)
            if(i) t[t[i].lnk].cnt += t[i].cnt;
    }
};
```

---

## Usage

### 1. Build SAM & Propagate Occurrence Counts
Construct the automaton from string `s` (state `0` is the root) and call `preCount()` once if you need substring frequencies:
```cpp
string s = "abacaba";
SuffixAutomaton sam(s);
sam.preCount();
```

### 2. Check Substring Existence & Count Occurrences in $O(\vert{}P\vert{})$
Walk from root state `v = 0` using `sam.move(v, c)`:
```cpp
auto countOccurrences = [&](const string &pat) -> int {
    int v = 0;
    for (char c : pat) {
        v = sam.move(v, c);
        if (v == -1) return 0; // Substring does not exist
    }
    return sam.t[v].cnt; // Exact number of times `pat` appears in `s`
};

int freqAba = countOccurrences("aba"); // 2
int freqXyz = countOccurrences("xyz"); // 0
```

### 3. Count Total Number of Distinct Substrings in $O(N)$
Each state $v \ge 1$ represents all substrings ending at its equivalence class whose lengths lie in $[t[t[v].\text{lnk}].\text{len} + 1,\; t[v].\text{len}]$:
```cpp
long long distinctSubstrings = 0;
for (int v = 1; v < (int)sam.t.size(); v++) {
    distinctSubstrings += sam.t[v].len - sam.t[sam.t[v].lnk].len;
}
```

### 4. Longest Common Substring (LCS) of Two Strings $S$ and $T$ in $O(\vert{}T\vert{})$
Build `sam(S)`, then stream characters of `T` while tracking the current state `v` and matched length `curLen`:
```cpp
int v = 0, curLen = 0, bestLen = 0;
for (char c : T) {
    while (v != 0 && sam.move(v, c) == -1) {
        v = sam.t[v].lnk;
        curLen = sam.t[v].len;
    }
    if (sam.move(v, c) != -1) {
        v = sam.move(v, c);
        curLen++;
    }
    bestLen = max(bestLen, curLen);
}
```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste SuffixAutomaton struct here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    string s = "abracadabra";
    SuffixAutomaton sam(s);
    sam.preCount();

    // 1. Distinct substrings count
    long long distinct = 0;
    for (int v = 1; v < (int)sam.t.size(); v++) {
        distinct += sam.t[v].len - sam.t[sam.t[v].lnk].len;
    }
    cout << "Distinct substrings: " << distinct << "\n"; // 54

    // 2. Pattern occurrence queries
    for (string q : {"abra", "cad", "a", "xyz"}) {
        int v = 0;
        for (char c : q) v = sam.move(v, c);
        cout << q << " -> " << (v == -1 ? 0 : sam.t[v].cnt) << "\n";
    }

    return 0;
}
```

---

## Important Notes

1. **Const-Correctness Improvements Applied:** Changed `SuffixAutomaton(string &s)` to `SuffixAutomaton(const string &s)` and `move(int v, char &c)` to `move(int v, char c) const` so string literals (e.g., `SuffixAutomaton("abc")`) and character literals (e.g., `sam.move(v, 'a')`) compile without non-const lvalue reference errors.
2. **Call `preCount()` Only Once:** `preCount()` accumulates `t[t[i].lnk].cnt += t[i].cnt` in place. Never call `preCount()` more than once on the same `SuffixAutomaton` instance, and never call `insert()` after calling `preCount()`.
3. **Memory Footprint for Large $N$:** A SAM creates up to $2N - 1$ states, each storing `array<int, 26>` ($\approx 116$ bytes/state $\implies \approx 23\text{ MB}$ for $N = 10^5$, or $\approx 230\text{ MB}$ for $N = 10^6$). Call `t.reserve(2 * s.size())` inside the constructor when $N \ge 5 \times 10^5$ to prevent vector reallocation memory spikes.
4. **Lowercase Alphabet Assumption:** `c = ch - 'a'` assumes lowercase `'a'`–`'z'`. For arbitrary ASCII or large alphabets, increase `A` (or replace `array<int, A>` with `map<int, int>` at an $O(\log \Sigma)$ factor).