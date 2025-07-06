

### Lambda 函数

用lambda函数初始化全局变量，可以在所有代码执行之前初始化，这里的lambda表达式必须要有个返回值（即使没有用），因为void类型的返回值无法赋值给一个init变量

这种写法通常被称为“**立即执行的函数表达式**”

```cpp

#define MX  100'000
#define MOD  1'000'000'007

long long pow2[MX+1];
auto init = [] {
    pow2[0]=1;
    for (int i=1;i<=MX;i++) {
        pow2[i]=(pow2[i-1]*2)%MOD;
    }
    return 0;
}();

``


### 遍历map

```cpp
    vector<int>& answers;
    unordered_map<int, int> counter;
    for (auto x: answers) {
        counter[x]++;
    }
    int ans=0;

    for (auto& [k, v]: counter) {
        ans += (v + k) / (k + 1) * (k + 1);
    }
```
### 初始化vector

```cpp
// 二维 n*2 vector
vector<vector<int>> dp(n, vector<int>(2, 0));
```

### 优先队列
priority_queue 默认是一个最大堆，即优先级最高的元素是最大的元素。<br>使用 greater<> 会将其变成最小堆。
```cpp
// 二个元素
priority_queue<pair<int, int>, vector<pair<int, int>>, greater<>> pq;

// 三个元素
priority_queue<tuple<int, int, int>, vector<tuple<int, int, int>>, greater<>> pq;

```

### 数组求和
```cpp
// 两种方法
long long s = accumulate(nums1.begin(), nums1.end(), 0LL);

long long s = reduce(candies.begin(), candies.end(), 0LL);
```

## ranges 库 (>=C++20)

[ref](https://cppreference.cn/w/cpp/algorithm)

### 计数
```cpp
// 统计x的个数
int z1 = ranges::count(nums, x);

// 自定义计数函数
ranges::count_if(words, [&](string x)->bool{
    return s.starts_with(x);
});
```

### 最大值
```cpp
// vector 求最大值
long long mx = ranges::max(candies);

// map 求最大值
unordered_map<int, int> dis;
ranges::max(dis | views::values);   //这个写法 leetcode 需要加 #include <ranges>

```

### 排序
```cpp
ranges::sort(cpx)   // vector 排序

// 可以有第三个参数，投影函数，把数组中的值映射为另一个数，再进行比较，有点类似python中sort的key参数
vector<int>& nums = {2,1,3,3};
vector<int> idx(n, 0);
ranges::iota(idx, 0);  // 从0开始的递增序列
ranges::sort(idx, {}, [&](int i) {return -nums[i];});

// 普通的排序
sort(intervals.begin(), intervals.end(), [](const std::vector<int> &a, const std::vector<int> &b)
     { return a[1] < b[1]; });
```
