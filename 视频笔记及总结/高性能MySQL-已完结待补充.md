# 高性能MySQL

## 一、MySQL的体系结构

## 一、MySQL架构介绍

### 1.1 高级MySQL

MySQL内核

MySQL服务器优化

各种参数常量设置

查询语句优化

容灾备份

SQL编程

### 1.2 MySQL配置文件

#### 1.2.1 MySQL安装

​	以GA(General Availability)版为例：

```shell
rpm -ivh xxx.rpm
mysqladmin -version
#查看用户名和用户组
cat /etc/passwod
cat /etc/groups
#启动、停止mysql的服务
service mysql start
service mysql stop
#连接mysql
mysql -u root -p
#设置开机自启动
chkconfig mysql on
chkconfig --list
cat /etc/inittab
#查看开机自启动配置
ntsysv
```

#### 1.2.2 MySQL目录结构

数据文件：/var/lib/mysql/...

配置文件：/usr/share/mysql

命令目录：/usr/bin

启停脚本：/etc/init.d/mysql

#### 1.2.3 MySQL配置文件

my-default.cnf

```shell
mv my-default.cnf /etc/my.cnf
```

主要配置文件：

- 二进制日志log-bin，主从复制
- 错误日志log-error，默认是关闭的，记录
- 查询日志log，记录查询的sql语句
- 数据文件
  1. frm文件：存放表结构
  2. myd文件：存放表数据
  3. myi文件：存放表索引
- 

#### 1.2.4 常用管理命令

```sql
show tables;
show databases;
use xxxdatabase; 
-- 字符集，装完数据库后立即修改
show variables like '%char%'
```

#### 1.2.5 MySQL逻辑架构

![](..\image\MySQL架构图.jpg)

1. 连接层
2. 服务层
3. 引擎层
4. 存储层

#### 1.2.6 存储引擎

​	查询命令：

```sql
show engines;
show variables like '%storage_engine%'
```

InnoDB与MyISAM对比：

| 对比项 | MyISAM                     | InnoDB                                                       |
| ------ | -------------------------- | :----------------------------------------------------------- |
| 主外键 | 不支持                     | 支持                                                         |
| 事务   | 不支持                     | 支持                                                         |
| 行表锁 | 表级锁                     | 行级锁                                                       |
| 缓存   | 只缓存索引，不缓存真实数据 | 不仅缓存索引还要缓存真是数据，对内存要求较高，而且内存大小对性能有决定性的影响 |
| 表空间 | 小                         | 大                                                           |
| 关注点 | 性能                       | 事务                                                         |

阿里更换了存储引擎Percona、xtradb。

AliSql+AliRedis

## 二、SQL优化分析

### 2.1 性能下降原因分析

查询语句写的烂；

索引失效：单值索引、复合索引

```sql
create index idx_user_name on user(name);
create index idx_user_name_Email on user(name, email);
```

关联查询太多join（设计缺陷）

服务器与数据库各个参数的设置（缓冲、线程池等）

### 2.2 常见Join查询

#### 2.2.1 查询语句的执行顺序

from -> on -> where -> group by ->having -> select -> order by -> limit

![](..\image\select语句的执行顺序.JPG)

***TO OD: 与Oracle数据库对比***

![](..\image\SQL_JOIN图解.jpg)

*练习：*

```sql
select * from tbl_emp a inner join tbl_dept b on a.dept_id = b.id;
select * from tbl_emp a left join tbl_dept b on a.dept_id = b.id;
select * from tbl_emp a right join tbl_dept b on a.dept_id = b.id;
select * from tbl_emp a left join tbl_dept b on a.dept_id = b.id where b.id is null;
select * from tbl_emp a right join tbl_dept b on a.dept_id = b.id where a.dept_id is null;
-- Oracle可以，MySQL不可以
select * from tbl_emp a full outer join tbl_dept b on a.dept_id = b.id;
-- MySQL union联合去重
select * from tbl_emp a left join tbl_dept b on a.dept_id = b.id 
union 
select * from tbl_emp a right join tbl_dept b on a.dept_id = b.id;
```

