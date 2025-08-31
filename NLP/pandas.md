

2025/4/26

## 1. groupby用法

在使用 groupby() 后，分组的结果通常会将分组键作为索引。如果需要将分组键恢复为普通列，可以使用 reset_index()

```python
def game_analysis(activity: pd.DataFrame) -> pd.DataFrame:
    grouped = activity.groupby('player_id')
    ans = grouped['event_date'].min().reset_index()
    ans.rename(columns={'event_date': 'first_login'}, inplace=True)
    return ans
```

可以groupby多个字段
```python
grouped = df.groupby(['Category', 'Region'])
```

### min的写法
统计每个分组中某个字段的最小值
```python
grouped = activity.groupby('player_id')
ans = grouped['event_date'].min().reset_index()
```

### max的写法
统计每个分组中某个字段的最小值
```python
grouped = activity.groupby('player_id')
ans = grouped['event_date'].min().reset_index()
```

### size的写法

统计每个分组中记录的个数
```python
grouped = selected.groupby('managerId')
g1 = grouped.size().reset_index(name='count')
print(g1.to_markdown(index=False))
```

|   managerId |   count |
|------------:|--------:|
|         101 |       5 |


### sum/count 的写法
```python
print('1:', df.to_markdown(index=False))
```
|   id |   client_id |   driver_id |   city_id | status              | request_at   |   comp |
|-----:|------------:|------------:|----------:|:--------------------|:-------------|-------:|
|    1 |           1 |          10 |         1 | completed           | 2013-10-01   |      1 |
|    3 |           3 |          12 |         6 | completed           | 2013-10-01   |      1 |
|    4 |           4 |          13 |         6 | cancelled_by_client | 2013-10-01   |      0 |
|    5 |           1 |          10 |         1 | completed           | 2013-10-02   |      1 |
|    7 |           3 |          12 |         6 | completed           | 2013-10-02   |      1 |
|    9 |           3 |          10 |        12 | completed           | 2013-10-03   |      1 |
|   10 |           4 |          13 |        12 | cancelled_by_driver | 2013-10-03   |      0 |

按request_at分组统计comp的和，组内的记录数， 并重新命名。agg函数可以几列聚集列

```python
grouped = df.groupby('request_at')
ans = grouped.agg({'comp': 'sum', 'request_at': 'count'}).rename(columns={'request_at': 'count'}).reset_index()
print('2:', ans.to_markdown(index=False))
```
| request_at   |   comp |   count |
|:-------------|-------:|--------:|
| 2013-10-01   |      2 |       3 |
| 2013-10-02   |      2 |       2 |
| 2013-10-03   |      1 |       2 |

### 综合用法
groupby + 汇总 同时进行
```python
ans = (
        transactions
        .groupby(['country', 'month'], dropna=False)   # 显式保留 NaN
        .agg(
                trans_count=('amount', 'size'),  # 新增：统计每组的行数
                approved_count=('ca', 'sum'),
                trans_total_amount=('amount', 'sum'),
                approved_total_amount=('aa', 'sum')
        )
    ).reset_index()
```

### 分组后执行复杂的运算
```python
ans = (
        result.groupby('product_id')
        .apply(lambda g: round(g['units'].mul(g['price']).sum() / g['units'].sum(), 2))
        .reset_index(name='average_price')
    )
```


### 在每个分组内执行操作
使用 **transform** 对每个分组应用**聚合函数**（如 count），并将结果广播到原始 DataFrame 的每一行。

```python
# 使用 groupby 和 transform 计算每个 t_rank 分组的行数

    print(df.to_markdown(index=False))
    df['count'] = df.groupby('t_rank')['t_rank'].transform('count')
    // ['t_rank'] 并没有实际的作用，count函数可以作用与任何列，但需要写一个
    print(df.to_markdown(index=False))

```
|   index |   id | visit_date          |   people |   t_rank |
|--------:|-----:|:--------------------|---------:|---------:|
|       0 |    2 | 2017-01-02 00:00:00 |      109 |        2 |
|       1 |    3 | 2017-01-03 00:00:00 |      150 |        2 |
|       2 |    5 | 2017-01-05 00:00:00 |      145 |        3 |
|       3 |    6 | 2017-01-06 00:00:00 |     1455 |        3 |
|       4 |    7 | 2017-01-07 00:00:00 |      199 |        3 |
|       5 |    8 | 2017-01-09 00:00:00 |      188 |        3 |


