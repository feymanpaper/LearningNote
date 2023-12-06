left join 
查出来的结果显示左边的所有数据，然后右边显示的是和左边有交集部分的数据

right join
查出表2所有数据以及表1和表2有交集的数据

join
inner join，表示以两个表的交集

聚合函数（count），where字句无法与聚合函数一起使用。因为where子句的运行顺序排在第二，运行到where时，表还没有被分组

182. 查找重复的电子邮箱
```sql
select 列名
from 表名
group by 列名
having count(列名) > n;
```

619. 只出现一次的最大数字
查询结果为空时，如何返回空值
https://leetcode.cn/problems/biggest-single-number/solutions/683252/dang-biao-ge-wei-kong-shi-ru-he-fan-hui-6qpzg/

```
-可以使用聚合函数进行空值null值的转换，具体的聚合函数包括SUM/AVG/MAX/MIN
-可以使用select语句进行转换，但空值应直接写在select中而非from中
-limit语句无法出现新的null值
-where和having同样无法出现新的null值
```

176. 第二高的薪水

```sql
SELECT(SELECT MAX(salary)
FROM Employee
WHERE salary < (SELECT MAX(DISTINCT salary) FROM Employee))
AS SecondHighestSalary
```
举一反三, 如何求第N高的

limit n子句表示查询结果返回前n条数据
offset n表示跳过x条语句
limit y offset x 分句表示查询结果跳过 x 条数据，读取前 y 条数据
```sql
SELECT IFNULL(
(SELECT DISTINCT salary
FROM Employee
ORDER BY salary DESC
LIMIT 1,1), NULL) AS SecondHighestSalary
```
