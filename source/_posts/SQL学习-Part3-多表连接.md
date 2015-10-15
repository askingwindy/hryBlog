title: "3.SQL学习：多表连接"
date: 2015-04-09 10:36:30
tags: [学习：SQL,程序员初级修炼]
categories: SQL
description: 主要学习了多表连接查询：`LEFT JOIN`、`RIGHT JOIN`、`INNER JOIN` 以及视图
---
#Pre：表
一张表就是一个集合，一行数据是集合的一个元素

* 理论上讲，不可能出现完全相同的两个行（所以union去重复了）
* 但是，表中可以存在完全的两行（表内部的rowid是不可重复的，这个rowid看不见）
* 集合相乘，就是笛卡尔积，两个集合的完全组合

* * *

#连接查询
1. 两个表连接在一起
2. 连接条件

##语法
###左连接(LEFT JOIN)
以左表为准，去右表找匹配数据
- 找不到用NULL补齐
- 表A LEFT JOIN 表B = 表B RIGHT JOIN 表A

```sql
SELECT  列1， 列2， 列N FROM
tableA LEFT JOIN tableB
ON tableA.列 = tableB.列 #此处表连接成为一张大表，完全当成普通表看
#后面可加 WHERE, GROUP BY, HAVING, ORDER BY, LIMIT
```
###右连接(RIGHT JOIN)
左连接：A站在B的左边，以A为准
右连接：B站在A的右边，以A为准
####如何记忆
1. 可以把右连接转换为左连接
2. 推荐把右连接转化为左连接（推荐使用左连接代替右连接）

```sql
SELECT  列1， 列2， 列N FROM
tableA RIGHT JOIN tableB
ON tableA.列 = tableB.列 
```
###内连接(INNER JOIN)
查询左右表都有的数据，即不要左/右中NULL的那一部分。
内连接是左右连接的交集
###外连接OUTER JOIN
- 查出左右连接的并集
- 可以用UNION达到目的（左连接UNION右连接）

```sql
SELECT  列1， 列2， 列N FROM
tableA INNER JOIN tableB
ON tableA.列 = tableB.列
```

- - -

##面试题
Match赛程表：

| 字段名称 | 字段类型 |描述|
|--------|--------|--|
|    matchId    |  int      |主键|
|hostTeamId|int|主队ID|
|guestTeamId|int|客队ID|
|matchResult|varchar(20)|比赛结果，如:(2:0)|
|matchTime|date|比赛开始时间|

Team参赛队伍表：

| 字段名称 |	字段类型	|	描述	|
|--------|--------|--|
|teamId|int|主键|
|teamName|varchar(10)|队伍名称|

Match的hostTeamID与Team中的teamId关联
查出2006-6-1到2007-7-1之间举行的所有比赛，并且用一下形式列出：

|hostTeamId |matchResult|	guestTeamId|matchTime|
|---|---|---|---|
|拜仁 |2:0|		不莱梅|		2006-06-21|

###创建表
```sql
#创建表并插入数据
CREATE TABLE m(
matchid int PRIMARY KEY AUTO_INCREMENT,
hid int,
gid int,
result varchar(20),
mtime date
);

INSERT INTO m
(hid, gid, result, mtime)
VALUES
(1,2,'2:0','2006-05-21'),
(2,3,'1:2','2006-06-21'),
(3,1,'2:5','2006-06-25'),
(2,1,'4:2','2006-07-21');

CREATE TABLE t(
tid int,
tname varchar(10)
);

INSERT INTO t
VALUES
(1, '国安'),
(2, '申花'),
(3, '全兴');
```
###解题步骤
将问题分解为小问题
####1：先弄对输出顺序
```sql
SELECT hid, result, gid, mtime FROM m;
```
####2：再取出主队名字
做连接
```sql
SELECT hid, tname AS hname, result, gid,mtime 
FROM
m LEFT JOIN t
ON m.hid = t.tid;
```
####3：再取出客队名字
再一次左连接
```sql
SELECT hid, t1.tname AS hname, result, gid, t2.tname AS gname, mtime 
FROM
m LEFT JOIN t AS t1
ON m.hid = t1.tid
#左连接两次
LEFT JOIN t AS t2
ON m.gid = t2.tid;
```
####4:限制时间
```sql
SELECT hid, t1.tname AS hname, result, gid, t2.tname AS gname, mtime 
FROM
m LEFT JOIN t AS t1
ON m.hid = t1.tid

LEFT JOIN t AS t2
ON m.gid = t2.tid

WHERE mtime 
BETWEEN '2006-06-01' AND '2006-07-01';
```
####5:输出格式更改
```sql
SELECT t1.tname AS hname, result, t2.tname AS gname, mtime 
FROM
m LEFT JOIN t AS t1
ON m.hid = t1.tid

LEFT JOIN t AS t2
ON m.gid = t2.tid

WHERE mtime 
BETWEEN '2006-06-01' AND '2006-07-01';
```
***
#视图
视图是由查询结果形成的一张虚拟表
##语法
###创建视图
```sql
CREATE VIEW 视图名
AS SELECT 语句
```
###删除视图
```sql
DRIP VIEW 视图名
```
###修改视图
```sql
ALTER VIEW 视图名 
AS SELECT 语句
```
##视图的作用
1. 简化查询
2. 进行权限控制
	- 把表的权限封闭，但是开放相应的视图权限，视图只开放部分数据
3. 大数据分表时用到
	- 表的行数超过200w，速度很慢==>把1张表的数据拆成4张表来存放

##视图的algorithm
```sql
CREATE ALGORITHM = 规则 VIEW 视图名 ...
```
### `MERGE`
当引用视图时，引用视图的语句与定义视图的语句合并
- 当查询视图时，把查询视图的语句（`WHERE`那些）与创建视图的语句合并，分析形成一条新的`SELECT`语句

### `TEMPTABLE`
当引用视图时，根据视图的创建语句建立一张临时表
然后查询视图的语句==从这张临时表查看数据
###`UNDEFINED`
未定义，系统选定
##视图与表的关系
视图是表的查询结果，所以表的数据改变会影响视图的结果
###如果视图改变了呢？
1. 视图的增删改也会影响表
2. 但是视图并不总是能增删改：只有视图的数据与表的数据一一对应时才可以修改
3. 对于`INSERT`还需要注意：视图必须包含表中没有默认值的列