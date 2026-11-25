# Prime Numbers, Sieves, Factorization & Multiplicative Functions Guide

A comprehensive C++ reference covering primality testing, classical & segmented sieves, Smallest Prime Factor (SPF) factorization, Euler's Totient Function ($\varphi$), and the Möbius Function ($\mu$).

---

## Complexity Overview

| Function | Time Complexity | Space Complexity | Purpose |
| :--- | :--- | :--- | :--- |
| **`isPrime(n)`** | $O(\sqrt{n})$ | $O(1)$ | Single-number primality check up to $10^{14}$ (with `LL`). |
| **`sieve()`** | $O(N \log \log N)$ | $O(N)$ | Boolean prime lookup table `isPrime[0..N-1]`. |
| **`primes(n)`** | $O(N \log \log N)$ | $O(N)$ | Populates global `vector<LL> prims` with all primes $\le n$. |
| **`segmentedSieve(l, r)`** | $O((R - L + 1)\log\log R + \sqrt{R})$ | $O(\sqrt{R} + S)$ | Sieves large ranges $[l, r]$ (e.g., $R \le 3 \times 10^8$) in cache-friendly chunks of size $S$. |
| **`next_prime(num)`** | $O(\log \pi(N))$ | $O(1)$ | Binary searches `prims` for the smallest prime $\ge \text{num}$. |
| **`getPrimeFactor(n)`** | $O(\sqrt{n})$ | $O(\omega(n))$ | Trial-division prime factorization returning `{prime, exponent}` pairs. |
| **`largestPrimeFactor(n)`** | $O(\sqrt{n} / \ln \sqrt{n})$ | $O(1)$ | Finds the largest prime factor of $n$ using precomputed `prims`. |
| **`buildSPF()`** | $O(N \log \log N)$ | $O(N)$ | Computes `spf[x]` (Smallest Prime Factor of $x$) for fast $O(\log x)$ factorization. |
| **`bulidCnt()` / `bulidPref()`** | $O(N)$ | $O(N)$ | Computes $\Omega(x)$ (total prime factors with multiplicity) and its prefix sums. |
| **`getFactorization(arr)`** | $O(N \log \log N + \sum \log a_i)$ | $O(N + \sum \log a_i)$ | Factorizes every integer in `arr` into its sorted prime factors using `spf`. |
| **`phi_generator()`** | $O(N \log \log N)$ | $O(N)$ | Computes Euler's Totient $\varphi(i)$ for all $0 \le i < \text{MAXN}$. |
| **`phi_factn(n)`** | $O(n \sqrt{n})$ | $O(1)$ | Computes $\varphi(n!)$ directly using the product formula. |
| **`mobiusSieve()`** | $O(N \log \log N)$ | $O(N)$ | Computes the Möbius function $\mu(i) \in \{-1, 0, 1\}$ for all $1 \le i < \text{MAXN}$. |

---

## Implementation

### 1. Basic Primality & Classical Sieve

```cpp
using LL = long long;
const int N = 1e6 + 6;

bool isPrimeArr[N];
vector<LL> prims;
bool notPrime[N];

// Single-number primality check: O(sqrt(n))
bool isPrime(LL n) {
    if (n <= 1) return false;
    for (LL i = 2; i * i <= n; i++)
        if (n % i == 0) return false;
    return true;
}

// Boolean sieve: O(N log log N)
void sieve() {
    fill(isPrimeArr, isPrimeArr + N, true);
    isPrimeArr[0] = isPrimeArr[1] = false;
    for (int i = 2; i * i < N; i++) {
        if (isPrimeArr[i]) {
            for (int m = i * i; m < N; m += i)
                isPrimeArr[m] = false;
        }
    }
}

// Collect primes up to n into global `prims`: O(n log log n)
void primes(LL n) {
    prims.clear();
    for (LL i = 2; i <= n; i++) {
        if (!notPrime[i]) {
            prims.push_back(i);
            for (LL j = i * i; j <= n; j += i) {
                notPrime[j] = true;
            }
        }
    }
}

// Smallest prime >= num (requires `primes(n)` first)
LL next_prime(LL num) {
    auto it = lower_bound(prims.begin(), prims.end(), num);
    if (it != prims.end()) {
        return *it;
    }
    return -1;
}
```

