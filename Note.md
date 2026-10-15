# 演算法與資料結構筆記（Algorithm Notes）

---

## 一、 計算幾何（Computational Geometry）

### 1. 經典問題：zj C316 最遠點對
* **解法 1**：暴力 $O(N^2)$（適合 $N \le 1000$）
* **解法 2**：凸包 + 旋轉卡殼法 $O(N \log N)$（適合 $N \ge 10^5$）

### 2. 凸包演算法（Convex Hull）
* **Quickhull（快包演算法）**：分治法概念，平均 $O(N \log N)$，最壞 $O(N^2)$。
* **Monotone Chain（安德魯單調鏈演算法）**：
  * 先依 $X$ 座標（次要依 $Y$ 座標）排序 $O(N \log N)$。
  * 利用向量外積（Cross Product）分別維護下凸包與上凸包。

### 3. 旋轉卡殼法（Rotating Calipers）
* 基於凸包的多邊形性質，利用雙指針（卡尺）平行旋轉，在 $O(N)$ 時間內尋找最遠點對、最小外接矩形等。

---

## 二、 動態規劃（Dynamic Programming）

### 1. 經典基礎題目
* **費氏數列 / 爬樓梯**：
  * 單次只能走 1 階或 2 階，轉移式：$DP[n] = DP[n-1] + DP[n-2]$
  * **基本**：建立陣列 `arr[n] = arr[n-1] + arr[n-2]`
  * **滾動變數優化**：`c = a + b; a = b; b = c;`（空間複雜度 $O(1)$）
  * **高級**：倍角公式 / 矩陣快速冪 $O(\log n)$（詳見四、費氏數列高級演算法）
* **最大子陣列和（Kadane's Algorithm）**：`dp[i] = max(nums[i], dp[i-1] + nums[i])`
* **最長遞增子序列（LIS）**：DP $O(n^2)$ / 二分搜 + 貪婪 $O(n \log n)$
* **二維不同路徑 / 矩陣 DP**：`dp[i][j] = dp[i-1][j] + dp[i][j-1]`
* **最長公共子序列（LCS）**

### 2. 背包問題（Knapsack Problems）

#### 完全背包問題（物品可無限次選取）
* 雙層 `for` 迴圈：外層遍歷物品，內層由**小到大**跑容量。

```cpp
// 核心計算步驟
for (int i = 0; i < N; i++) {
    for (int j = items[i].weight; j <= W; j++) {
        dp[j] = max(dp[j], dp[j - items[i].weight] + items[i].val);
    }
}
```

#### 0/1 背包問題（物品只能選一次）
* 雙層 `for` 迴圈：外層遍歷物品，內層由**大到小**跑容量（避免同一物品被重複計算）。

```cpp
// 核心計算步驟
for (int i = 0; i < N; i++) {
    for (int j = W; j >= items[i].weight; j--) {
        dp[j] = max(dp[j], dp[j - items[i].weight] + items[i].val);
    }
}
```

---

## 三、 質數與數論（Prime & Number Theory）

