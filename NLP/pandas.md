

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
```

how的取值还可以有 left/right

如果是两个关联字段
```python
# 按 EmployeeID 和 Year 两个字段进行关联
result = df1.merge(df2, on=['EmployeeID', 'Year'], how='inner')

# 按 EmployeeID 和 Year 两个字段进行关联，字段名不同
result = df1.merge(df2, left_on=['EmployeeID', 'Year'], right_on=['ID', 'Year'], how='inner')
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

## 4. 其他

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