### 2. Segmented Sieve (Range $[l, r]$)

```cpp
const int MAX_R = 300000000;
const int SEGMENT_SIZE = 1000000;

int segmentedSieve(int l, int r) {
    int sqrtR = sqrt(r) + 1;

    vector<bool> isPrimeSmall(sqrtR + 1, true);
    vector<int> smallPrimes;
    isPrimeSmall[0] = isPrimeSmall[1] = false;

    for (int i = 2; i <= sqrtR; i++) {
        if (isPrimeSmall[i]) {
            smallPrimes.push_back(i);
            for (int j = i * i; j <= sqrtR; j += i) {
                isPrimeSmall[j] = false;
            }
        }
    }

    int count = 0;

    for (int low = l; low <= r; low += SEGMENT_SIZE) {
        int high = min(low + SEGMENT_SIZE - 1, r);
        vector<char> segment(high - low + 1, true);

        for (int p : smallPrimes) {
            if ((long long)p * p > high) break;

            long long start = max((long long)p * p,
                                  ((long long)low + p - 1) / p * p);

            if (p == 2) {
                if (start % 2 == 0) start += p;
            }

            for (long long j = start; j <= high; j += p) {
                segment[j - low] = false;
            }
        }

        if (low <= 1) {
            if (1 >= low && 1 <= high) segment[1 - low] = false;
            if (0 >= low && 0 <= high) segment[0 - low] = false;
        }

        // Counts primes == 2 or p ≡ 1 (mod 4) (sum of two squares)
        for (int i = low; i <= high; i++) {
            if (segment[i - low]) {
                if (i == 2 || i % 4 == 1) {
                    count++;
                }
            }
        }
    }

    return count;
}
```

### 3. Trial-Division Factorization & Largest Prime Factor

```cpp
// Prime factorization as {prime, exponent} pairs: O(sqrt(n))
vector<pair<LL, int>> getPrimeFactor(LL n) {
    vector<pair<LL, int>> ret;
    for (LL p = 2; p * p <= n; p++) {
        int e = 0;
        while (n % p == 0) {
            n /= p;
            e++;
        }
        if (e > 0) ret.push_back({p, e});
    }
    if (n > 1) ret.push_back({n, 1});
    return ret;
}

// Largest prime factor of n (requires `primes()` precomputed up to sqrt(n))
LL largestPrimeFactor(LL n) {
    LL res = 1;
    for (int i = 0; i < (int)prims.size() && prims[i] * prims[i] <= n; i++) {
        while (n % prims[i] == 0) {
            res = max(res, prims[i]);
            n /= prims[i];
        }
    }
    return max(res, n);
}
```

### 4. Smallest Prime Factor (SPF) & Fast Batch Factorization

```cpp
int spf[N], cnt[N];
LL pref[N];

void buildSPF() {
    for (int i = 1; i < N; i++) spf[i] = i;

    for (int i = 2; i * i < N; i++) {
        if (spf[i] == i) {
            for (int j = i * i; j < N; j += i) {
                if (spf[j] == j) spf[j] = i;
            }
        }
    }
}

// Total prime factors with multiplicity Omega(i)
void bulidCnt() {
    cnt[1] = 0;
    for (int i = 2; i < N; i++) {
        cnt[i] = cnt[i / spf[i]] + 1;
    }
}

// Prefix sums of Omega(i)
void bulidPref() {
    pref[0] = 0;
    for (int i = 1; i < N; i++) {
        pref[i] = pref[i - 1] + cnt[i];
    }
}

// Factorizes every element in `arr` in O(log x) per element
vector<vector<int>> getFactorization(const vector<int>& arr) {
    buildSPF();
    vector<vector<int>> ret;
    ret.reserve(arr.size());
    for (int x : arr) {
        vector<int> Pfactors;
        while (x > 1) {
            Pfactors.push_back(spf[x]);
            x /= spf[x];
        }
        ret.push_back(Pfactors);
    }
    return ret;
}
```

### 5. Euler's Totient ($\varphi$) & Möbius Function ($\mu$)

