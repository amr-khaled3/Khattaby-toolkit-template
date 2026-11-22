# دليل استخدام Segment Tree (شجرة المقاطع)

بنية البيانات **Segment Tree** هي هيكل بيانات شجري يُستخدم للإجابة على استعلامات النطاق (Range Queries) وتحديث العناصر (Point Updates) بكفاءة عالية تبلغ **$O(\log N)$** لكل عملية، وتتميز بمرونتها لدعم مختلف العمليات التجميعية (Associative Operations) مثل الجمع، الضرب، الأصغر، الأكبر، و`GCD`.

---

## 📌 التعقيد الزمني والمكاني (Complexity)

| العملية | التعقيد (Complexity) | ملاحظات |
| :--- | :--- | :--- |
| **البناء (`init`)** | $O(N)$ | يتم بناء الشجرة بالكامل مرة واحدة. |
| **التحديث (`set`)** | $O(\log N)$ | تحديث قيمة عنصر واحد وتحديث أسلافه في الشجرة. |
| **الاستعلام (`get`)** | $O(\log N)$ | حساب النتيجة في النطاق نصف المفتوح $[l, r)$. |
| **الذاكرة (Space)** | $O(4N)$ | تحديداً `2 * tree_size` حيث `tree_size` هو أول قوة للعدد 2 أكبر من أو تساوي $N$. |

---

## 🛠️ الكود الأساسي (Implementation)

```cpp
struct Node
{
    int sum;

    Node()
    {
        sum = 0;
    }

    Node(int x)
    {
        sum = x;
    }

    void change(int x)
    {
        sum = x;
    }
};

struct SegTree
{
    int tree_size;
    vector<Node> SegData;

    SegTree(int n)
    {
        tree_size = 1;
        while (tree_size < n)
            tree_size *= 2;
        SegData.assign(2 * tree_size, Node());
    }

    // change
    Node merge(Node &lf, Node &ri)
    {
        Node ans = Node();
        ans.sum = lf.sum + ri.sum;

        return ans;
    }

    void init(vector<int> &arr, int ni, int lx, int rx)
    {
        if (rx - lx == 1)
        {
            if (lx < arr.size())
                SegData[ni] = Node(arr[lx]);
            return;
        }

        int mid = (rx + lx) / 2;
        init(arr, 2 * ni + 1, lx, mid);
        init(arr, 2 * ni + 2, mid, rx);

        SegData[ni] = merge(SegData[2 * ni + 1], SegData[2 * ni + 2]);
    }

    void init(vector<int> &arr)
    {
        init(arr, 0, 0, tree_size);
    }

    void set(int idx, int val, int ni, int lx, int rx)
    {
        if (rx - lx == 1)
        {
            SegData[ni].change(val);
            return;
        }

        int mid = (rx + lx) / 2;
        if (idx < mid)
        {
            set(idx, val, 2 * ni + 1, lx, mid);
        }
        else
        {
            set(idx, val, 2 * ni + 2, mid, rx);
        }

        SegData[ni] = merge(SegData[2 * ni + 1], SegData[2 * ni + 2]);
    }
    

    void set(int idx, int val)
    {
        set(idx, val, 0, 0, tree_size);
    }

    void update_range(int l, int r, int ni, int lx, int rx) {
        if (lx >= r || rx <= l || SegData[ni].mx < 10)
            return;

        if (rx - lx == 1) {
            int nxt = digit_sum(SegData[ni].mx);
            SegData[ni] = Node(nxt);
            return;
        }

        int mid = (rx + lx) / 2;
        update_range(l, r, 2 * ni + 1, lx, mid);
        update_range(l, r, 2 * ni + 2, mid, rx);
        SegData[ni] = merge(SegData[2 * ni + 1], SegData[2 * ni + 2]);
    }

    void update_range(int l, int r) {
        update_range(l, r, 0, 0, tree_size);
    }

    Node get(int l, int r, int ni, int lx, int rx)
    {
        if (lx >= l && rx <= r)
            return SegData[ni];
        if (lx >= r || rx <= l)
            return Node();

        int mid = (rx + lx) / 2;
        Node lf = get(l, r, 2 * ni + 1, lx, mid);
        Node ri = get(l, r, 2 * ni + 2, mid, rx);

        return merge(lf, ri);
    }

    int get(int l, int r)
    {
        return get(l, r, 0, 0, tree_size).sum;
    }
};
```

