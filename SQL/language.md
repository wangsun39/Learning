

## 1. order by 用法

取第一行
```sql
--oracle
select customer_number from (select customer_number, count(1) as cnt from Orders group by customer_number order by cnt desc) where ROWNUM = 1;

--pg or Mysql 可以用limit
select customer_number from (select customer_number, count(1) as cnt from Orders group by customer_number order by cnt desc limit 1) a;
```

取某一行可以加offset

```sql
--oracle
SELECT *
FROM (
    SELECT *
    FROM (
        SELECT *
        FROM students
        ORDER BY score DESC
    )
    WHERE ROWNUM <= 2
)
WHERE ROWNUM = 1;

--pg or Mysql 可以用limit
SELECT *
FROM students
ORDER BY score DESC
LIMIT 1 OFFSET 1;
```


## 2. 窗口函数

### ROW_NUMBER()
为每一行分配一个唯一的序号，序号在每个分区中从 1 开始递增。

```sql
SELECT id, name, salary,
       ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) AS row_num
FROM employees;
```
### RANK() 和 DENSE_RANK()
- RANK()：为每一行分配一个排名，如果存在并列值，则会跳过一些排名。
- DENSE_RANK()：为每一行分配一个排名，即使存在并列值，排名也不会跳过。

```sql
SELECT id, name, salary,
       RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS rank_num,
       DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS dense_rank_num
FROM employees;
```

### 聚合函数（SUM()、AVG()、MIN()、MAX() 等）
这些函数可以在窗口函数中使用，计算分区内的聚合值，同时保留每一行的原始数据。
```sql
SELECT id, name, salary,
       SUM(salary) OVER (PARTITION BY department_id) AS total_salary,
       AVG(salary) OVER (PARTITION BY department_id) AS avg_salary
FROM employees;
```

### LEAD() 和 LAG()
- LEAD(column, n)：获取当前行之后第 n 行的值。
- LAG(column, n)：获取当前行之前第 n 行的值。
```sql
SELECT id, name, salary,
       LEAD(salary, 1) OVER (ORDER BY salary) AS next_salary,
       LAG(salary, 1) OVER (ORDER BY salary) AS prev_salary
FROM employees;
```

## 3. case when
```sql
select id,
    case when p_id is null then 'Root'
         when id in (select p_id from tree) then 'Inner'
    else 'Leaf' end type
from tree;
```


## 4. 其他


### null 值映射
```
-- oracle
NVL(employee_id, 'No Employee')
```

### Postgres

除法的四舍五入写法，不能直接使用 $round$ 函数
```sql
round((cast(cnt as numeric)-comp)/cnt, 2) as "Cancellation Rate"
```
或者
```sql
round((cnt-comp)/cnt::NUMERIC, 2) as "Cancellation Rate"
```

其他 sql 可以直接写成
```sql
round((cnt-comp)/cnt, 2) as "Cancellation Rate"
```

日期+1的写法
```sql
event_date + INTERVAL '1 day'
```

### Oracle

引入 Dual 表，一个虚拟表
可以将两个select出来的数字直接做运算，如

```sql
SELECT round(
    (SELECT count(*)
    FROM activity a, 
        (SELECT player_id,
         min(event_date) AS first_day,
         min(event_date) + 1 AS second_day
        FROM activity
        GROUP BY  player_id) b
        WHERE a.player_id=b.player_id
                AND a.event_date =b.second_day)/
    (SELECT COUNT(DISTINCT player_id)
    FROM activity), 2) AS fraction
FROM dual;
```
对于Postgres和Mysql都不需要用dual，select出来的单个数值型结果可以直接运算

### Mysql

Data类型+1
```sql
DATE_ADD(MIN(event_date), INTERVAL 1 DAY)
```
