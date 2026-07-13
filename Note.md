zj C316 最遠點對

凸包演算法 Quickhull（快包演算法）

Monotone Chain (安德魯單調鏈演算法)

旋轉卡殼法（Rotating Calipers）

### 動態規劃（Dynamic Programming)
- 費氏數列
  - 變形：爬樓梯(單次只能一階或二階) 

  基本：建立陣列 arr[n] = arr[n-1] + arr[n-2]
  進階：滾動變數優化 c = a + b, a = b, b = c 
  高級：參考下方資料
- 最大子陣列和
- 最長遞增子序列
- 二維不同路徑 / 矩陣 dp
- 最長公共子序列
- 完全背包問題
通常是雙層 for 迴圈，外層遍歷物品，內層由**小到大**跑容量。
```cpp=
//核心計算步驟
for(int i = 0; i < N; i++) {
    for(int j = 0; j <= W; j++) {
        dp[j] = max(dp[j], dp[j-items[i].weight] + items[i].val);
    }
}
```
- 0/1 背包問題
通常是雙層 for 迴圈，外層遍歷物品，內層由**大到小**跑容量。
```cpp=
//核心計算步驟
for(int i = 0; i < N; i++) {
    for(int j = W; j >= items[i].weight; j--) {
        dp[j] = max(dp[j], dp[j-items[i].weight] + items[i].val);
    }
}
```
### 質數相關
- 埃氏篩法
基本上可以處理大部分的質數搜查，由小到大找到第一次出現的數字即為質數
(零和一除外)，接下來把該質數的倍數都標記為合數，最後留下一個質數表。
```cpp=
#include <bits/stdc++.h>
using namespace std;

void sieveOfEratosthenes(int n) {
    vector<bool> isPrime(n + 1, true);

    // 0 和 1 不是質數
    if (n >= 0) isPrime[0] = false;
    if (n >= 1) isPrime[1] = false;

    // 從 2 開始篩選，直到開根號 n 即可
    for (int i = 2; i * i <= n; i++) {
        // 如果 i 是質數，則篩除它的倍數
        if (isPrime[i]) {
            //這邊要long long 是怕 i*i 這裡會溢位
            for (long long j = (long long)i * i; j <= n; j += i) {
                isPrime[j] = false;
            }
        }
    }

int main() {
    int n = 50; //搜查範圍
    sieveOfEratosthenes(n);
    return 0;
}
```
- 米勒-拉賓質數判定法
- 哥德巴赫猜想

### 費氏數列
- 費氏數列恆等式 (倍角恆等式)
偶數項公式：$F(2n) = F(n) \cdot [2F(n+1) - F(n)]$
奇數項公式：-$F(2n+1) = F(n+1)^2 + F(n)^2$
#### 遞迴版本 (Top-Down)
```cpp=
#include <bits/stdc++.h>
using namespace std;

const long long MOD = 1e9 + 7;

pair<long long, long long> fibDoublingRecursive(long long n) {
    // 終止條件（最底層）
    if (n == 0) return {0, 1};
    
    // 往下砍半探測 (n >>= 1)
    auto p = fibDoublingRecursive(n >> 1);
    long long fn = p.first;     // F(k)
    long long fn1 = p.second;   // F(k+1)
    
    // 公式翻倍：k -> 2k
    long long c = (fn * (2 * fn1 % MOD - fn + MOD)) % MOD; // F(2k)
    long long d = (fn1 * fn1 % MOD + fn * fn % MOD) % MOD; // F(2k+1)
    
    // 根據目前的 n 是奇數還是偶數，組合回傳答案
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
#### 非遞迴版本 (Bottom-Up) **(此方法可用於大數處理)**
下面的 i 是 for 迴圈中的 i，k 為所求的項數，f(x) 函數拿來計算第 x 項的結果，從第 0 項與第 1 項套入奇偶項公式 **(只是這邊是用反向計算，原本是透過原數除二往下找，現在是從最底層乘二向上加)**，直到算到 f(k)。
從 f(a) 跟 f(b) 開始，也就是從第零與第一項開始，最後的目標就是算到 f(k) 根據下方二進位轉十進位的方式，最後的結果會是一樣的。
```text=
i = 3      i = 2      i = 1      i = 0   (指針 i 從大變小)
     ┌───────┐  ┌───────┐  ┌───────┐  ┌───────┐
     │   1   │  │   1   │  │   0   │  │   1   │  (n = 13 的二進位)
     └───────┘  └───────┘  └───────┘  └───────┘
         │          │          │          │
  k=0 ──►┘          │          │          │
