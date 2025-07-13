
### 方向

```c++
static constexpr int dirs[4][2] = {{-1, 0}, {1, 0}, {0, -1}, {0, 1}};
```

### 矩阵快速幂

```c++
class Solution {
    static constexpr int MOD = 1'000'000'007;
    static constexpr int SIZE = 26;

    using Matrix = array<array<int, SIZE>, SIZE>;

    // 返回矩阵 a 和矩阵 b 相乘的结果
    Matrix mul(Matrix& a, Matrix& b) {
        Matrix c{};
        for (int i = 0; i < SIZE; i++) {
            for (int k = 0; k < SIZE; k++) {
                if (a[i][k] == 0) {
                    continue;
                }
                for (int j = 0; j < SIZE; j++) {
                    c[i][j] = (c[i][j] + (long long) a[i][k] * b[k][j]) % MOD;
                }
            }
        }
        return c;
    }

    // 返回 n 个矩阵 a 相乘的结果
    Matrix pow(Matrix a, int n) {
        Matrix res = {};
        for (int i = 0; i < SIZE; i++) {
            res[i][i] = 1; // 单位矩阵
        }
        while (n) {
            if (n & 1) {
                res = mul(res, a);
            }
            a = mul(a, a);
            n >>= 1;
        }
        return res;
    }
}
```
### 区间的优先队列

```cpp
// 按 区间右端点 排序，越小的排在前面，如果相同，按区间左端点小的排在前面
std::priority_queue<std::vector<int>, std::vector<std::vector<int>>, 
                    [](const std::vector<int>& a, const std::vector<int>& b) {
                        if (a[1] == b[1]) return a[0] > b[0];
                        return a[1] > b[1];
                    }> hp;


```
