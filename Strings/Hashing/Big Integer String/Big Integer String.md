# Big Integer String Multiplication & Division Usage Guide

A C++ arbitrary-precision arithmetic utility for multiplying and dividing non-negative integers represented as base-10 strings (`std::string`)—handling arbitrarily large numbers beyond 64-bit `long long` and `__int128` limits using grade-school digit-by-digit multiplication and long division.

---

## Complexity Overview

| Function | Time Complexity | Space Complexity | Notes |
| :--- | :--- | :--- | :--- |
| **`multiplyStrings(a, b)`** | $O(N \cdot M)$ | $O(N + M)$ | Computes exact decimal product $a \times b$ for strings of lengths $N = \Vert{}a\Vert{}$ and $M = \Vert{}b\Vert{}$. |
| **`divideStrings(num, divisor)`** | $O(N \cdot M)$ | $O(N + M)$ | Computes exact integer quotient $\lfloor \text{num} / \text{divisor} \rfloor$ via long division (`"Infinity"` if `divisor == "0"`). |

---

## Implementation

```cpp
string multiplyStrings(const string &a, const string &b) {
    if (a == "0" || b == "0") return "0";

    int n = (int)a.size(), m = (int)b.size();
    vector<int> res(n + m, 0);

    for (int i = n - 1; i >= 0; i--) {
        for (int j = m - 1; j >= 0; j--) {
            int mul = (a[i] - '0') * (b[j] - '0');

            int sum = mul + res[i + j + 1];

            res[i + j + 1] = sum % 10;
            res[i + j] += sum / 10;
        }
    }

    string ans;

    for (int x : res) {
        if (!(ans.empty() && x == 0))
            ans += char(x + '0');
    }

    return ans.empty() ? "0" : ans;
}

string divideStrings(const string &num, const string &divisor) {
    if (divisor == "0")
        return "Infinity";

    auto smaller = [](const string &a, const string &b) {
        if (a.size() != b.size())
            return a.size() < b.size();
        return a < b;
    };

    auto subtract = [](string a, string b) {
        string res;
        int carry = 0;

        reverse(a.begin(), a.end());
        reverse(b.begin(), b.end());

        for (int i = 0; i < (int)a.size(); i++) {
            int x = a[i] - '0';
            int y = (i < (int)b.size() ? b[i] - '0' : 0);

            x -= y + carry;

            if (x < 0) {
                x += 10;
                carry = 1;
            } else {
                carry = 0;
            }

            res += char(x + '0');
        }

        while (res.size() > 1 && res.back() == '0')
            res.pop_back();

        reverse(res.begin(), res.end());
        return res;
    };

    string cur, ans;

    for (char c : num) {
        cur += c;

        while (cur.size() > 1 && cur[0] == '0')
            cur.erase(cur.begin());

        int q = 0;

        while (!smaller(cur, divisor)) {
            cur = subtract(cur, divisor);
            q++;
        }

        ans += char(q + '0');
    }

    int pos = 0;
    while (pos < (int)ans.size() - 1 && ans[pos] == '0')
        pos++;

    return ans.substr(pos);
}
```

---

## Usage

### 1. Arbitrary-Precision Multiplication (`multiplyStrings`)
Pass two non-negative decimal integer strings (without leading zeros):
```cpp
string a = "123456789012345678901234567890";
string b = "987654321098765432109876543210";

string prod = multiplyStrings(a, b);
// "121932631137021795226185032733622923332237463801111263526900"
```

### 2. Arbitrary-Precision Integer Division (`divideStrings`)
Computes floor division $\lfloor \text{num} / \text{divisor} \rfloor$:
```cpp
string q1 = divideStrings("100000000000000000000", "3");
// "33333333333333333333"

string q2 = divideStrings("42", "100");
// "0"
```

### 3. Obtaining Both Quotient and Remainder (`num % divisor`)
At the end of the `for (char c : num)` loop in `divideStrings`, `cur` holds the exact remainder `num % divisor` (or compute `subtract(num, multiplyStrings(q, divisor))`).

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

// Paste multiplyStrings and divideStrings here

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    // Compute 50! exactly
    string fact = "1";
    for (int i = 2; i <= 50; i++) {
        fact = multiplyStrings(fact, to_string(i));
    }
    cout << "50! = " << fact << "\n";
    // 30414093201713378043612608166064768844377641568960512000000000000

    // Divide 50! by 10^12
    cout << "50! / 10^12 = " << divideStrings(fact, "1000000000000") << "\n";

    return 0;
}
```

---

## Important Notes

1. **Non-Negative Normalized Inputs Assumed:** Both functions assume inputs consist strictly of digits `'0'`–`'9'` with no leading zeros (except `"0"` itself) and no `+`/`-` signs. If signed numbers are possible, strip and track signs (`negA ^ negB`) before calling.
2. **Const-Reference Parameters:** Updated parameter signatures to `const string &` (and `const string &` inside `smaller`) so string literals and `const` strings can be passed directly without unnecessary copies.
3. **Performance Scaling Limits:**
   * `multiplyStrings` runs in $O(N \cdot M)$, which handles up to $\sim 5,000$ digits easily. For numbers with $10^5+$ digits, use **FFT / NTT** convolution on base-$10^4$ chunks instead.
   * `divideStrings` runs at most $9$ subtractions per output digit, performing $O(N \cdot M)$ character operations overall.