### 1. 埃氏篩法（Sieve of Eratosthenes）
由小到大找到第一個未被標記的數即為質數，將其倍數標記為合數。時間複雜度 $O(n \log \log n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

void sieveOfEratosthenes(int n) {
    vector<bool> isPrime(n + 1, true);

    if (n >= 0) isPrime[0] = false;
    if (n >= 1) isPrime[1] = false;

    // 從 2 開始篩選，直到開根號 n 即可
    for (int i = 2; i * i <= n; i++) {
        if (isPrime[i]) {
            // (long long)i * i 避免溢位
            for (long long j = (long long)i * i; j <= n; j += i) {
                isPrime[j] = false;
            }
        }
    }
}

int main() {
    int n = 50; // 搜查範圍
    sieveOfEratosthenes(n);
    return 0;
}
```

### 2. 米勒-拉賓質數判定法（Miller-Rabin Primality Test）
* 概率性質數測試演算法，適合快速判斷大整數（如 $10^{18}$ 以上）是否為質數。
* 時間複雜度：$O(k \log^3 n)$。

### 3. 哥德巴赫猜想（Goldbach's Conjecture）
* 內容：任何大於 2 的偶數都可以表示為兩個質數之和。

---

## 四、 費氏數列高級演算法（Fibonacci Fast Doubling）

### 1. 費氏數列倍角恆等式
* **偶數項公式**：$F(2n) = F(n) \cdot [2F(n+1) - F(n)]$
* **奇數項公式**：$F(2n+1) = F(n+1)^2 + F(n)^2$

### 2. 遞迴版本（Top-Down）
時間複雜度 $O(\log n)$。

```cpp
#include <bits/stdc++.h>
using namespace std;

const long long MOD = 1e9 + 7;

pair<long long, long long> fibDoublingRecursive(long long n) {
    if (n == 0) return {0, 1};
    
    // 往下砍半探測 (n >>= 1)
    auto p = fibDoublingRecursive(n >> 1);
    long long fn = p.first;     // F(k)
    long long fn1 = p.second;   // F(k+1)
    
    // 公式翻倍：k -> 2k
    long long c = (fn * (2 * fn1 % MOD - fn + MOD)) % MOD; // F(2k)
    long long d = (fn1 * fn1 % MOD + fn * fn % MOD) % MOD; // F(2k+1)
    
    if (n & 1) { 
        return {d, (c + d) % MOD}; // 奇數項，回傳 { F(2k+1), F(2k+2) }
    } else {      
        return {c, d};             // 偶數項，回傳 { F(2k), F(2k+1) }
    }
}

int main() {
    ios_base::sync_with_stdio(0);
    cin.tie(0);
    long long n;
    cin >> n;
    cout << fibDoublingRecursive(n).first;
}
```

### 3. 非遞迴版本（Bottom-Up，適用於大數與位元處理）

```text
i = 3       i = 2       i = 1       i = 0   (指針 i 從大變小)
     ┌───────┐   ┌───────┐   ┌───────┐   ┌───────┐
     │   1   │   │   1   │   │   0   │   │   1   │  (n = 13 的二進位)
     └───────┘   └───────┘   └───────┘   └───────┘
         │           │           │           │
  k=0 ──►┘           │           │           │
(項數)     k=1 ─────►┘           │           │
                     k=3 ──────►┘           │
                                 k=6 ──────►┘
                                             k=13  (項數 k 從小變大)
```

```cpp
#include <bits/stdc++.h>
using namespace std;

const long long MOD = 1e9 + 7;

long long fibDoublingIterative(long long n) {
    if (n == 0) return 0;
    if (n == 1 || n == 2) return 1;

    long long a = 0; // F(k)
    long long b = 1; // F(k+1)

    // 找到 n 的二進位最高位
    int high_bit = 63 - __builtin_clzll(n);

    // 從最高位一路往最低位掃描
    for (int i = high_bit; i >= 0; i--) {
        // 公式翻倍：k -> 2k
        long long c = (a * (2 * b % MOD - a + MOD)) % MOD; // F(2k)
        long long d = (b * b % MOD + a * a % MOD) % MOD; // F(2k+1)

        a = c;
        b = d;

        // 如果目前位元是 1，項數順推一項：2k -> 2k+1
        if ((n >> i) & 1) {
            long long next_b = (a + b) % MOD;
            a = b;
            b = next_b;
        }
    }
    return a;
}

int main() {
    ios_base::sync_with_stdio(0);
    cin.tie(0);
    long long n;
    cin >> n;
    cout << fibDoublingIterative(n);
}
```

### 4. 矩陣快速冪（Matrix Exponentiation）

轉移矩陣公式：
$$
\begin{bmatrix} F(n+1) \\ F(n) \end{bmatrix} = \begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix} \begin{bmatrix} F(n) \\ F(n-1) \end{bmatrix} \implies \begin{bmatrix} F(n+1) & F(n) \\ F(n) & F(n-1) \end{bmatrix} = \begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}^{n-1}
$$

```cpp
#include <bits/stdc++.h>
using namespace std;

const long long MOD = 1e9 + 7;

void matrixMultiply(long long A[2][2], long long B[2][2]) {
    long long c00 = (A[0][0] * B[0][0] + A[0][1] * B[1][0]) % MOD;
    long long c01 = (A[0][0] * B[0][1] + A[0][1] * B[1][1]) % MOD;
    long long c10 = (A[1][0] * B[0][0] + A[1][1] * B[1][0]) % MOD;
    long long c11 = (A[1][0] * B[0][1] + A[1][1] * B[1][1]) % MOD;
    
    A[0][0] = c00; A[0][1] = c01;
    A[1][0] = c10; A[1][1] = c11;
}

int main() {
    ios_base::sync_with_stdio(0);
    cin.tie(0);
    long long n;
    cin >> n;
    if (n == 0) { cout << 0; return 0; }
    if (n == 1 || n == 2) { cout << 1; return 0; }
    
    long long ans[2][2] = {{1, 0}, {0, 1}};
    long long base[2][2] = {{1, 1}, {1, 0}};
    
    n--; // 計算 n-1 次方
    
    while (n) {
        if (n & 1) matrixMultiply(ans, base);
        matrixMultiply(base, base);
        n >>= 1;
    }
    
    cout << ans[0][0];
    return 0;
}
```

---

## 五、 排列組合與搜尋（Combinatorics, Backtracking & DP）

### 1. STL 工具介紹

#### `std::iota`（快速填入連續數）
```cpp
#include <numeric>
vector<int> v(5);
iota(v.begin(), v.end(), 1); // v = {1, 2, 3, 4, 5}
```

#### `std::next_permutation`（字典序排列）
* **升冪排列（從小到大）**：初始陣列需先排成小到大。
  ```cpp
  vector<int> v = {1, 2, 3};
  do {
      for (int x : v) cout << x;
      cout << " ";
  } while (next_permutation(v.begin(), v.end()));
  // 輸出：123 132 213 231 312 321
  ```
* **降冪排列（從大到小）**：初始陣列需先排成大到小，傳入 `greater<int>()`。
  ```cpp
  vector<int> v = {3, 2, 1};
  do {
      for (int x : v) cout << x;
      cout << " ";
  } while (next_permutation(v.begin(), v.end(), greater<int>()));
  // 輸出：321 312 231 213 132 123
  ```

---

### 2. 常見考法與模板（DFS / 回溯法）

#### (1) $1 \sim n$ 可重複選取，列出 $m$ 個
```cpp
void dfs_repeatable(int n, int m, vector<int>& path) {
    if (path.size() == m) {
        for (int x : path) cout << x << " ";
        cout << "\n";
        return;
    }
    for (int i = 1; i <= n; i++) {
        path.push_back(i);
        dfs_repeatable(n, m, path);
        path.pop_back(); // 回溯
    }
}
```

#### (2) $1 \sim n$ 不重複選取，列出 $m$ 個（組合 $C_m^n$）
```cpp
void dfs_combination(int start, int n, int m, vector<int>& path) {
    if (path.size() == m) {
        for (int x : path) cout << x << " ";
        cout << "\n";
        return;
    }
    for (int i = start; i <= n; i++) {
        path.push_back(i);
        dfs_combination(i + 1, n, m, path); // i + 1 避免重複選取
        path.pop_back();                    // 回溯
    }
}
```

#### (3) $1 \sim n$ 全排列 $P_n^n$（不重複）
```cpp
void dfs_permutation(int n, vector<bool>& visited, vector<int>& path) {
    if (path.size() == n) {
        for (int x : path) cout << x << " ";
        cout << "\n";
        return;
    }
    for (int i = 1; i <= n; i++) {
        if (!visited[i]) {
            visited[i] = true;
            path.push_back(i);
            dfs_permutation(n, visited, path);
            path.pop_back();    // 回溯
            visited[i] = false;
        }
    }
}
```

#### (4) 括號匹配問題（生成 $n$ 對合法括號）
```cpp
void dfs_parentheses(int left, int right, int n, string path) {
    if (path.length() == 2 * n) {
        cout << path << "\n";
        return;
    }
    if (left < n) {
        dfs_parentheses(left + 1, right, n, path + "(");
    }
    if (right < left) {
        dfs_parentheses(left, right + 1, n, path + ")");
    }
}
```

---

### 3. 組合數與計數 DP 應用

#### 組合數 DP（巴斯卡三角形求 $C_k^n$）
* **遞迴式**：$C(n, k) = C(n-1, k-1) + C(n-1, k)$
* **邊界條件**：$C(n, 0) = C(n, n) = 1$

```cpp
int C(int n, int k) {
    vector<vector<int>> dp(n + 1, vector<int>(k + 1, 0));
    for (int i = 0; i <= n; i++) {
        for (int j = 0; j <= min(i, k); j++) {
            if (j == 0 || j == i) dp[i][j] = 1;
            else dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j];
        }
    }
    return dp[n][k];
}
```

#### 合法括號計數（卡特蘭數 Catalan Number）
* **遞迴式**：$C_n = \sum_{i=0}^{n-1} C_i \times C_{n-1-i}$
* **邊界條件**：$C_0 = 1$

```cpp
int countParenthesesDP(int n) {
    vector<int> dp(n + 1, 0);
    dp[0] = 1;

    for (int i = 1; i <= n; i++) {
        for (int j = 0; j < i; j++) {
            dp[i] += dp[j] * dp[i - 1 - j];
        }
    }
    return dp[n];
}
```

---

### 4. 搜尋與組合問題複雜度對照表

| 問題類型 | 輸出目標 | 建議作法 | 時間複雜度 |
| :--- | :--- | :--- | :--- |
| **列舉全排列** | 具體路徑 | `next_permutation` / DFS 回溯 | $O(n \cdot n!)$ |
| **列舉組合 $C_k^n$** | 具體路徑 | DFS 回溯（帶 `start` 參數） | $O(k \cdot C_k^n)$ |
| **求組合總數 $C_k^n$** | 數字大小 | 巴斯卡三角形 DP | $O(n \cdot k)$ |
| **求合法括號總數** | 數字大小 | 卡特蘭數 DP | $O(n^2)$ |

## 六、容器

C++ STL 常見容器整理。使用時通常需包含對應標頭；競賽環境若支援，也可使用 `#include <bits/stdc++.h>`。

