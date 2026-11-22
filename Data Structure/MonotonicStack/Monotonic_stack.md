# Monotonic Stack Usage Guide

A C++ utility struct (`MonotonicStack`) that computes the **0-based indices** of the nearest strictly greater or strictly smaller elements to the left and right for every element in an array in linear **$O(N)$** time.

---

## Complexity Overview

| Operation | Time Complexity | Space Complexity | Fallback Value (If None Exists) |
| :--- | :--- | :--- | :--- |
| **`MonotonicStack(arr)`** | $O(N)$ | $O(N)$ | — |
| **`nextGreaterIndex()`** | $O(N)$ | $O(N)$ | `n` |
| **`nextSmallerIndex()`** | $O(N)$ | $O(N)$ | `n` |
| **`prevGreaterIndex()`** | $O(N)$ | $O(N)$ | `-1` |
| **`prevSmallerIndex()`** | $O(N)$ | $O(N)$ | `-1` |

---

## Implementation

```cpp
struct MonotonicStack {
    vector<int> a;
    int n;

    MonotonicStack(const vector<int>& arr) {
        a = arr;
        n = a.size();
    }

    // -----------------------------
    // Index-based functions 
    // -----------------------------

    // Next Greater Element Index (Right)
    vector<int> nextGreaterIndex() {
        vector<int> res(n, n); // n = no element
        stack<int> st;
        for (int i = n - 1; i >= 0; i--) {
            while (!st.empty() && a[st.top()] <= a[i]) st.pop();
            if (!st.empty()) res[i] = st.top();
            st.push(i);
        }
        return res;
    }

    // Next Smaller Element Index (Right)
    vector<int> nextSmallerIndex() {
        vector<int> res(n, n);
        stack<int> st;
        for (int i = n - 1; i >= 0; i--) {
            while (!st.empty() && a[st.top()] >= a[i]) st.pop();
            if (!st.empty()) res[i] = st.top();
            st.push(i);
        }
        return res;
    }

    // Previous Greater Element Index (Left)
    vector<int> prevGreaterIndex() {
        vector<int> res(n, -1);
        stack<int> st;
        for (int i = 0; i < n; i++) {
            while (!st.empty() && a[st.top()] <= a[i]) st.pop();
            if (!st.empty()) res[i] = st.top();
            st.push(i);
        }
        return res;
    }

    // Previous Smaller Element Index (Left)
    vector<int> prevSmallerIndex() {
        vector<int> res(n, -1);
        stack<int> st;
        for (int i = 0; i < n; i++) {
            while (!st.empty() && a[st.top()] >= a[i]) st.pop();
            if (!st.empty()) res[i] = st.top();
            st.push(i);
        }
        return res;
    }
};
```

---

## Usage

### 1. Initialization
Pass your `vector<int>` into the `MonotonicStack` constructor:
```cpp
vector<int> arr = {2, 1, 5, 6, 2, 3};
MonotonicStack ms(arr);
```

### 2. Query Nearest Element Indices
Each method returns a `vector<int>` of size `n` storing the **0-based index** of the target neighbor:
```cpp
vector<int> nge = ms.nextGreaterIndex();   // {2, 2, 3, 6, 5, 6}
vector<int> nse = ms.nextSmallerIndex();   // {1, 6, 4, 4, 6, 6}
vector<int> pge = ms.prevGreaterIndex();   // {-1, 0, -1, -1, 3, 3}
vector<int> pse = ms.prevSmallerIndex();   // {-1, -1, 1, 2, 1, 4}
```

### 3. Retrieving Values Safely
Always check against the sentinel value (`n` for right queries, `-1` for left queries) before indexing back into `arr`:
```cpp
int idx = nge[0];
int val = (idx == (int)arr.size()) ? -1 : arr[idx]; // 5
```

---

## Complete Example (Largest Rectangle in Histogram)

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste MonotonicStack struct here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    vector<int> heights = {2, 1, 5, 6, 2, 3};
    int n = heights.size();

    MonotonicStack ms(heights);
    vector<int> left = ms.prevSmallerIndex();
    vector<int> right = ms.nextSmallerIndex();

    long long maxArea = 0;
    for (int i = 0; i < n; i++) {
        long long width = right[i] - left[i] - 1;
        maxArea = max(maxArea, 1LL * heights[i] * width);
    }

    cout << maxArea << "\n"; // Outputs 10

    return 0;
}
```

---

## Important Notes

1. **Sentinel Boundaries (`-1` and `n`):** Using `-1` for missing left elements and `n` for missing right elements makes subarray boundary math clean without extra `if` checks—for example, `right[i] - left[i] - 1` directly gives the maximum contiguous subarray length where `a[i]` is the strict minimum.
2. **Strict vs. Non-Strict Comparisons:** All four methods currently find **strictly** greater (`>`) or **strictly** smaller (`<`) elements because the `while` loops pop equal elements (`<=` and `>=`). If you need **greater-than-or-equal (`>=`)** or **less-than-or-equal (`<=`)**, change `<=` to `<` and `>=` to `>` inside the `while` condition.
3. **Handling Duplicate Elements in Subarray Counting:** When counting subarrays where `a[i]` is the minimum/maximum and duplicates exist, make **one** side strict (`<=` / `>=`) and the **other** side non-strict (`<` / `>`) to avoid double-counting subarrays containing duplicate extrema.
4. **64-Bit Values (`long long`):** If input values exceed $2 \times 10^9$, change `vector<int> a` and the constructor parameter `const vector<int>& arr` to `long long` (the returned index vectors remain `vector<int>`).