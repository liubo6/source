---
title: SQL语句执行顺序
date: 2016-11-22 18:03:24
categories: SQL
tags: SQL
---
# sql语句的逻辑执行顺序,顺序序列
SELECT语句的**实际物理执行顺序**可能会由于查询处理器的不同而与这个顺序有所出入
由于 SQL 语句语法顺序和执行顺序的不同，很多人会认为SELECT 中的字段信息是 SQL 语句的核心。其实真正的核心在于对表的引用


1. FROM <left_table>
2. ON <join_condition>
3. JOIN <join_type> JOIN <right_table>
4. WHERE <where_condition>
5. GROUP BY <group_by_list>
6. WITH <CUBE or WITH ROLLUP>
7. HAVING <having_condition>
8. SELECT
9. DISTINCT
10. ORDER BY <order_by_list>
11. TOP <TOP_specification>

<!--more-->
实例解析
```
(8)SELECT (9)DISTINCT
(11)<TOP_specification> <select_list>
(1)FROM <left_table>
(3)　<join_type> JOIN <right_table>
(2)　 ON <join_condition>
(4)WHERE <where_condition>
(5)GROUP BY <group_by_list>
(6)WITH {CUBE | ROLLUP}
(7)HAVING <having_condition>
(10)ORDER BY <order_by_list>
```
每个步骤产生一个虚拟表，该虚拟表被用作下一个步骤的输入。只有最后一步生成的表返回给调用者
如果没有某一子句，则跳过相应的步骤
```
1. FROM:对FROM子句中的前两个表执行笛卡尔积，生成虚拟表VT1。
 
2. ON:对VT1应用ON筛选器。只有那些使<join_condition>为真的行才被插入VT2。
 
3. OUTER(JOIN):如果指定了OUTER JOIN，保留表中未找到匹配的行将作为外部行添加到VT2，生成VT3。
 
如果FROM子句包含两个以上的表，则对上一个联接生成的结果表和下一个表重复执行步骤1到步骤3，直到
 
处理完所有的表为止。
 
4. 对VT3应用WHERE筛选器。只有使<where_condition>为TRUE的行才被插入VT4。
 
5. GROUP BY:按GROUP BY 子句中的列列表对VT4中的行分组，生成VT5。
 
6. CUBE|ROLLUP:把超组插入VT5，生成VT6。
 
7. HAVING:对VT6应用HAVING筛选器。只有使<having_condition>为TRUE的组才会被插入VT7。
 
8. SELECT:处理SELECT列表，产生VT8。
 
9. DISTINCT:将重复的行从VT8中移除，产生VT9。
 
10. ORDER BY:将VT9中的行按ORDER BY子句中的列列表排序，生成一个有表(VC10)。
 
11. TOP:从VC10的开始处选择指定数量或比例的行，生成表VT11,并返回给调用者。
```


# select
select子句--少用*号，尽量取字段名称
在解析的过程中, 会将依次转换成所有的列名, 这个工作是通过查询数据字典完成的, 使用列名意味着将减少消耗时间

# from
from 子句--执行顺序为从后往前、从右到左
表名(最后面的那个表名为驱动表，执行顺序为从后往前, 所以数据量较少的表尽量放后）


# where
where子句--执行顺序为自下而上、从右到左
以过滤掉最大数量记录的条件必须写在Where 子句的末尾

# group by
group by--执行顺序从左往右分组
提高GROUP BY 语句的效率, 可以通过将不需要的记录在GROUP BY 之前过滤掉。
即在GROUP BY前使用WHERE来过虑，而尽量避免GROUP BY后再HAVING过滤。

# having
having 子句----很耗资源，尽量少用
避免使用HAVING 子句, HAVING 只会在检索出所有记录之后才对结果集进行过滤. 这个处理需要排序,总计等操作
on、where、having 这三个都可以加条件的子句中，on 是最先执行，where 次之，having 最后

# order by
order by子句--执行顺序为从左到右排序,很耗资源


# SQL联合（join）
虽然已经不推荐使用连表操作，还是可以了解了解的
假设我们有两张表A左表，B右表
```
A         B
id  name  id  name
--- ----  --- ----
1  liubo  1  hello
2  hello  2  liubo
3  world  3  welcome
```
## 内联合（inner join）
只生成同时匹配表A和表B的记录集
```
mysql> select * from a inner join b on a.name = b.name;
+----+-------+----+-------+
| id | name  | id | name  |
+----+-------+----+-------+
|  2 | hello |  1 | hello |
|  1 | liubo |  2 | liubo |
+----+-------+----+-------+
```
![](http://ww1.sinaimg.cn/mw690/69045600gw1fa0vr8cyl4j20dy0953yd.jpg)

## 全外联合（full outer join）
生成表A和表B里的记录全集，包括两边都匹配的记录。如果有一边没有匹配的，缺失的这一边为null

![](http://ww3.sinaimg.cn/mw690/69045600gw1fa0vr7ngq5j20dy095mwz.jpg)

## 左外联合（left outer join）
生成表A的所有记录，包括在表B里匹配的记录。如果没有匹配的，右边将是null。
```
mysql> select * from a left outer join b on a.name =b.name;
+----+-------+------+-------+
| id | name  | id   | name  |
+----+-------+------+-------+
|  2 | hello |    1 | hello |
|  1 | liubo |    2 | liubo |
|  3 | world | NULL | NULL  |
+----+-------+------+-------+
3 rows in set
```
![](http://ww3.sinaimg.cn/mw690/69045600gw1fa0vr96iojj20dy095wec.jpg)
outer 可以省略

为了生成只在表A里而不在表B里的记录集，我们用同样的左外联合，然后用where语句排除我们不想要的记录
```
mysql> select * from a left outer join b on a.name =b.name where b.name is null;
+----+-------+------+------+
| id | name  | id   | name |
+----+-------+------+------+
|  3 | world | NULL | NULL |
+----+-------+------+------+
1 row in set
```
![](http://ww4.sinaimg.cn/mw690/69045600gw1fa0vrb4l8sj20dy095dfp.jpg)

## 总览
![](http://ww2.sinaimg.cn/mw690/69045600gw1fa0vr9w8gzj20qu0l4dic.jpg)