### 1. 序列容器

| 容器 | 標頭 | 特性與常見用途 |
| :--- | :--- | :--- |
| `vector<T>` | `<vector>` | 動態陣列；可隨機存取，尾端新增/刪除為均攤 $O(1)$，中間插入/刪除為 $O(n)$。最常用。 |
| `array<T, N>` | `<array>` | 固定長度陣列，大小在編譯期決定；可使用 `.size()`、迭代器等 STL 介面。 |
| `deque<T>` | `<deque>` | 雙端佇列；兩端新增/刪除為 $O(1)$，也可隨機存取。 |
| `list<T>` | `<list>` | 雙向鏈結串列；已知位置的插入/刪除為 $O(1)$，但不能用索引隨機存取。 |
| `forward_list<T>` | `<forward_list>` | 單向鏈結串列，較省空間；只能向前走，提供 `insert_after`、`erase_after`。 |
| `string` | `<string>` | 動態字元序列；支援索引、串接、搜尋等操作，常用於文字處理。 |

```cpp
vector<int> v = {3, 1, 2};
v.push_back(4);
sort(v.begin(), v.end()); // 排序：1, 2, 3, 4

deque<int> dq;
dq.push_front(1);
dq.push_back(2);          // dq = {1, 2}
```

`vector` 的尾端新增可能造成重新配置，讓原有的迭代器、指標與參考失效；若已知大約元素數量，可先用 `v.reserve(n)` 減少重新配置。

