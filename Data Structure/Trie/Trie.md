# Prefix Tree (Trie) with Autocomplete Usage Guide

A C++ pointer-based **Trie** (`Trie`) for lowercase English strings (`'a'`–`'z'`) that supports exact word counting, prefix counting, and **$O(\vert{}P\vert{})$ autocomplete queries** returning the highest-frequency word (with lexicographical tie-breaking) for any given prefix.

---

## Complexity Overview

| Operation | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`insert(s, freq)`** | $O(\Vert{}s\Vert{}^2)$ worst-case | $O(\Vert{}s\Vert{}^2)$ worst-case | $O(\Vert{}s\Vert{})$ pointer traversal, plus $O(\Vert{}s\Vert{})$ string copy/comparison when updating `bestWord`. |
| **`query(s)`** | $O(\Vert{}s\Vert{})$ | $O(1)$ | Returns `{bestWord, bestFreq}` among all inserted words having prefix `s`. |
| **`countWordsEqualTo(s)`** | $O(\Vert{}s\Vert{})$ | $O(1)$ | Returns the exact number of times `s` was inserted. |
| **`countWordsStartingWith(s)`** | $O(\Vert{}s\Vert{})$ | $O(1)$ | Returns the total count of inserted words that start with prefix `s`. |

*(Total trie memory across $N$ strings of total length $L$ is $O(L \cdot \Sigma + L^2)$ where $\Sigma = 26$, due to storing `bestWord` at each prefix node.)*

---

## Implementation

```cpp
using LL = long long;

struct Trie {
    struct Node {
        Node *ch[26];
        int endCount;
        int minDepth;
        int prefix;

        int bestFreq;
        string bestWord;

        Node() {
            memset(ch, 0, sizeof ch);
            endCount = 0;
            prefix = 0;
            minDepth = INT_MAX;

            bestFreq = 0;
            bestWord = "";
        }
    };

    Node *root = new Node();

    void insert(const string &s, int freq) {
        Node *cur = root;

        for (char c : s) {
            int idx = c - 'a';

            if (!cur->ch[idx])
                cur->ch[idx] = new Node();

            cur = cur->ch[idx];

            cur->prefix++;

            if (freq > cur->bestFreq ||
                (freq == cur->bestFreq && (cur->bestWord.empty() || s < cur->bestWord))) {
                cur->bestFreq = freq;
                cur->bestWord = s;
            }
        }

        cur->endCount++;
    }

    pair<string, int> query(const string &s) {
        Node *cur = root;

        for (char c : s) {
            int idx = c - 'a';

            if (!cur->ch[idx])
                return {"", 0};

            cur = cur->ch[idx];
        }

        return {cur->bestWord, cur->bestFreq};
    }

    LL countWordsEqualTo(const string &s) {
        Node *cur = root;
        for (char c : s) {
            int idx = c - 'a';
            if (!cur->ch[idx]) {
                return 0;
            }
            cur = cur->ch[idx];
        }
        return cur->endCount;
    }

    LL countWordsStartingWith(const string &s) {
        Node *cur = root;
        for (char c : s) {
            int idx = c - 'a';
            if (!cur->ch[idx])
                return 0;
            cur = cur->ch[idx];
        }
        return cur->prefix;
    }
};
```

---

## Usage

### 1. Insert Words with Frequencies (`insert`)
Pass each lowercase word along with its weight/frequency:
```cpp
Trie trie;
trie.insert("apple", 10);
trie.insert("app", 15);
trie.insert("apricot", 15);
trie.insert("apple", 10);
```

### 2. Autocomplete Best Match (`query`)
Retrieve the most frequent word starting with a given prefix (ties break toward the lexicographically smaller string):
```cpp
auto [word1, f1] = trie.query("ap");   // {"app", 15} ("app" < "apricot")
auto [word2, f2] = trie.query("appl"); // {"apple", 10}
auto [word3, f3] = trie.query("cat");  // {"", 0} (prefix not found)
```

### 3. Exact & Prefix Counts
```cpp
LL exact = trie.countWordsEqualTo("apple");        // 2
LL pref  = trie.countWordsStartingWith("ap");      // 4
```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

using LL = long long;

// Paste Trie struct here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    Trie trie;
    trie.insert("codeforces", 50);
    trie.insert("code", 80);
    trie.insert("coder", 80);
    trie.insert("contest", 30);

    auto [bestWord, bestFreq] = trie.query("cod");
    cout << bestWord << " (" << bestFreq << ")\n"; // code (80)

    cout << "Exact 'code': " << trie.countWordsEqualTo("code") << "\n";        // 1
    cout << "Prefix 'co': " << trie.countWordsStartingWith("co") << "\n";      // 4
    cout << "Prefix 'xyz': " << trie.countWordsStartingWith("xyz") << "\n";    // 0

    return 0;
}
```

---

## Important Notes

1. **Duplicate Insertions Do Not Accumulate `freq`:** Calling `insert("apple", 10)` twice increments `endCount` and `prefix` by `2`, while `bestFreq` compares the individual `freq` argument passed on each call rather than summing historical frequencies of `"apple"`. If a problem gives words one by one and asks for total occurrence frequency, pre-aggregate counts in a `map<string, int>` first, then insert each unique word once with its total count.
2. **Memory & Speed Optimization for Long Strings:** Storing `string bestWord` at every node takes $O(|s|^2)$ memory and copy time per insertion. If total input length $\sum |s| > 10^5$ with long single words, store `int bestId` (an index into a global `vector<string> words`) instead of copying `string` into every Trie node.
3. **Empty String Query (`query("")`):** Because `root->prefix` and `root->bestWord` are not updated inside the loop, `query("")` returns `{"", 0}` and `countWordsStartingWith("")` returns `0`.
4. **Character Set Restriction:** `int idx = c - 'a'` assumes strictly lowercase `'a'`–`'z'`. Adjust `ch[26]` and index mapping if uppercase letters or digits are included.