(項數)     k=1 ─────►┘          │          │
                    k=3 ──────►┘          │
                               k=6 ──────►┘
                                          k=13  (項數 k 其實是從小變大！)     
```
```cpp=
#include <bits/stdc++.h>
using namespace std;

const long long MOD = 1e9 + 7;

long long fibDoublingIterative(long long n) {
    if (n == 0) return 0;
    if (n == 1 || n == 2) return 1;

    long long a = 0; // 代表 F(k)
    long long b = 1; // 代表 F(k+1)

    // 找到 n 的二進位最高位 (例如 13 是 1101，最高位在第 3 位)
    int high_bit = 63 - __builtin_clzll(n);

    // 從最高位一路往最低位掃描
    for (int i = high_bit; i >= 0; i--) {
        /*

            這邊依舊用 1101 舉例，從左到右讀
            令一個數字 k = 0，程式碼中省去了 k，直接做向上替換
            
            第一個 1 抓起來 x2 + 1
            -> k1 = 0*2 + 1 = 1
            第二個 1 抓起來 x2 + 1
            -> k2 = k1 * 2 + 1 = 3
            第三個 0 抓起來 x2
            -> k3 =  k2 * 2 = 6
            第四個 1 抓起來 x2 + 1
            -> k4 = k3 * 2 + 1 = 13 //得到了 13 
            
            由高位數向下做 x2 且該位數為 1 時多做一個 +1 保留該位數
            這樣即可以透過累加還原該數字。
            
        */
        // 公式翻倍：k -> 2k，這邊出現 + MOD 是為了防負數，強行取正
        long long c = (a * (2 * b % MOD - a + MOD)) % MOD; // F(2k)
        long long d = (b * b % MOD + a * a % MOD) % MOD; // F(2k+1)

        a = c;
        b = d;

        // 如果目前位元是 1，項數再順推一項：2k -> 2k+1
        if ((n >> i) & 1) {
            /*
                位元為一代表算到這裡是奇數，以 k 的角度是十進位為奇數
                在二進位代表的是第 2^0 位等於 1
                這邊要處理的就是二進位的補位問題
                k 補位是 + 1，但 f(k) 就要變成 f(k+1)
                這邊 a = f(2k), b = f(2k+1)
                補位後變成 a = f(2k+1), b = f(2k) + f(2k+1) = f(2k + 2)
            */
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
- 矩陣快速冪 (Matrix Exponentiation)
透過下列矩陣運算可以得知，當第 n-1 與第 n 項乘上矩陣可以得到第 n 與 n+1項。
 $$\begin{bmatrix} F(n+1) \\ F(n) \end{bmatrix} = \begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix} \begin{bmatrix} F(n) \\ F(n-1) \end{bmatrix}$$
 因此我們將公式整理
 $$\begin{bmatrix} F(n+1) & F(n) \\ F(n) & F(n-1) \end{bmatrix} = \begin{bmatrix} 1 & 1 \\ 1 & 0 \end{bmatrix}^{n-1}$$
 這時，求費氏數列的問題，轉變成了「求一個矩陣的 $n-1$ 次方」。
 ```cpp=
#include <bits/stdc++.h>
using namespace std;

const long long MOD = 1e9 + 7;

void matrixMultiply(long long A[2][2], long long B[2][2]) {
	//矩陣運算  
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
	if(n == 0) {
		cout << 0;
		return 0;
	} else if(n == 1 || n == 2) {
		cout << 1;
		return 0;
	}
	
	long long ans[2][2] = {{1, 0}, {0, 1}};
	long long base[2][2] = {{1, 1}, {1, 0}};
	
	n--; // 計算 n-1 次方  
	
	while(n) {
        // n & 1 相當於是把數字轉乘二進位來計算
        /*
            譬如說 13 轉乘二進位變成 1101 這時候 base 相當於是2^0
            這是最後一號是 1 & 1 代表 ans 需要進入計算
            經過計算後 base 變成 2^1，接著 n >>= 1 做除二
            現在是 110，最後一號 0 & 1 是 0，代表 ans 不用加
            base 進入計算變成 2^2，n 再除二
            以此類推...
            利用二進制的 0、1 表示 ans 是否要加，base 依照二進制平方
            這樣可以讓計算時間降到 O(log n)
        */
		if(n & 1) {
			matrixMultiply(ans, base);
		}
		matrixMultiply(base, base); //基底平方 (把次數改成二進位表示)  
		n >>= 1; // 相當於 n / 2 
	}
	
	cout << ans[0][0];
	return 0;
}
```