```cpp
const int MAXN = 1e6 + 5;
char phiPrime[MAXN];
int phi[MAXN];

// Euler's Totient Sieve: phi[i] = count of 1 <= k <= i with gcd(k, i) == 1
void phi_generator() {
    memset(phiPrime, 1, sizeof(phiPrime));

    for (int i = 0; i < MAXN; i++)
        phi[i] = i;

    phiPrime[0] = phiPrime[1] = 0;

    for (int i = 2; i < MAXN; i++) {
        if (phiPrime[i]) {
            for (int j = i; j < MAXN; j += i) {
                phiPrime[j] = 0;
                phi[j] -= phi[j] / i;
            }
        }
    }
}

// Euler's Totient of n!: phi(n!)
LL phi_factn(int n) {
    LL res = 1;
    for (int i = 2; i <= n; i++)
        res = res * (isPrime(i) ? i - 1 : i);
    return res;
}

// Möbius Function Sieve
int mu[MAXN];
bool mobPrime[MAXN];

void mobiusSieve() {
    for (int i = 1; i < MAXN; i++) {
        mu[i] = 1;
        mobPrime[i] = true;
    }

    mobPrime[0] = mobPrime[1] = false;

    for (int i = 2; i < MAXN; i++) {
        if (mobPrime[i]) {
            for (int j = i + i; j < MAXN; j += i)
                mobPrime[j] = false;

            for (int j = i; j < MAXN; j += i)
                mu[j] *= -1;

            long long sq = 1LL * i * i;
            for (long long j = sq; j < MAXN; j += sq)
                mu[j] = 0;
        }
    }
}
```

---

## Usage

### 1. Primes & Single-Number Factorization
```cpp
primes(1000000);
LL nxt = next_prime(50);              // 53
LL lpf = largestPrimeFactor(13195);   // 29

auto pf = getPrimeFactor(360);
// pf = {{2, 3}, {3, 2}, {5, 1}} -> 2^3 * 3^2 * 5^1
```

### 2. Fast Query Factorization with `SPF`
Call `buildSPF()` once at the start of `main()`. For any $x < N$, factorize in $O(\log x)$:
```cpp
buildSPF();
bulidCnt();
bulidPref();

// Number of prime factors in n! = a! / b! -> pref[a] - pref[b]
LL factorsInRange = pref[10] - pref[5];
```

### 3. Euler's Totient ($\varphi$) & Möbius ($\mu$)
```cpp
phi_generator();
int coprimesTo12 = phi[12]; // 4 ({1, 5, 7, 11})

mobiusSieve();
// mu[1] = 1, mu[6] = 1 (2*3), mu[12] = 0 (divisible by 2^2), mu[30] = -1 (2*3*5)
```

---

## Important Notes

1. **Bug Fixes Applied from Original Snippets:**
   * **`phi_factn` Typo:** Fixed `res = ret * ...` to `res = res * ...` (as `ret` was undeclared).
   * **`primes(n)` Inner Loop:** Moved `for (LL j = i * i; ...)` inside `if (!notPrime[i])` so composite `i` values do not waste $O(N \log N)$ work.
   * **64-Bit Loops in `isPrime` & `getPrimeFactor`:** Changed `int i` / `int p` to `LL` so `i * i <= n` does not overflow 32-bit `int` when $n > 46340^2$.
   * **Bounds Guard in `largestPrimeFactor`:** Added `i < (int)prims.size()` before `prims[i] * prims[i] <= n` to prevent out-of-bounds access when `prims` only contains primes up to $\sqrt{n}$.
   * **Unified `MAXN` & Naming:** Renamed colliding global `primes` / `isPrime` / `MAXN` identifiers so all functions compile cleanly together in one template file.
2. **Segmented Sieve Filter:** Notice that `segmentedSieve(l, r)` specifically counts primes $p \in [l, r]$ where $p = 2$ or $p \equiv 1 \pmod 4$ (primes expressible as a sum of two squares). To count **all** primes in $[l, r]$, remove the `if (i == 2 || i % 4 == 1)` condition.
3. **One-Time Precomputation:** Always call `sieve()`, `buildSPF()`, `phi_generator()`, and `mobiusSieve()` **once** before the `TEST` loop—never inside per-testcase functions.