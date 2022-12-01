# Tips

1、`count`、`sum`一般同`group by`一起使用

- [Leetcode 1158. 市场分析 I](https://leetcode.cn/problems/market-analysis-i/description/?envType=study-plan&id=sql-beginner&plan=sql&plan_progress=12pggev)

- [Leetcode 1407. 排名靠前的旅行者](https://leetcode.cn/problems/top-travellers/description/?envType=study-plan&id=sql-beginner&plan=sql&plan_progress=12pggev)

2、按多列分组：`group by date, name`

- [leetcode **1693**](https://leetcode.cn/problems/daily-leads-and-partners/description/?envType=study-plan&id=sql-beginner&plan=sql&plan_progress=12pggev)

3、时间函数`dateDiff`的使用

- [leetcode **1141**](https://leetcode.cn/problems/user-activity-for-the-past-30-days-i/?envType=study-plan&id=sql-beginner&plan=sql&plan_progress=12pggev)

4、`where`后面不能使用聚合函数(`group by`)做过滤，此时用`having`

having后只能跟聚合函数

- [leetcode 1084. 销售分析III](https://leetcode.cn/problems/sales-analysis-iii/description/?envType=study-plan&id=sql-beginner&plan=sql&plan_progress=12pggev)

5、四舍五入：round(num, digit)

- round(1.34, 1) = 1.3

6、可以在select后面直接插入子查询

- [1633. 各赛事的用户注册率](https://leetcode.cn/problems/percentage-of-users-attended-a-contest/description/?envType=study-plan&id=sql-basic&plan=sql&plan_progress=1g3dies)

7、时间取年-月，可以直接left截取：left(2019-12-12,7) -> 2019-12

- [1193. 每月交易 I](https://leetcode.cn/problems/monthly-transactions-i/description/?envType=study-plan&id=sql-basic&plan=sql&plan_progress=1g3dies)

# 类别

1、分类统计表中重复记录、某列和：`group by`、`having`、`count`、`sum`

- [182. 查找重复的电子邮箱](https://leetcode.cn/problems/duplicate-emails/description/?envType=study-plan&id=sql-beginner&plan=sql&plan_progress=12pggev)

- [1050. 合作过至少三次的演员和导演](https://leetcode.cn/problems/actors-and-directors-who-cooperated-at-least-three-times/description/?envType=study-plan&id=sql-beginner&plan=sql&plan_progress=12pggev)

- [1587. 银行账户概要 II](https://leetcode.cn/problems/bank-account-summary-ii/description/?envType=study-plan&id=sql-beginner&plan=sql&plan_progress=12pggev)

2、统计比例相关题目：

通常会用到：avg(求比例，小数)、group by(分组)、round(四舍五入)

- [1211. 查询结果的质量和占比](https://leetcode.cn/problems/queries-quality-and-percentage/description/?envType=study-plan&id=sql-basic&plan=sql&plan_progress=1g3dies)

- [1173. 即时食物配送 I](https://leetcode.cn/problems/immediate-food-delivery-i/description/?envType=study-plan&id=sql-basic&plan=sql&plan_progress=1g3dies)

- [1633. 各赛事的用户注册率](https://leetcode.cn/problems/percentage-of-users-attended-a-contest/description/?envType=study-plan&id=sql-basic&plan=sql&plan_progress=1g3dies)

# 子查询相关题目

子查询：即返回一张临时表，在子查询返回的临时表上再进行查询；

[1303. 求团队人数 - 力扣（Leetcode）](https://leetcode.cn/problems/find-the-team-size/?envType=study-plan&id=sql-basic&plan=sql&plan_progress=1g3dies)

[1264. 页面推荐 - 力扣（Leetcode）](https://leetcode.cn/problems/page-recommendations/?envType=study-plan&id=sql-basic&plan=sql&plan_progress=1g3dies)

自连接：

[570. 至少有5名直接下属的经理 - 力扣（Leetcode）](https://leetcode.cn/problems/managers-with-at-least-5-direct-reports/description/?envType=study-plan&id=sql-basic&plan=sql&plan_progress=1g3dies)