### 2. 佇列與堆疊容器配接器

| 容器 | 標頭 | 特性與常見用途 |
| :--- | :--- | :--- |
| `stack<T>` | `<stack>` | 後進先出（LIFO）；使用 `push`、`pop`、`top`。 |
| `queue<T>` | `<queue>` | 先進先出（FIFO）；使用 `push`、`pop`、`front`、`back`，常用於 BFS。 |
| `priority_queue<T>` | `<queue>` | 優先佇列，預設最大值優先；使用 `push`、`pop`、`top`，插入與取出為 $O(\log n)$。 |

```cpp
queue<int> q;             // FIFO
q.push(10);
int first = q.front();
q.pop();

priority_queue<int> maxHeap; // 預設：最大值優先
maxHeap.push(3);
maxHeap.push(8);
int largest = maxHeap.top(); // 8

priority_queue<int, vector<int>, greater<int>> minHeap; // 最小值優先
minHeap.push(3);
minHeap.push(8);
int smallest = minHeap.top(); // 3
```

注意：`pop()` 只會移除元素，不會回傳元素；需先透過 `top()`、`front()` 或 `back()` 讀取。

### 3. 有序關聯容器

| 容器 | 標頭 | 特性與常見用途 |
| :--- | :--- | :--- |
| `set<T>` | `<set>` | 不重複且依序排列；搜尋、插入、刪除為 $O(\log n)$。 |
| `multiset<T>` | `<set>` | 依序排列且允許重複；可用 `count(x)` 計算元素數量。 |
| `map<K, V>` | `<map>` | 依 key 排序的鍵值表；key 唯一，搜尋、插入、刪除為 $O(\log n)$。 |
| `multimap<K, V>` | `<map>` | 依 key 排序的鍵值表；允許多筆資料使用相同 key。 |