|   index |   id | visit_date          |   people |   t_rank |   cnt |
|--------:|-----:|:--------------------|---------:|---------:|------:|
|       0 |    2 | 2017-01-02 00:00:00 |      109 |        2 |     2 |
|       1 |    3 | 2017-01-03 00:00:00 |      150 |        2 |     2 |
|       2 |    5 | 2017-01-05 00:00:00 |      145 |        3 |     4 |
|       3 |    6 | 2017-01-06 00:00:00 |     1455 |        3 |     4 |
|       4 |    7 | 2017-01-07 00:00:00 |      199 |        3 |     4 |
|       5 |    8 | 2017-01-09 00:00:00 |      188 |        3 |     4 |


组内取最小值，过滤出到达最小值的行
```python
    sales['first_year'] = sales.groupby('product_id')['year'].transform('min')
    # print(sales)
    sales[sales['year'] == sales['first_year']][['product_id', 'first_year', 'quantity', 'price']]
```

|   sale_id |   product_id |   year |   quantity |   price |
|----------:|-------------:|-------:|-----------:|--------:|
|         1 |          100 |   2008 |         10 |    5000 |
|         2 |          100 |   2009 |         12 |    5000 |
|         7 |          200 |   2011 |         15 |    9000 |

|   product_id |   first_year |   quantity |   price |
|-------------:|-------------:|-----------:|--------:|
|          100 |         2008 |         10 |    5000 |
|          200 |         2011 |         15 |    9000 |

## 2. merge 用法

```python
pd.merge(table1, table2, left_on='player_id', right_on='player_id', how='inner')
# 笛卡尔积，不需要关联字段
ss = pd.merge(students, subjects, how='cross')
```

**how**的取值还可以有 **left/right/outer**

如果是两个关联字段
```python
# 按 EmployeeID 和 Year 两个字段进行关联
result = df1.merge(df2, on=['EmployeeID', 'Year'], how='inner')

# 按 EmployeeID 和 Year 两个字段进行关联，字段名不同
result = df1.merge(df2, left_on=['EmployeeID', 'Year'], right_on=['ID', 'Year'], how='inner')
```

在 pandas 的 merge 操作中，如果某些行在另一个 DataFrame 中没有匹配的记录，合并后这些字段会变成 NaN。如果你想把这些 NaN 设置成默认值，可以使用 .fillna()。
```python
ans = pd.merge(users, g, left_on='user_id', right_on='buyer_id', how='left')
ans['count'] = ans['count'].fillna(0)
```

## 列求和
```python
total_sales = df['Sales'].sum()

# 使用 apply() 方法求和
total_sales = df['Sales'].apply(lambda x: x).sum()
```

## 3. loc 用法

### 修改某(些)列的值
```python
# 修改 score 大于 80 的行的 score 值，减去 10
df.loc[df['score'] > 80, 'score'] -= 10

# 修改 id 为 3 的行的 score 值，设置为 100，并将 name 设置为 "New Name"
df.loc[df['id'] == 3, ['score', 'name']] = [100, 'New Name']
```
**iloc** 可以定位第i行

```python
    for i, row in grouped.iterrows():  # 遍历每条记录
        if i < 6:
            cur += row['amount']
            continue
        cur += row['amount']
        if i > 6:
            cur -= grouped.iloc[i - 7]['amount']
        ans.append([row['visited_on'], cur, round(cur / 7, 2)])
```


## 4. 行转列

pivot() 是 pandas.DataFrame 的一个“长转宽”变形函数，它把多行记录按照某个键值展开成列，常用于透视表或报表格式化。
```python
DataFrame.pivot(index=None, columns=None, values=None)
```
| 参数        | 说明                                                  |
| --------- | --------------------------------------------------- |
| `index`   | 用作新 DataFrame **行索引** 的列名（可以多个）                     |
| `columns` | 用作新 DataFrame **列名** 的列（其唯一值会变成列）                   |
| `values`  | 要填充到新 DataFrame **单元格** 的列（只能一个；多个请用 `pivot_table`） |