### 2.3 索引

​	MySQL：索引（index）是帮助MySQL高效获取数据的数据结构。

​	在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用数据；B数索引；

​	一般来说索引本身也很大，不可能全部存储在磁盘上，因此索引往往以索引文件的形式存储在磁盘上；

​	聚集索引、复合索引、前缀索引、唯一索引默认都是使用B+树索引；

​	索引的优点：

- 提高检索效率，降低数据库的IO成本；

- 降低了数据排序成功，降低CPU的消耗；

  索引的缺点：

- 降低更新表的速度；
- 需要根据业务场景建立索引；

#### 2.3.1 MySQL索引分类

单值索引

唯一索引

复合索引

全文索引

```sql
show index from tbl_emp;
CREATE [UNIQUE] INDEX indexName ON mytable(columnname);
alter table tbl_name ad UNIQUE index_name(column_list);

```

BTree索引：数据存储在叶子节点上，非叶子节点不存储真是的数据，3层的B+树可以表示上百万的数据；

Hash索引

FULL-TEXT全文索引

R-Tree索引



区、段、块；

#### 2.3.2 哪些情况需要创建索引

- 主键自动建立唯一索引
- 频繁作为查询条件的字段应该创建索引
- 查询中与其他表关联的字段，外键关系建立索引
- 频繁更新的字段不适合创建索引
- 单值/复合索引的选择（在高并发下倾向于创建复合索引）
- 查询中排序的字段
- 查询中统计或分组的字段

#### 2.3.3 哪些情况不适合建立索引

- 表记录太少，300万左右需要做优化
- 经常增删改的表
- 数据重复比较多的字段，效果不明显；

### 2.4 性能分析

### 2.4.1 MySQL Query Optimizer

### 2.4.2 MySQL常见瓶颈

### 2.4.3 Explain

​	Explain关键字查看执行计划: Explain + SQL语句；

​	Explain输出内容解析：

- id：id相同，执行顺序从上到下；id不同，id值越大，优先级越高；
- select_type：PRIMARY(最外层查询)、SUBQUERY(子查询)、SIMPLE、DERIVED、UNION、UNION RESULT
- table
- type：ALL(全表扫描)、index(Full Index Scan，全索引扫描)、range、ref(非唯一性索引扫描)、eq_ref(唯一性索引扫描，对于每个索引建，表中只有一条记录与之匹配)、const,system、NULL；从最好到最差：system>const>eq_ref>ref>range>index>ALL
- possible_keys：显示可能应用到在这张表中的索引，一个或多个；查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用；
- key：实际使用的索引
- key_len：索引中使用的字节数
- ref
- row
- filter
- Extra：Using filesort(使用了外部索引排序)、Using temporary(使用了临时表)、Using index（使用了覆盖索引，如果同时出现Using where表明索引被用来执行索引键值得查找）、Using where、using join buffer、impossible where、select tables optimized away、distinct

### 2.4.4 索引优化分析

```sql
EXPLAIN select * from class left join book on class.card = book.card;
alter table book add index Y('card'); -- 针对上面语句应加在被连接表中
show index from book;
```

- 尽量减少Join语句中的NestedLoop的循环总次数，“永远用小结果集驱动大的结果集”；
- 优先优化NestedLoop的内层循环；
- 保证Join语句中被驱动表上Join条件字段已经被索引；
- 当无法保证被驱动表的join条件字段被索引且内存资源充足的前提下，不要太林溪JoinBuffer的设置；

### 2.4.5 索引失效分析

