# count

```sql
| id | departmentId |
| -- | ------------ |
| 1  | 1            |
| 2  | 1            |
| 3  | 2            |
| 4  | null         |
| 5  | 1            |
```

1、count只计算表达式内列的数量，非行记录数量；count(null)不计数；

- count(departmentId)=4

- count(*)=5

[580. 统计各专业学生人数](https://leetcode.cn/problems/count-student-number-in-departments/?envType=study-plan&id=sql-basic&plan=sql&plan_progress=1g3dies)

2、如果需要计数null为0，作为结果，使用：

```sql
ifnull(count(departmentId ),0)
```

3、count(express)

不能用：count(if(score>90,1,0))，0也会计数，只有null不计数；

所以用：

- count(if(score>90,1,null))

- count(score>90 OR null)

# group by

1、group by 配和聚合函数 max、min

**分组并取另一列的极值问题**：

- [1164. 指定日期的产品价格](https://leetcode.cn/problems/product-price-at-a-given-date/description/?envType=study-plan&id=sql-basic&plan=sql&plan_progress=1g3dies)
- [1988. 找出每所学校的最低分数要求](https://leetcode.cn/problems/find-cutoff-score-for-each-school/description/?envType=study-plan&id=sql-basic&plan=sql&plan_progress=1g3dies)

# union/union all

将多张表查询到的数据直接叠加，每个表查询的列数要相同；

union：叠加<mark>并去重</mark>；

union all：叠加后，不去重；

用处：可以将多列数据，合并到一列；union不考虑列之间的关系，直接叠加；

- [1783. 大满贯数量](https://leetcode.cn/problems/grand-slam-titles/description/?envType=study-plan&id=sql-basic&plan=sql&plan_progress=1g3dies)

# 笛卡尔积

t1表每一条记录都和t2表全部记录join一次

1、t1 cross join t2

2、select * from t1, t2

[1280. 学生们参加各科测试的次数](https://leetcode.cn/problems/students-and-examinations/description/?envType=study-plan&id=sql-basic&plan=sql&plan_progress=1g3dies)

# where/having

1、作用时机：`where`在结果返回之前过滤，`having`是返回结果集后再次过滤；

2、`having`后只能跟聚合函数；

3、`where`后面不能使用聚合函数；

- [leetcode 1084. 销售分析III](https://leetcode.cn/problems/sales-analysis-iii/description/?envType=study-plan&id=sql-beginner&plan=sql&plan_progress=12pggev)

- [1398. 购买了产品 A 和产品 B 却没有购买产品 C 的顾客](https://leetcode.cn/problems/customers-who-bought-products-a-and-b-but-not-c/description/?envType=study-plan&id=sql-basic&plan=sql&plan_progress=1g3dies)

# if/case when

两值：if

```sql
if(exp, value1, value2)
```

多值：when

```sql
case 
    when exp then value1
    when exp then value2
else value3 end 别名
```

- [1440. 计算布尔表达式的值](https://leetcode.cn/problems/evaluate-boolean-expression/description/?envType=study-plan&id=sql-basic&plan=sql&plan_progress=1g3dies)
- [1294. 不同国家的天气类型](https://leetcode.cn/problems/weather-type-in-each-country/description/?envType=study-plan&id=sql-basic&plan=sql&plan_progress=1g3dies)

# 需要空值返回NULL

1、IFNULL函数

```sql
select ifnull(value,null)
```

2、IF(exp, value, null)

3、包一层子查询，空值自动转null

# using

join条件 on 的表的列名相同，可用using替换：

```sql
t1 join t2 on t1.id = t2.id and t1.name = t2.name
t1 join t2 using(id,name)
```