举例

```python
def reformat_table(department: pd.DataFrame) -> pd.DataFrame:
    # 更有解法
    months = ["Jan", "Feb", "Mar", "Apr", "May", "Jun",
              "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"]

    # 使用 pivot 重塑数据
    data = department.pivot(index='id', columns='month', values='revenue')
    print(data)

    # 按指定月份顺序重新排列列，缺失月份用 NaN 填充
    data = data.reindex(columns=months)
    print(data)

    # 重命名列：加上 _Revenue 后缀
    data.columns = [f"{m}_Revenue" for m in data.columns]
    print(data)

    # 重置索引，恢复 id 为列
    return data.reset_index()
```


## 5. 其他

### 排序
```python
# 单列升序排序
df_sorted = df.sort_values(by='age')

# 单列降序排序
df_sorted = df.sort_values(by='score', ascending=False)

# 多列排序：先按年龄升序，再按成绩降序
df_sorted = df.sort_values(by=['age', 'score'], ascending=[True, False])
```


### 过滤
```python
# 或者的关系，'|' 两侧的括号不能省
mg[(mg['bonus']<1000) | (mg['bonus'].isna())]
ans.loc[(ans['x'] + ans['y'] > ans['z']) & (ans['x'] + ans['z'] > ans['y']) & (ans['y'] + ans['z'] > ans['x']), 'triangle'] = 'Yes'
```

简单过滤某列的值，是否有重复元素，达到group by + having的效果

```python
cond1 = insurance['tiv_2015'].duplicated(keep=False)
// 按两列groupby过滤
cond2 = ~insurance[['lat','lon']].duplicated(keep=False)
df = insurance[cond1 & cond2]['tiv_2016']
```

not in 的对应写法

```python
# 实现 NOT IN
result = df1[~df1['A'].isin(df2['A'])]
```

### 删掉某列

```python
# 删除某列
df = df.drop(columns=['score'])  # 删除 'score' 列
# 或者
df = df.drop('score', axis=1)    # 删除 'score' 列
# 原地删除一列
result.drop(columns=['matched'], inplace=True)
```

### 堆叠两个表

concat 
```python
    df1 = request_accepted[['requester_id']].copy()
    df1.rename(columns={'requester_id': 'id'}, inplace=True)
    df2 = request_accepted[['accepter_id']].copy()
    df2.rename(columns={'accepter_id': 'id'}, inplace=True)
    df = pd.concat([df1, df2], ignore_index=True)
```

### 获取行号

```python
    df = df.reset_index()  # 增加 'index' 列记录行号
    df['t_rank'] = df['id']-df['index']

    # 或者下面的写法
    df['t_rank'] = df['id'] - df.index
```

### 删除重复行
**keep**：指定保留重复行的方式。
- 'first'（默认值）：保留重复行中的第一行。
- 'last'：保留重复行中的最后一行。
- False：删除所有重复行。
```python
df_drop = my_numbers.drop_duplicates(subset=['num'], keep=False)
```
### 调整列顺序
```python
df = df.reindex(columns=['C', 'A', 'B'])
```

### lambda 函数

修改某个字段
```python
salary['sex'] = salary['sex'].apply(lambda x: 'm' if x == 'f' else 'f')
```


### 列重命名

```python
ans.rename(columns={'activity_date': 'day'}, inplace=True)
```


### reset_index同时重命名新列

```python
ans = grouped['user_id'].nunique().reset_index(name='active_users')
```

### round函数的四舍五入
在Python中的round函数和NumPy中的np.round函数默认使用的是"四舍五入"规则。具体来说，当要舍入的数字是正好在5的一半上方时，它们将向最接近的偶数舍入。这种规则被称为"银行家舍入法"或"四舍六入五留双"，它旨在减少舍入误差的累积，从而提高精度。
如果我们想要使用不同的舍入规则，需要通过指定精度来使用decimal模块中的ROUND_HALF_UP等舍入模式，或者自定义舍入逻辑。
[参考](https://leetcode.cn/problems/queries-quality-and-percentage/solutions/1508376/by-zg104-1sdy)