- 最佳左前缀原则，如果中间字段没用，则只可以使用一部分；
- 不在索引列上做任何操作（计算、函数、类型转换），会导致索引失效而转向全表扫描；
- 尽量使用覆盖索引，减少select *
- MySQL在使用不等于的时候无法使用索引导致全表扫描；
- is null，is not null也无法使用索引；
- like以通配符开头，MySQL索引失效会变成全表扫描；覆盖索引还可以生效；
- 字符串不加单引号索引失效；varchar类型查询，不要省略单引号；
- 少用or，用它来连接时会索引失效；
- 存储引擎不能使用索引中范围条件右边的列；

![](..\image\索引使用题目理解.JPG)

goup by基本上都需要进行排序，会有临时表产生；

![](..\image\索引一般建议.JPG)

## 三、查询截取分析

常用SQL优化步骤：

1. 首先，根据生产情况，初步定为查询语句；
2. 开启慢查询日志，设置阈值，将SQL抓取出来；
3. explain分析慢SQL；
4. show profile；比explain更细致，执行细节和生命周期；
5. 进行SQL数据库服务器的参数调优；

如果不在索引列上，filesort有两种算法：双路排序和单路排序；

1. Order by时尽量不要用select *；
2. 尝试提高sort_buffer_size;
3. max_length_for_sort_data参数设置

group by：

- group by 实质是先排序后分组，遵照索引建的最佳左前缀；
- where高于having，能写在where限定的条件就不要去having限定了；

### 3.1 慢查询日志

​	默认没有打开慢查询日志：

```sql
show variables like '%slow_query_log%';
set global_slow_query_log=1;
show variables like 'long_query_time%'; -- 阈值
select sleep(4); -- 可以用来测试
show global ...
```

永久生效需修改配置；

分析工具mysqldumpslow --help；

### 3.2 批量数据脚本

```sql
DELIMITER $$
CREATE FUNCTION rand_string(n INT) RETURNS VARCHAR(255)
BEGIN
	DECLARE chars_str VARCHAR(100) ...
END $$

CREATE PROCEDURE insert_emp(IN START INT(10), IN max_num INT(10))
BEGIN
	REPEAT
	SET i = i + 1;
	UNTIL i = max_num
	END REPEAT;
END

```

### 3.3 Show Profile

分析当前会话中语句执行的资源消耗情况，默认情况下，参数处于关闭状态，并保存最近15次的运行结果；

```sql
show variables like 'profiling';
show profile cpu,block io for query 3;
```

## 四、MySQL锁机制

读锁(共享锁)、写锁（排他锁）

表锁、行锁

```sql
show open tables;
lock table mylock read, book write;
unlock tables;
show status like 'table%';
show status like '%innodb_row_lock%'
```

MyISAM引擎加读锁后，当前会话不能查询其他未锁定的表，也不能修改锁定的表；其他会话可以查询；

![](../image/SQL锁优化建议.JPG)

## 五、主从复制

待补充

以上内容是听课笔记

-------------------------

下面内容是《高性能MySQL》中的读书笔记

## 第四章 Schema与数据类型优化



## 第五章 创建高性能的索引



## 第六章 查询性能优化

**查询优化器**：

可能导致优化器选择错误的执行计划的原因：



MySQL能够处理的优化类型：

- 重新定义关联表的顺序；
- 将外连接转化为内连接；
- 使用等价变换规则；
- 优化COUNT()、MIN()、MAX()；
- 预估并转化为常数表达式；
- 覆盖索引扫描；
- 子查询优化；
- 提前终止查询：典型例子如使用LIMIT的时候；
- 等值传播；
- 列IN()的比较：MySQL中不等同于OR条件的子句，MySQL先将列表中的数据进行排序，然后用二分查找来确定是否满足条件；







----------------------------------

性能优化：

show profile

show processlist

show variables like '%max_connections%'



schema与数据类型优化

数据类型优化：

- 更小的通常更好
- 简单就好
- 尽量避免null
- datetime、timestamp、date
- 使用枚举替代字符串类型

合理使用范式和反范式

主键的选择

字符集



索引、外键、锁















