title: "2.SQL学习：SQL基本操作"
date: 2015-04-05 21:36:30
tags: [学习：SQL,程序员初级修炼]
categories: SQL
description: SQL表的建立、数据的增删改查以及列的增删改，重点学习了查询关键字，包括了普通的WHERE, GROUP BY,  HAVING,ORDER BY, LIMIT，以及子查询
---
#对表的操作
##建表
###语法
```sql
CREATE TABLE 表名(
列名称 列类型 [列属性] [默认值],#[]里的内容表示可选
...
);
```
###实例
```sql
CREATE TABLE class1(
id INT PRIMARY KEY AUTO_INCREMENT,
name CHAR(6) NOT NULL DEFAULT '',
age TINYINT UNSIGNED NOT NULL DEFAULT 0,
email VARCHAR(30) NOT NULL DEFAULT '',
salary DECIMAL(5,2) NOT NULL DEFAULT 0.00,
enroll DATE NOT NULL DEFAULT '2000-03-01'
)CHARSET UTF8;
```
##列管理
###增加列
####语法
```sql
ALTER TABLE 表名
ADD 列名 列属性
#AFTER 列名1 #将新增列放在列1的后面
#FIRST #将新增列放在最前面
```
###修改列
####语法
```sql
ALTER TABLE 表名
CHANGE 被改变的列声明 现在的列声明（列声明：列名+列属性）
```
###删除列
####语法
```sql
ALTER TABLE 表名
DROP 列名
```
##行操作
###增加行
三要素：
- 往哪张表添加？
- 给哪几列添加？
- 分别添加什么值？
####语法
```sql
INSERT INTO 表名
(列1, 列2, ..., 列N)
VALUES
(值1，值2,, ..., 值N)
```
如果不写列，默认插入所有列(所以要写所有列的值)
###修改行的值
改哪张表？需要改哪几列的值？分别改什么值？在哪些行生效？
```sql
UPDATE 表名
SET
列1 = 值1，
列2 = 值2，
...
列N = 值N
# WHERE 表达式 #不写这个默认所有行
```
###删除行
只能删除整行，而不是一行中某几列的值
```sql
DELETE 表名
WHERE 表达式
```
***
#查询
`SELECT`有5个字句（如果写了多种，关键字的顺序必须固定如下：`WHERE` ==> `GROUP BY`==>`HAVING`==>`ORDER BY`==>`LIMIT`）
TIPS：
可以用括号`()`提高关键字的优先级
## `WHERE`
判断每一行的表达式是否成立
###比较运算符
####普通的符号
```sql
<,<=
=,!=
>, >=
```
####`IN`在某个集合内
```sql
IN (值1, 值2, ..., 值N) #表示取出值=值，值=值2,...值=值N得所有行
```
####`BETWEEN`在某个范围内
```sql
BETWEEN 值1 AND 值2 #表示取出满足条件：值1 <= 值 <= 值2的所有行
```
###逻辑运算符
- `NOT`:不属于某个条件的行
- 'OR'
- 'AND'
###模糊查询
####`LIKE`
- `%`：通配任意长度的字符
- `_`：通配单个字符
##`GROUP BY`
把行按**字段**分组
###语法
```sql
GROUP BY 列, 列2, ..., 列N
```
###运用场合
常见于统计场合：统计平均成绩、最高分等
####统计函数
- `MAX`
```sql
 SELECT goods_id, max(shop_price) FROM test
 #这样写是没有意义的，这样goods_id是第一个遇见的id，而不是最大价格的goods_id
```
- `MIN`
- `SUM`
- `AVG`
- `COUNT`

将列名当做变量名来看待，同时，可以给计算结果利用`AS`取别名
##`HAVING`
- `WHERE`是对表起作用，不能对查询结果（虽然结构和表一致）起作用
- `HAVING`是继续对查询结果继续查询
##`ORDER BY`
默认是根据字段升序排序（`ASC`显式声明）
- 最后加`DESC`改为降序排列
###根据多个字段进行排序
```sql
ORDER BY 列1， 列2 ...
```
##`LIMIT`
放在语句最后，起到限制条目的作用
```sql
LIMIT [OFFSET], [N]
```
- `OFFSET`:偏移量（结果不包含该值）
- `N`：取出的条目个数
`OFFSET`不写，相当于从第1行开始取知道取出N个数据为止
# 子查询
逻辑上构建一张"临时表"(这张表不存在)==>将查询结果当做这张"临时表"==>在这张临时表上进行子查询
##WHERE型子查询 
 把内层查询的结果当做外层查询的**比较条件**
###查询每个栏目下最贵商品
```sql
SELECT goods_name, cat_id,shop_price FROM test 
WHERE shop_price IN 
(SELECT MAX(shop_price) FROM test GROUP BY cat_id);
```
##FROM型子查询
把内层查询结果当做**临时表**，供外层再次查询
##EXISTS子查询
把外层查询结果拿到内层，看内层查询是否成立
#良好的理解模型
- `WHERE`表达式：把表达式放在行中，看表达式是否为真
- 列：理解成变量，可以晕死
- 取出结果：可以理解成一张临时表
***
#练习题
有如下的表以及数据:
|name|subject|score|
|---|---|---|
|张三|数学|90|
|张三|语文|50|
|张三|地理|40|
|李四|语文|55|
|李四|政治|45|
|王五|政治|30|
要求：查出2门及两门以上不及格者的平均成绩
##只利用`WHERE-HAVING-GROUP`
将大问题分解为小问题一步步来：
###1.查出每个人的平均分
```sql
SELECT name, avg(score)
FROM tableScoreGROUP BY name;
```
###2.计算每个人挂科总数目
```sql
SELECT name, sum(score<60)
FROM tableScore GROUP BY name;
```
###3.在2的基础上找出每个人的平均分
```sql
SELECT name, sum(score<60), avg(score)
FROM tableScore GROUP BY name;
```
###4.在3的基础上找出挂科数目>=2的人
```sql
SELECT name, sum(score<60) as he, avg(score)
FROM tableScore GROUP BY name 
HAVING he >=2;
```
##利用子查询
```sql
SELECT name, avg(score) FROM tableScore
WHERE name IN
(SELECT name FROM
(SELECT name, sum(score<60) as co FROM tableScore 
GROUP BY name HAVING co>=2) 
AS tmp) 
GROUP BY name;
```
##前期准备
###建表
```sql
CREATE TABLE tableScore(
name VARCHAR(4),
subject VARCHAR(4),
score TINYINT
);
```
###插入数据
```sql
INSERT INTO tableScore
(name, subject, score)
VALUES
('张三','数学',90),
('张三','语文',50),
('张三','地理',40),
('李四','语文',55),
('李四','政治',45),
('王二','政治',30);

#增加额外的数据
INSERT INTO tableScore
(name, subject, score)
VALUES
('赵六','A',100),
('赵六','A',80),
('赵六','A',55);
```