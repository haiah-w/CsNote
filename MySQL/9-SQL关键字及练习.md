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
case when 
    exp then value1
    exp then value2
    else value3
end value
```

- [1440. 计算布尔表达式的值](https://leetcode.cn/problems/evaluate-boolean-expression/description/?envType=study-plan&id=sql-basic&plan=sql&plan_progress=1g3dies)

# 需要空值返回NULL



1、IFNULL函数

```sql
select ifnull(value,null)
```

2、IF(exp, value, null)

3、包一层子查询，空值自动转null