---

## 🚀 طريقة الاستخدام (Usage)

### 1. التهيئة والبناء (Initialization & Build)
قم بإنشاء الشجرة بتمرير حجم المصفوفة $N$، ثم استدعِ `init` لتعبئتها بعناصر المصفوفة:
```cpp
int n = 5;
vector<int> arr = {1, 3, 5, 7, 9};

SegTree st(n);
st.init(arr);
```

### 2. الاستعلام (Querying)
تستخدم الدالة `get(l, r)` بنظام **0-based indexing** وبنطاق نصف مفتوح **$[l, r)$** (أي يشمل `l` ولا يشمل `r`):
```cpp
// لحساب مجموع العناصر من الفهرس 1 إلى 3 (arr[1] + arr[2] + arr[3])
int ans = st.get(1, 4); 
```

### 3. التحديث (Updating)
تستخدم الدالة `set(idx, val)` لتعديل العنصر عند الفهرس `idx` (0-based):
```cpp
// في الكود الحالي، دالة change تقوم بإضافة val إلى القيمة الحالية: arr[2] += 10
st.set(2, 10);
```

---

## 💡 مثال عملي كامل

```cpp
#include <bits/stdc++.h>
using namespace std;

// ضع تعريف Node و SegTree هنا

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    vector<int> a = {2, 4, 6, 8, 10};
    SegTree st(a.size());
    st.init(a);

    // استعلام في النطاق [0, 3) -> (2 + 4 + 6) = 12
    cout << st.get(0, 3) << "\n";

    // إضافة 5 إلى العنصر في الفهرس 1 (تصبح قيمته 4 + 5 = 9)
    st.set(1, 5);

    // استعلام في النطاق [0, 3) بعد التحديث -> (2 + 9 + 6) = 17
    cout << st.get(0, 3) << "\n";

    return 0;
}
```

---

## 🔄 تخصيص الكود لعمليات أخرى

لتغيير العملية التي تقوم بها الشجرة، تحتاج إلى تعديل **3 أجزاء** رئيسية فقط:
1. **العنصر المحايد (Neutral Element)** داخل `Node()`.
2. **طريقة التحديث** داخل `change(int x)` (هل هي إضافة `+=` أم استبدال `=`).
3. **طريقة الدمج** داخل `merge(Node &lf, Node &ri)`.

* **استبدال القيمة بدلاً من الإضافة (Point Assignment):**
  ```cpp
  void change(int x) { sum = x; } // بدلاً من sum += x
  ```
* **أصغر عنصر (Range Minimum Query - RMQ):**
  ```cpp
  Node() { sum = INT_MAX; } // العنصر المحايد للـ min
  Node merge(Node &lf, Node &ri) {
      return Node(min(lf.sum, ri.sum));
  }
  ```
* **أكبر عنصر (Range Maximum Query):**
  ```cpp
  Node() { sum = INT_MIN; } // العنصر المحايد للـ max
  Node merge(Node &lf, Node &ri) {
      return Node(max(lf.sum, ri.sum));
  }
  ```

---

## ⚠️ ملاحظات هامة
1. **النطاق نصف المفتوح $[l, r)$:** تذكر دائماً أن الحد الأيمن `r` غير مشمول في الاستعلام `get(l, r)`. إذا كنت تريد الاستعلام من `l` إلى `r` (شاملاً `r`)، استدعِ `st.get(l, r + 1)`.
2. **سلوك دالة `set`:** في القالب الحالي، `set(idx, val)` تقوم **بزيادة** قيمة العنصر بمقدار `val` (`sum += x`). إذا كانت المسألة تتطلب **تعيين** قيمة جديدة (`arr[idx] = val`)، غيّر `sum += x` إلى `sum = x`.
3. **تجنب Overflow:** في مسائل المجموع، غالباً ما يتجاوز المجموع حدود `int`، لذا يُنصح بتغيير نوع `sum` والمتغيرات المرتبطة به إلى `long long`.