```cpp
set<int> s = {4, 1, 4}; // 自動去重並排序：{1, 4}
s.insert(3);
auto it = s.lower_bound(3); // 第一個 >= 3 的元素

map<string, int> score;
score["Amy"] = 95;
if (score.count("Amy")) {
    cout << score["Amy"];
}
```

### 4. 雜湊容器

| 容器 | 標頭 | 特性與常見用途 |
| :--- | :--- | :--- |
| `unordered_set<T>` | `<unordered_set>` | 不重複元素，不保證順序；搜尋、插入、刪除平均 $O(1)$。 |
| `unordered_multiset<T>` | `<unordered_set>` | 不保證順序且允許重複；適合快速計數或判斷元素是否存在。 |
| `unordered_map<K, V>` | `<unordered_map>` | 不保證順序的唯一 key 鍵值表；操作平均 $O(1)$。 |
| `unordered_multimap<K, V>` | `<unordered_map>` | 不保證順序且允許重複 key 的鍵值表。 |

雜湊容器的操作最壞情況可能退化至 $O(n)$，而且不適合需要排序或 `lower_bound` 的情況；需要有序結果時使用 `set` / `map`。

### 5. 特殊用途容器

#### `bitset<N>`：固定大小的位元集合
標頭：`<bitset>`。適合小範圍狀態、集合運算或快速標記；位元運算可一次處理多個布林狀態。

```cpp
bitset<8> bits;
bits.set(2);                 // 設定第 2 位
bits.flip(5);                // 翻轉第 5 位
bool exists = bits.test(2);
size_t count = bits.count(); // 計算 1 的個數
```

#### `span<T>`：檢視連續資料
標頭：`<span>`（C++20）。`span` 不擁有資料，只提供對陣列或 `vector` 連續區段的檢視；適合將不同來源的連續資料傳給同一個函式。請確保被檢視的資料在使用期間仍然有效。

## 七、Reference（參考）

Reference 是既有變數的另一個名稱（別名），不是一份新的資料。宣告時必須立即綁定到有效物件；綁定後不能改為參考另一個物件。透過 reference 讀寫，等同於直接操作原變數。

```cpp
int value = 10;
int& ref = value; // ref 是 value 的別名
ref = 20;         // value 也變成 20
```

### 函式參數的傳遞方式

