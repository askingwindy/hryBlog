title: "4.SQL学习：SQL高级语法"
date: 2015-04-10 16:36:20
tags: [学习：SQL,程序员初级修炼]
categories: SQL
description: 主要学习了：字符集、校对集；触发器；事务；索引以及存储过程
---
#字符集
Mysql的字符集设置非常灵活，可以设置服务器默认的字符集
- 数据库默认字符集
- 表默认字符集
- 列字符集
- 某个级别没有指定的话，直接继承上一级

##以表声明为UTF8为例
```sql
CREATE TABLE a(
id int
)CHARSET UTF8;
```
- 存储在数据表中的数据最终是UTF8

### 客户端与服务器
客户端是GBK编码，服务器是UTF8编码
![服务器与客户端字符集](http://askingwindy-gitcafe.qiniudn.com/SQL字符集.png)
#### SQL语句字符集
客户端(GBK) ===> 字符集链接器(GBK(来自客户端)->GBK(连接器)->UTF8(发给服务器))===>服务器（UTF8
1. 告诉服务器，我给你发送的数据是什么编码
```sql
SET CHARACTER_SET_CLIENT = GBK;
```
2. 告诉连接器，转化成什么编码
```sql
SET CHARACTER_SET_CONNECTION = GBK;#指定的是连接器中中间的那个GBK
```
3. 查询结果是什么编码
```sql
SET CHARACTER_SET_RESULTS = GBK;
```

### 什么时候出现乱码
1. 客户端声明的字符集与事实不符
2. 查询结果字符集与客户端页面不符合

#校对集
就是排队规则，一个字符集可以有一个或多个排序规则
##以UTF8为例
默认使用`UTF8-GENERAL-CI`
###如何声明校对集
```sql
CREATE TABLE T(
id int
)CHARSET UTF8 COLLATE UTF8_BIN; #按二进制排序
```
###查看校对集
```sql
SHOW COLLATION
```
***
#触发器
作用：监视某种情况，并触发某种操作
- 能触发的操作：增、删、改
- 能监视的操作：增、删、改

应用场合：向一张表添加、删除时，需要在相关的表中进行同步
##触发器创建四要素
| 名字 | 语法 |
|--------|--------|
|   监视地点     |    TABLE    |
|监视事件|INSERT/UPDATE/DELETE|
|触发时间|AFTER/BEFORE|
|触发事件|INSERT/UPDATE/DELETE|

### 创建触发器语法
```sql
CREATE TRIGGER 触发器名字
AFTER/BEFORE INSERT/UPDATE/DELETE ON 表名
FOR EACH ROW #固定语句
BEGIN
sql语句（限于增删改），一句或多句;
END ||
```
###删除触发器语法
```sql
DROP TRIGGER 触发器名字;
```
###查看触发器
```sql
SHOW TRIGGERS;
```
##实例：订单与库存管理
触发时间：`AFTER`
###准备工作
```sql
#建立商品表
CREATE TABLE g(
id int,
name varchar(10),
num int
)CHARSET UTF8;
#建立订单表
CREATE TABLE o(
oid int,
gid int,
much int
)CHARSET UTF8;

INSERT INTO g
VALUES
(1, '猪', 22),
(2,'羊', 19),
(3,'狗', 12),
(4,'猫', 8);

INSERT INTO o
VALUES
(1,2,3);

#手动更新
UPDATE g SET num = num-3
WHERE id = 2;
```

###增加一个订单
要求：增加订单，库存数量相应变化

- 监视地点：o表
- 监视操作：INSERT
- 触发操作：UPDATE
- 触发时间：AFTER

####建立简单触发器
每一次只能对id=2的num进行操作
```sql
#改变结束符
DELIMITER || #结束符为||,注意后面没有分号

#建立触发器
CREATE TRIGGER tg1
AFTER INSERT ON o
FOR EACH ROW
BEGIN
UPDATE g SET NUM = NUM-3 WHERE id = 2;
END ||
```
####如何在触发器引用行的值
- 对于`INSERT`而言，新增的行用`NEW`来表示
	- 行中每一列的值，用`NEW.列名`来表示
- 对于`DELETE`而言，删除的行的值用`OLD.列名`来表示
- 对于`UPDATE`而言，修改前的行中的值用`OLD.列名`来引用，而修改后的数据用`NEW.列名`来引用

####简单触发器升级版

```sql
CREATE TRIGGER tg2
AFTER INSERT ON o
FOR EACH ROW
BEGIN
UPDATE g SET NUM = NUM - NEW.much WHERE id = NEW.gid;
END ||
```
### 删除一个订单

```sql
CREATE TRIGGER tg3
AFTER DELETE ON o
FOR EACH ROW 
BEGIN
UPDATE g SET NUM = NUM + OLD.much WHERE id = OLD.gid;
END ||
```
### 更新一个订单
```sql
CREATE TRIGGER tg4
AFTER UPDATE ON o
FOR EACH ROW
BEGIN
UPDATE g SET NUM = NUM + OLD.much WHERE id = OLD.gid;
UPDATE g SET NUM = NUM - NEW.much WHERE id = NEW.gid;
END ||
```
##`AFTER`和`BEFORE`区别
- `AFTER`:先完成数据的增删改，在触发
	- 触发语句晚于增删改，无法影响前面的增删改
- `BEFORE`：先完成触发，再完成增删改
	- 触发语句先于监视的增删改发生  

### 典型案例
对于所下订单进行判断，如果订单数量>5， 认为是恶意订单，强制把所定商品数量改成5
#### 触发器四要素
1. 监视地点: o
2. 监视事件：INSERT
3. 触发事件：UPDATE
4. 触发事件: BEFORE

```sql
CREATE TRIGGER tg5
BEFORE INSERT ON o
FOR EACH ROW
BEGIN
    IF NEW.much > 5 THEN
        SET NEW.much = 5;
    END IF;
    UPDATE g SET NUM = NUM - NEW.much WHERE id = NEW.gid;
END ||
```
***
# 事务
## 存储引擎
对于用户而言，同样一张表无论用什么引擎来存储，用户看到的数据是一样的，但是对于**服务器**而言有区别
```sql
CREATE TABLE account(
id int
mondy int
)ENGINE INNODB CHARSET UTF8;
```
### 常用表的引擎
|特点| MYISAM | INNODB |
|--------|--------|--------|
|  批量插入的速度|高      | 低       |
|事物安全| |支持|
|全文索引|支持| 支持|
|锁机制|表锁|行锁|
|B树索引|支持|支持
|哈希表索引||支持
##事务的引擎
INNODB或者BDB（使用不多）
##事务的ACID特性
- 通俗的说事物，指一组操作，要么都成功执行，要么都不执行 ==> **原子性**
- 在所有操作没有执行完毕之前，其他会话不能看到中间改变的过程 ==> **隔离性**
- 事务发生前后，数据总额匹配 ===> **一致性**
- 事物产生的影响不能够撤销 ===> **持久性**
	- 如果出了错误，事物不允许撤销，只能通过“补偿性事务” 

##事务的语法
在`COMMIT`前，整个事务是隔离的，其他窗口无法见到操作结果
```sql
#开启事务
START TRANSACTION;
#SQL语句操作
...
#提交事务
COMMIT;#提交
#或者：ROLLBACK; #回滚
```
### 显式提交事务
在一个事务`COMMIT`或者`ROLLBACK`后，事务结束。下次使用需要再次开启事务。
有些语句会默认提交事务，例如：`START TRANSACTION;`
![事务与普通语句的比较](http://askingwindy-gitcafe.qiniudn.com/事务.png)

***
# 索引
索引是针对数据建立的目录
作用：加快查询速度
坏处：
- 降低增删改的速度
- 增大了表的文件大小（索引文件甚至可能比数据文件还大）

##案例
设新闻表有15列10行，共500W行数据，如何快速导入？
1. 空白索引全部删除
2. 导入数据
3. 数据导入完毕后，集中建立索引

## 索引原则
1. 不过度索引
2. 建立索引条件列:
	- `WHERE`后面最频繁的条件比较适宜索引
3. 索引散列值，过于集中的值不要索引（没有意义）

## 索引类型
| 索引名    | 普通索引   |  唯一索引 |  主键索引 | 全文索引  |
|----------|----------|----------|----------|----------|
|作用       |   仅是加快查询速度   |   行上的值不能重复     |主键必唯一，但是唯一索引不一定是主键| 在mysql默认情况下，对中文意义不大|
|声明语法|`INDEX`|`UNIQUE`|`PRIMARY KEY`|`FULLTEXT`|
|默认类型|BTREE|BTREE|BTREE|FULLTEXT|

- 一张表上只能有一个主键，但是可以有一个或多个唯一索引

###FULLTEXT
全文索引对中文意义不大
因为英文中有空格、标点符号来拆单词，从而对单词进行索引
而中文没有空格来隔开单词，mysql无法识别每个中文词
####用法
``` sql
SELECT * FROM MATCH (索引名) AGAINSET ('索引词');
```
####全文索引停止词
全文索引不针对非常频繁的词做索引，如：this, is, you等
## 索引语句
###查看一张表的索引
```sql
SHOW INDEX FROM 表名
```
###建立索引
```sql
ALTER TABLE 表名 ADD INDEX/UNIQUE/FULLTEXT [索引名](列名) #索引名可选
/PRIMARY KEY #主键值不需要索引名的
```

```sql
CREATE TABLE member(
id int,
email varchar(20),
tel char(11),
intro text
)ENGINE MYISAM CHARSET UTF8;

#给tel列加一个普通的索引
ALTER TABLE member ADD INDEX tel(tel);
#给email加一个唯一索引
ALTER TABLE member ADD UNIQUE (email);
#给intro加一个全文索引
ALTER TABLE member ADD FULLTEXT (intro);
#给id加一个主键索引
ALTER TABLE MEMBER ADD PRIMARY KEY (id);
```
###删除索引
```sql
#删除非主键索引
ALTER TABLE 表名 DROP INDEX 索引名;
#删除主键索引
ALTER TABLE 表名 DROP PRIMARY KEY;
```
***
#存储过程
概念类似函数，就是把一段代码封装起来
当执行这一段代码的时候，可以调用该存储过程来执行。

在封装的语句体里，可以用if/else, case,while等控制结构：可以进行sql编程

##存储过程语法
###新建一个存储过程
```sql
CREATE PROCEDURE 存储过程名字()
BEGIN
  #SQL语句
END||
```
###查看现有的存储过程
```sql
SHOW PROCEDURE STATUS;
```
###删除存储过程
```sql
DROP PROCEDURE 存储过程名字;
```
###调用存储过程
```sql
CALL 存储过程名字();
```