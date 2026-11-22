# Two-Heap Running Median & Distance Sum Usage Guide

A C++ dual-heap data structure (`MedianFinder`) that maintains a dynamic stream of integers split into a lower half (`left`, max-heap) and an upper half (`right`, min-heap), supporting **$O(\log N)$** insertions and **$O(1)$** queries for the **running median**, half-sums, and half-counts—ideal for minimizing $\sum \vert{}a_i - x\vert{}$ in **$O(1)$** per query.

---

## Complexity Overview

| Operation | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`add(x)`** | $O(\log N)$ | $O(1)$ amortized | Inserts `x` into the appropriate heap and rebalances sizes. |
| **`balance()`** | $O(\log N)$ | $O(1)$ | Enforces $\Vert{}left\Vert{} = \Vert{}right\Vert{}$ or $\Vert{}left\Vert{} = \Vert{}right\Vert{} + 1$. |
| **`median()`** | $O(1)$ | $O(1)$ | Returns the exact median as `double` (lower median is always `left.top()`). |
| **`leftSum()` / `rightSum()`** | $O(1)$ | $O(1)$ | Returns the `long long` sum of elements in the lower or upper half. |
| **`leftCount()` / `rightCount()`** | $O(1)$ | $O(1)$ | Returns the number of elements in the lower or upper half. |

*(Total space complexity is $O(N)$ across $N$ inserted elements.)*

---

## Implementation

```cpp
struct MedianFinder {

    priority_queue<int> left;
    priority_queue<int, vector<int>, greater<int>> right;

    long long sumLeft = 0, sumRight = 0;

    void add(int x) {

        if (left.empty() || x <= left.top()) {
            left.push(x);
            sumLeft += x;
        } else {
            right.push(x);
            sumRight += x;
        }

        balance();
    }

    void balance() {

        while (left.size() > right.size() + 1) {
            int x = left.top();
            left.pop();
            sumLeft -= x;

            right.push(x);
            sumRight += x;
        }

        while (right.size() > left.size()) {
            int x = right.top();
            right.pop();
            sumRight -= x;

            left.push(x);
            sumLeft += x;
        }
    }

    double median() {
        if (left.size() == right.size())
            return (left.top() + right.top()) / 2.0;
        return left.top();
    }

    long long leftSum() const { return sumLeft; }
    long long rightSum() const { return sumRight; }

    int leftCount() const { return left.size(); }
    int rightCount() const { return right.size(); }
};
```

---

## Usage

### 1. Add Elements & Query Running Median
```cpp
MedianFinder mf;
mf.add(5);
mf.add(15);
mf.add(1);
mf.add(3);

// Sorted elements: [1, 3 | 5, 15]
double med = mf.median();      // (3 + 5) / 2.0 = 4.0
int lowerMed = mf.left.top();  // 3 (always the integer/lower median)
```

### 2. Compute Minimum Cost to Make All Elements Equal ($\sum \vert{}a_i - m\vert{}$)
Because `left` holds all elements $\le m$ and `right` holds all elements $\ge m$ (where $m = \text{left.top()}$), the minimum total absolute deviation is computed in **$O(1)$**:
```cpp
long long m = mf.left.top();
long long minCost = (m * mf.leftCount() - mf.leftSum())
                  + (mf.rightSum() - m * mf.rightCount());
```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste MedianFinder struct here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    vector<int> stream = {2, 8, 4, 10, 6};
    MedianFinder mf;

    for (int x : stream) {
        mf.add(x);
        long long m = mf.left.top();
        long long cost = (m * mf.leftCount() - mf.leftSum())
                       + (mf.rightSum() - m * mf.rightCount());

        cout << "Median: " << fixed << setprecision(1) << mf.median()
             << " | Min L1 Cost: " << cost << "\n";
    }

    return 0;
}
```

---

## Important Notes

1. **Integer Overflow in `median()`:** When `left.size() == right.size()` and element values exceed $10^9$, `left.top() + right.top()` overflows 32-bit `int` before dividing by `2.0`. Cast to `long long` or `double` first: `(1LL * left.top() + right.top()) / 2.0`.
2. **Non-Empty Requirement:** Never call `median()` or access `left.top()` before calling `add(x)` at least once, as `left` will be empty.
3. **64-Bit Elements (`long long`):** If stream values exceed $2 \times 10^9$, change `int` to `long long` in both `priority_queue` declarations, `add(long long x)`, and `balance()`.
4. **No Arbitrary Deletion:** Standard `std::priority_queue` only supports inserting elements, not removing arbitrary values (e.g., for sliding-window median problems). For sliding windows, use two `multiset<long long>` containers or lazy deletion with a frequency map.