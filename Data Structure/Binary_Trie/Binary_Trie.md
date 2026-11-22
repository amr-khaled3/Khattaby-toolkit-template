# Binary Trie (Bitwise Trie) Usage Guide

A C++ pointer-based **Binary Trie** (`BinaryTrie`) operating on 32-bit integers (bits `31` down to `0`) for solving bitwise XOR optimization and counting queries in **$O(B)$** time per operation ($B = 32$).

---

## Complexity Overview

| Operation | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`insert(num)`** | $O(B)$ | $O(B)$ worst-case | Inserts a 32-bit integer and increments path edge/node counts. |
| **`remove(num, 31, root)`** | $O(B)$ | $O(B)$ recursion | Decrements path edge/node counts for an existing number. |
| **`maxXor(num)`** | $O(B)$ | $O(1)$ | Finds $\max_{y \in \text{Trie}} (\text{num} \oplus y)$. |
| **`minXor(num)`** | $O(B)$ | $O(1)$ | Finds $\min_{y \in \text{Trie}} (\text{num} \oplus y)$. |
| **`queryTrie(x, k)`** | $O(B)$ | $O(1)$ | Counts elements $y \in \text{Trie}$ satisfying $(x \oplus y) < k$. |

*(where $B = 32$ bits; total space is $O(N \cdot B)$ for $N$ inserted integers)*

---

## Implementation

```cpp
using LL = long long;

struct BinaryTrie {
    struct Node {
        Node *child[2];
        int freq[2];
        int cnt;
        Node() {
            child[0] = child[1] = nullptr;
            freq[0] = freq[1] = 0;
            cnt = 0;
        }
    };
    Node* root;
    BinaryTrie() {
        root = new Node();
    }

    void insert(int num) {
        Node* curr = root;
        curr->cnt++;               
        for (int i = 31; i >= 0; i--) {
            int bit = (num >> i) & 1;
            if (!curr->child[bit])
                curr->child[bit] = new Node();
            curr->freq[bit]++;
            curr = curr->child[bit];
            curr->cnt++;
        }
    }

    void remove(int num, int i, Node* curr) {
        if (i < 0 || !curr) return;

        int bit = (num >> i) & 1;
        Node* nxt = curr->child[bit];
        if (!nxt) return;

        curr->freq[bit]--;
        remove(num, i - 1, nxt);
        nxt->cnt--;
        if (curr == root) curr->cnt--;
    }

    LL maxXor(LL num) {
        Node* curr = root;
        LL res = 0;
        for (int i = 31; i >= 0; i--) {
            int bit = (num >> i) & 1;
            if (curr->child[1 - bit] && curr->freq[1 - bit] > 0) {
                res |= (1LL << i);  
                curr = curr->child[1 - bit];
            } else {
                curr = curr->child[bit];
            }
        }
        return res;
    }

    LL minXor(LL num) {
        Node* curr = root;
        LL xorNum = 0;
        for (int i = 31; i >= 0; i--) {
            int bit = (num >> i) & 1;
            if (curr->child[bit] && curr->freq[bit] > 0) {
                curr = curr->child[bit];
            } else {
                xorNum |= (1LL << i);
                curr = curr->child[1 - bit];
            }
        }
        return xorNum;
    }

    LL queryTrie(int x, int k) { // count of numbers y in the trie such that (x ^ y) < k
        Node* cur = root;
        LL res = 0;

        for (int bit = 31; bit >= 0; bit--) {
            if (!cur) break;

            int xb = (x >> bit) & 1;
            int kb = (k >> bit) & 1;

            if (kb == 1) {
                if (cur->child[xb])
                    res += (cur->child[xb]->cnt);

                cur = cur->child[xb ^ 1];
            } else {
                cur = cur->child[xb];
            }
        }

        return res;
    }
};
```

---

## Usage

### 1. Initialize & Insert Elements
Insert at least one number into the Trie before calling `maxXor` or `minXor`:
```cpp
BinaryTrie trie;
trie.insert(3);
trie.insert(10);
trie.insert(5);
```

### 2. Maximum & Minimum XOR Queries
```cpp
LL mx = trie.maxXor(6); // max(6 ^ 3, 6 ^ 10, 6 ^ 5) = 6 ^ 10 = 12
LL mn = trie.minXor(6); // min(6 ^ 3, 6 ^ 10, 6 ^ 5) = 6 ^ 5  = 3
```

### 3. Count Pairs with XOR Strictly Less Than $k$ (`queryTrie`)
```cpp
// Count y in trie such that (6 ^ y) < 8:
// 6 ^ 3 = 5 (< 8), 6 ^ 10 = 12 (>= 8), 6 ^ 5 = 3 (< 8) -> returns 2
LL cntLess = trie.queryTrie(6, 8);
```

### 4. Remove an Existing Element
Pass the value, starting bit `31`, and `trie.root`:
```cpp
trie.remove(10, 31, trie.root);
```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

using LL = long long;

// Paste BinaryTrie struct here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    BinaryTrie trie;
    vector<int> a = {3, 10, 5, 25, 2, 8};

    for (int x : a) {
        trie.insert(x);
    }

    cout << "Max XOR with 5: " << trie.maxXor(5) << "\n";        // 5 ^ 25 = 28
    cout << "Min XOR with 5: " << trie.minXor(5) << "\n";        // 5 ^ 5  = 0
    cout << "Count (5 ^ y) < 10: " << trie.queryTrie(5, 10) << "\n";

    trie.remove(25, 31, trie.root);
    cout << "Max XOR with 5 after removing 25: " << trie.maxXor(5) << "\n"; // 5 ^ 10 = 15

    return 0;
}
```

---

## Important Notes

1. **Bug Fixes Applied from Original Snippet:**
   * **Uncommented `freq[2]`:** `insert`, `remove`, `maxXor`, and `minXor` all read/write `curr->freq[bit]`, which was commented out in `Node`.
   * **64-Bit Shift in `minXor`:** Replaced `1 << i` with `1LL << i` to prevent undefined behavior when `i = 31`.
   * **`root->cnt` Sync in `remove`:** `insert` increments `root->cnt++` at the start, whereas the original `remove` only decremented `nxt->cnt--` for child nodes, leaving `root->cnt` inflated. Added `if (curr == root) curr->cnt--;` to keep `root->cnt` accurate.
2. **Non-Empty Trie Requirement:** Never call `maxXor` or `minXor` when the Trie is empty (`root->cnt == 0`), as `curr` will dereference a `nullptr`.
3. **Only Remove Present Elements:** Only call `remove(num, 31, trie.root)` if `num` is guaranteed to currently exist in the Trie (e.g., tracked with a `map` or `unordered_map`), otherwise `freq` and `cnt` can become negative.
4. **64-Bit Values (`long long`):** If numbers can reach $10^{18}$, change parameter types from `int` to `LL`, loop from `i = 60` (or `62`) down to `0`, and use `(num >> i) & 1LL`.