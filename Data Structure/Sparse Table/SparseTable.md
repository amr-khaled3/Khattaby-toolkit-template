# دليل استخدام Sparse Table (الجدول المتناثر)

بنية البيانات **Sparse Table** هي هيكل بيانات يُستخدم للإجابة على استعلامات النطاق (Range Queries) الثابتة (التي لا تحتوي على تعديلات على العناصر) في وقت قياسي يبلغ **$O(1)$** للعمليات المتكررة (Idempotent Operations) مثل `min` و `max` و `GCD` و `Bitwise AND` و `Bitwise OR`.

---

## 📌 التعقيد الزمني والمكاني (Complexity)

| العملية | التعقيد (Complexity) | ملاحظات |
| :--- | :--- | :--- |
| **البناء (Precomputation)** | $O(N \log N)$ | يتم مرة واحدة عند تهيئة الهيكل. |
| **الاستعلام (Query)** | $O(1)$ | للعمليات المتكررة (Idempotent) مثل `&` و `min` و `max` و `gcd`. |
| **الذاكرة (Space)** | $O(N \log N)$ | لتخزين جدول `st` ثنائي الأبعاد. |

---

## 🛠️ الكود الأساسي (Implementation)

```cpp
struct SparseTable {
    int n, LOG;
    vector<vector<int>> st;
    vector<int> lg;

    SparseTable(const vector<int>& a) {
        n = a.size();
        if (n == 0) return; 
        LOG = 32 - __builtin_clz(n);

        st.assign(LOG, vector<int>(n));
        lg.assign(n + 1, 0);

        for (int i = 2; i <= n; i++)
            lg[i] = lg[i / 2] + 1;

        for (int i = 0; i < n; i++)
            st[0][i] = a[i];

        for (int j = 1; j < LOG; j++) {
            for (int i = 0; i + (1 << j) <= n; i++) {
                st[j][i] = merge(st[j - 1][i],
                                 st[j - 1][i + (1 << (j - 1))]);
            }
        }
    }

    inline int merge(int a, int b) {
        return (a & b); 
    }

    inline int query(int l, int r) {
        int j = lg[r - l + 1];
        return merge(st[j][l],
                     st[j][r - (1 << j) + 1]);
    }
};
```

---

## 🚀 طريقة الاستخدام (Usage)

### 1. التهيئة (Initialization)
قم بتمرير `vector<int>` إلى الـ Constructor الخاص بـ `SparseTable`:
```cpp
vector<int> arr = {7, 3, 15, 11, 2};
SparseTable st(arr);
```

### 2. الاستعلام (Querying)
تستخدم الدالة `query(l, r)` بنظام **0-based indexing** ويكون الحد الأيمن **مشمولاً (Inclusive)** أي في النطاق $[l, r]$:
```cpp
int ans = st.query(1, 3); // يحسب (arr[1] & arr[2] & arr[3])
```

---

## 💡 مثال عملي كامل

```cpp
#include <bits/stdc++.h>
using namespace std;

// ضع تعريف SparseTable هنا

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    vector<int> a = {12, 14, 15, 6, 7};
    SparseTable st(a);

    // استعلام من الفهرس 0 إلى 2 -> (12 & 14 & 15) = 12
    cout << st.query(0, 2) << "\n";

    // استعلام من الفهرس 1 إلى 4 -> (14 & 15 & 6 & 7) = 6
    cout << st.query(1, 4) << "\n";

    return 0;
}
```

---

## 🔄 تخصيص دالة الدمج (`merge`) لعمليات أخرى

يمكنك تغيير منطق الاستعلام بسهولة عبر تعديل دالة `merge` فقط:

* **أصغر عنصر (Range Minimum Query - RMQ):**
  ```cpp
  inline int merge(int a, int b) { return min(a, b); }
  ```
* **أكبر عنصر (Range Maximum Query):**
  ```cpp
  inline int merge(int a, int b) { return max(a, b); }
  ```
* **القاسم المشترك الأكبر (Range GCD):**
  ```cpp
  inline int merge(int a, int b) { return gcd(a, b); }
  ```
* **العملية المنطقية OR (Bitwise OR):**
  ```cpp
  inline int merge(int a, int b) { return (a | b); }
  ```

---

## ⚠️ ملاحظات هامة
1. **عدم وجود تعديلات (Static Array):** هذه البنية مخصصة للمصفوفات الثابتة؛ إذا تغيرت قيم المصفوفة فلن تتحدث الإجابات تلقائياً.
2. **العمليات المتكررة فقط (Idempotent Property):** يجب أن تحقق العملية الشرط $f(x, x) = x$. لذا **لا تصلح** هذه الطريقة لحساب المجموع (`Sum`) أو الضرب (`Product`) في وقت $O(1)$ بسبب تداخل النطاقين الجزئيين أثناء الاستعلام.
3. **التعامل مع `long long`:** إذا كانت القيم تتجاوز $2 \times 10^9$، استبدل `int` بـ `long long` في `st` ودوال `merge` و `query`.