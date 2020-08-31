# <center>Oracle数据库从入门到入魔

## P11 Oracle 11g的表

高水位线：

```sql
analyze table t2 compute statistics for table; -- 分析
alter table t2 enable row movement;            -- 设置表可行可移动
alter table t2 shrink space;                   -- 收缩表空间，收缩数据库，降低高水位线
-- 查询
select u.table_name, u.blocks, u.num_rows, from user_tables u where u.table_name='T2'; 

```

truncate删除全部记录，不能回滚，同时会将HWM降低到最低；

```sql
drop table t2 [cascade constraints][purge];
alter table t2 set unused column tele;  -- 设置某一列不可用
alter table t2 drop unused columns;     -- 删除，与上一语句可加快删除列的速度
```

IOT（索引组织表）：IOT的索引和数据存在一起，数据存储在索引块中；区别于普通表的无序组织表；经常需要通过主键访问表中数据时，可考虑建立索引组织表；语法如下：

```sql
create table iot_student (
	sno int,
    sname varchar2(100),
    sage int,
    constraint pk_student primary key(sno))
    organization index
    pctthreshold 30 overflow tablespace users;
```

***TO DO: pctthreshold的含义***

簇表：当两张表经常需要关联查询时，可以考虑

```sql
create cluster cluster1(code_key number);
create table student(sno1 number,sname varchar2(100)) cluster cluster1(sno1);
create table address(sno2 number, zz varchar2(100)) cluster cluster2(sno2);
create index index1 on cluster cluster1;
select * from user_clusters;
select * from user_clu_columns;
purge recyclebin; -- 清空回收站
```

### 临时表

```sql
create global temporary table
```

临时表是针对session来使用的，其他session中看不到插入的数据；commit和rollback时，数据也会消失

## P16 索引

索引简介：

- 索引是与表相关的一个可选结构；

- 目的在于提高查询语句的性能；

- 减少磁盘I/O；
- **在逻辑上和物理上都独立于表的数据**
- Oracle自动维护索引

索引分类：除了位图索引，其他的都是B树索引

1. 唯一索引
2. 组合索引
3. 反向键索引：反向存放，使B树更加平衡
4. 位图索引：
   - 适合存放在低基数列上；
   - 位图索引不直接存储ROWID，而是存储字节位到ROWID的映射，节省存储空间；
   - 如果索引列经常更新的话，不适合建立位图索引；
   - 总体来说位图索引适合于数据仓库中，不适合OLTP中
5. 基于函数的索引
   - 基于一个或多个列上的函数或表达式创建的索引
   - 表达式中不能出现聚合函数
   - 不能在LOB类型的列上创建
   - 创建时必须具有QUERY REWRITE权限

```sql
create index idx_student on student(sno);
select * from user_indexes;
select * from user_ind_columns u where u.index_name='idx_student';

analyze index <index_name> validate structure; -- 分析索引
select * from index_stats; -- 当发现pct_used较低时，可重建索引
alter index idx_student rebuild; -- 重建索引，ONLINE\NOLOGGING\COMPUTE STATISTICS

create unique index idx_sno on student(sno); -- null值可以重复插入

create index idx_student on student(sno) REVERSE; -- 反向索引

create bitmap index ... -- 位图索引

create index ind3 on student( upper(sname) ); -- 基于函数的索引
```

```sql
begin
	for i in 1..1300000 loop
		insert into t values(ltrim(to_char(i, '00000009')));
		if mode(i, 100)=0 then
			commit;
		end if;
	end loop;
end;
/
```

与分区有关的索引有三种：

- 局部分区索引：索引分区范围与数据表分区一致
- 全局分区索引
- 全局非分区索引

```sql
select * from user_ind_partitions
```

