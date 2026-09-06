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