#### 傳值：建立副本
參數會複製一份資料，函式內修改不影響呼叫端。適用於小型型別，或刻意需要獨立副本時。

```cpp
void addOne(int x) {
    x++;
}

int n = 1;
addOne(n); // n 仍為 1
```

#### 傳參考：直接操作原物件
參數成為呼叫端物件的別名，避免複製大型物件，也能讓函式修改原資料。

```cpp
void addOne(int& x) {
    x++;
}

int n = 1;
addOne(n); // n 變成 2
```

#### 傳常數參考：避免複製且禁止修改
使用 `const T&` 傳入大型物件，可避免複製，同時保證函式不會透過該參數修改物件。這是傳遞唯讀 `string`、`vector` 等大型物件的常見方式。

```cpp
void printValues(const vector<int>& values) {
    for (int x : values) {
        cout << x << " ";
    }
    // values.push_back(4); // 編譯錯誤：const reference 不可修改物件
}
```

### 函式回傳值與回傳參考

函式宣告的回傳型別寫在函式名稱之前。回傳參考的正確寫法是 `int& f()`，不是 `&int f()`；`int&` 表示回傳型別為「`int` 的參考」。

* **回傳值 `int f()`**：回傳一個值，呼叫端取得結果副本；修改結果不會改到函式內原本的物件。
* **回傳參考 `int& f()`**：回傳既有物件的別名；呼叫端可透過結果讀寫該物件，不會複製該物件。
* **回傳常數參考 `const int& f()`**：不複製物件，但呼叫端不能透過結果修改它。

```cpp
int getValue(vector<int>& values) {
    return values[0]; // 回傳 int 的值，呼叫端取得副本
}

int& getReference(vector<int>& values) {
    return values[0]; // 回傳第一個元素的參考
}

vector<int> numbers = {10, 20};
int copied = getValue(numbers);
int& element = getReference(numbers);

copied = 99;  // numbers[0] 仍是 10
element = 99; // numbers[0] 也變成 99
```

#### 容器操作常見的回傳型別

容器提供的元素存取介面通常會回傳參考，讓程式能直接操作容器內的元素；但並非每個容器函式都如此。

| 操作 | 常見回傳型別／行為 | 說明 |
| :--- | :--- | :--- |
| `vector<T>::operator[]`、`at()`、`front()`、`back()` | `T&`（容器為 `const` 時是 `const T&`） | 可直接讀寫對應元素。 |
| `map<K, V>::operator[]` | `V&` | 可讀寫 key 對應的 value；若 key 不存在，會先新增該 key。 |
| `priority_queue<T>::top()` | `const T&` | 不複製地讀取頂端元素，但不允許直接修改，以免破壞堆積順序。 |
| `queue<T>::front()`、`back()`；`stack<T>::top()` | `T&`（容器為 `const` 時是 `const T&`） | 參考目前端點或頂端元素。 |
| `pop()`、`erase()` 等移除操作 | 通常不回傳被移除的元素 | 需要元素時，先讀取 `top()` / `front()` 等，再呼叫移除操作。 |

```cpp
vector<int> values = {1, 2};
values[0] = 7;        // operator[] 回傳元素的參考，直接修改容器內容
int copy = values[0]; // 將元素值複製到 int
int& ref = values[0]; // ref 綁定到容器中的元素
```

參考不會擁有或延長容器元素的生命週期。若容器重新配置或刪除該元素，先前取得的參考可能失效；例如 `vector` 擴容時可能使所有元素參考失效。不要保留可能失效的參考供後續使用。

### 使用注意

* Reference 必須在宣告時初始化，且之後不能改綁定到其他物件。
* 不可回傳指向區域變數的 reference；區域變數離開作用域後就不存在，留下的 reference 會懸空。
* 可回傳仍然存活的物件或容器元素的 reference，但必須確保其生命週期涵蓋呼叫端使用該 reference 的期間。
* 若函式不需修改引數且不需建立副本，通常使用 `const T&`；若需修改呼叫端資料，使用 `T&`。
