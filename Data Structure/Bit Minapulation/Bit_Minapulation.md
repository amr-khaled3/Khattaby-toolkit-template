# Bit Manipulation Namespace Usage Guide

A C++ utility namespace (`Bit_Manipulation`) providing 64-bit (`long long`) bitwise operations, bit-position queries, power-of-two checks, and base-$b$ number conversion.

---

## Complexity Overview

| Function | Time Complexity | Space Complexity | Description |
| :--- | :--- | :--- | :--- |
| **`get_representation(n, b)`** | $O(\log_b n)$ | $O(\log_b n)$ | Converts $n$ into base $b$ (Least Significant Digit first). |
| **`isON(n, k)`** | $O(1)$ | $O(1)$ | Checks whether the $k$-th bit (0-indexed) of $n$ is set (`1`). |
| **`toggle(n, k)`** | $O(1)$ | $O(1)$ | Flips the $k$-th bit of $n$ (`0` $\leftrightarrow$ `1`). |
| **`set_bit(n, k)`** | $O(1)$ | $O(1)$ | Sets the $k$-th bit of $n$ to `1`. |
| **`clear_bit(n, k)`** | $O(1)$ | $O(1)$ | Clears the $k$-th bit of $n$ to `0`. |
| **`count_ones(n)`** | $O(1)$ | $O(1)$ | Returns the number of set bits (`1`s) in $n$. |
| **`is_power_of_two(n)`** | $O(1)$ | $O(1)$ | Returns `1` if $n = 2^k$ ($n > 0$), otherwise `0`. |
| **`lsb_pos(n)`** | $O(1)$ | $O(1)$ | Returns the 0-based index of the Lowest Significant Set Bit (`-1` if $n = 0$). |
| **`msb_pos(n)`** | $O(1)$ | $O(1)$ | Returns the 0-based index of the Most Significant Set Bit (`-1` if $n = 0$). |

---

## Implementation

```cpp
namespace Bit_Manipulation{
    // ~x means the bitwise NOT of x(complement)
    // preporties of XOR: a ^ a = 0, a ^ 0 = a
    // parity:- if(x & 1) return odd;
    // the inverse of ^ is itself, a ^ b = c -> a ^ c = b and b ^ c = a
    // __builtin_clzll(x) -> the number of leading zeros(leftmost for 64 bit)
    // __builtin_ctzll(x) -> the number of trailing zeros(rightmost for 64 bit)
    // __builtin_popcountll(x) -> the number of set bits in x
    vector<LL> get_representation(LL n, LL b){
        if(b == 1) return vector<LL>(1, n);
        else if(n == 0) return {0};
        vector<LL> res;
        while(n > 0) res.pb(n % b), n /= b;
        // while(sz(res) < 32) res.pb(0);
        // reverse(all(res));
        return res; }
    bool isON(LL n, LL k){
        // or return (n >> k) & 1;
        return n & (1LL << k); }
    LL toggle(LL n, LL k){
        return n ^ (1LL << k); }
    LL set_bit(LL n, LL k){
        return n | (1LL << k); }
    LL clear_bit(LL n, LL k){
        return n & (~(1LL << k)); }
    LL count_ones(LL n){
        return __builtin_popcountll(n); }
    LL is_power_of_two(LL n){
        return n > 0 && (n & (n - 1)) == 0; }
    
    // if you want a val make shift left by k, use (1LL << k) instead of (1 << k) to avoid overflow
    // if val is -1, means n = 0
    LL lsb_pos(LL n){
        return (n == 0) ? -1 : __builtin_ctzll(n); }
    LL msb_pos(LL n){
        if(n == 0) return -1;
        return 63 - __builtin_clzll(n); } 
}
```

---

## Usage

Bring the functions into scope with `using namespace Bit_Manipulation;` or prefix calls with `Bit_Manipulation::`. All bit indices `k` are **0-based** from right to left (LSB = `0`, MSB = `63`).

### 1. Single-Bit Inspection & Modification
```cpp
using namespace Bit_Manipulation;

LL mask = 10; // Binary: 1010 (bits 1 and 3 are ON)

bool b1 = isON(mask, 1);        // true
bool b2 = isON(mask, 2);        // false

mask = set_bit(mask, 2);        // 1010 -> 1110 (14)
mask = clear_bit(mask, 3);      // 1110 -> 0110 (6)
mask = toggle(mask, 0);         // 0110 -> 0111 (7)
```

### 2. Bit Counting & Bit Positions
```cpp
LL x = 40; // Binary: 101000

LL ones = count_ones(x);        // 2
LL low  = lsb_pos(x);           // 3 (rightmost set bit at 2^3)
LL high = msb_pos(x);           // 5 (leftmost set bit at 2^5)
bool p2 = is_power_of_two(x);   // false (0)
```

### 3. Base Conversion (`get_representation`)
Returns the digits of `n` in base `b` ordered from **Least Significant Digit (index `0`) to Most Significant Digit**:
```cpp
vector<LL> bin = get_representation(13, 2);
// 13 in binary is 1101 -> bin = {1, 0, 1, 1}
```

---

## Complete Example

```cpp
#include <bits/stdc++.h>
using namespace std;

using LL = long long;
#define pb push_back

// Paste Bit_Manipulation namespace here

using namespace Bit_Manipulation;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    LL n = 18; // Binary: 10010

    cout << "Set bits: " << count_ones(n) << "\n";
    cout << "LSB index: " << lsb_pos(n) << "\n";
    cout << "MSB index: " << msb_pos(n) << "\n";
    cout << "Is power of 2: " << is_power_of_two(n) << "\n";

    n = set_bit(n, 0);   // 10011 (19)
    n = toggle(n, 4);    // 00011 (3)
    cout << "Modified n: " << n << "\n";

    vector<LL> digits = get_representation(19, 2);
    for (LL d : digits) cout << d; // Prints 11001 (LSB first)
    cout << "\n";

    return 0;
}
```

---

## Important Notes

1. **Required Macros & Typedefs:** `get_representation` relies on `LL` (`long long`) and `pb` (`push_back`). Ensure both are defined before the namespace.
2. **Valid Bit Range ($0 \le k \le 62$):** Shifting `1LL << k` is well-defined for $0 \le k \le 62$ on signed 64-bit integers. Avoid $k = 63$ with signed `1LL` (use `1ULL << k` if manipulating the sign bit).
3. **Digit Order in `get_representation`:** Digits are stored in reverse order (LSB at index `0`), which makes `bin[k]` directly correspond to the $k$-th bit. Uncomment `reverse(all(res));` in the function if you need standard MSD-first order.
4. **Zero Handling:** Both `lsb_pos(0)` and `msb_pos(0)` safely guard against undefined behavior in `__builtin_ctzll(0)` and `__builtin_clzll(0)` by returning `-1`.