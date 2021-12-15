#### SQL分类

- DDL，数据定义语言，数据库对象。create,drop,alter
- DML，数据操作语句。insert,update,delete,select
- DCL，数据控制语句。grant,revoke

##### DDL

数据库内部对象的创建、删除、修改

1. 创建数据库

   ***create database dbname***

2. 删除数据库

   ***drop database dbname***

3. 创建表

   create teble tablename(

   column_name_1 column_type_1 constraints,

   column_name_2  column_type_2 constraints,

   ...,

   column_name_n  column_type_n constraints)

4. 删除表

   ***drop table tablename***

5. 修改表

   修改表类型	***alter table tablename modify column column_name***

   增加表字段	***alter table tablename add column column_name  column_type constraints***

   删除表字段	***alter table tablename drop column column_name***

   修改字段名	***alter table tablename change column _old column _new column_type***

   修改字段排序	after  first

   修改表名	***alter table tablename rename new_tablename***

##### DML

数据库中表记录的操作

1. 插入记录

   ***insert into tablename(field) values (value),(value1)***

2. 更新记录

   ***update tablename set field = value where***

3. 删除记录

   ***delete from tablename where***

4. 查询语句

   select

   排序	order by

   限制	***limit offset_start row_count***  offset_start 起始偏移量，默认0  row_count 显示行数

   聚合	group by  having(聚合后的结果进行筛选)

   表连接	区别是内连接仅选出表中互相匹配的记录；外连接选出其他不匹配的记录

   内连接	

   外连接	左连接（包含所有左边表的记录），右连接（所有右边表的记录）

   子查询	查询的条件是另一个select的结果
   
   记录联合
   
   unionall	记录结果直接联合在一起
   
   union	进行一次distinct

##### DCL

管理对象权限

#### 存储引擎

查看支持的存储引擎

***show ENGINES***

***show VARIABLES like 'have%'***

![image-20210709142808662](C:\Users\dell\Desktop\task\md\MySQL.assets\image-20210709142808662.png)

##### MyISAM

5.5之前的默认存储引擎。不支持事务，不支持外键。优势：访问速度快

每个MyISAM在磁盘上存储成3个文件，文件名和和表名相同。

***.frm（存储表定义）***

***.MYD（MYData，存储数据）***

***.MYI（MYIndex，存储索引）***

支持3中不同的存储格式

==**静态（固定长度）表**==

==**动态表**==

==**压缩表**==

##### InnoDB

5.5之后默认存储引擎，支持事务。写的处理效率差一些，占用更多的磁盘空间保存数据 和索引

1. **自动增长列**	必须是索引，或组合索引中第一列

2. **外键约束**（只有InnoDB）父表必须有对应的索引，子表在创建外键时也会自动 创建对应的索引

3. **存储方式**	

   **共享表空间存储**

   表结构存储在**.frm**中，数据和索引保存在innodb_data_home_dir和innodb_data_file_path中

   **多表空间存储**

   表结构存储在**.frm**中，每张表的数据和索引单独保存在**.idb**中。如果是分区表，文件名为**表名+分区名.idb**

##### 如何选择合适的存储引擎

MyISAM	以读和插入为主，少量更新和删除操作，对事物和并发要求不高

InnoDB	事物完整有较高要求，数据操作除了插入和查询，还有许多更新，删除，有效降低由更新，删除导致的锁定

#### 数据类型的选择

1. char与varchar

   MyISAM	建议固定长度代替可变长度数据列

   InnoDB	建议varchar类型。内部的行存储格式没区分固定和可变（==所有数据行都使用指向数据列值的头指针==）。只要性能因数是数据行使用的存储总量。==char平均占用空间多于varchar==，因此varchar处理数据行存储总量和磁盘I/O较好

2. 浮点数与定点数

   浮点数存在误差；对货币等精度敏感的数据，定点数表示

#### 索引的设计和使用

1. 设计原则

   最适合的索引列是在where或连接语句中指定的列

   使用唯一索引，索引列的基数越大，索引效果越好

   使用短索引，如果对字符串列进行索引，应该指定一个前缀长度。较小索引涉及**磁盘I/O较少**，较短值**比较更快**，高速缓存中的块能**容纳更多的键值**

   最左前缀

   不要过度索引	索引占用额外磁盘空间，降低写的性能

   InnoDB存储引擎的表，普通索引会保存主键的键值，所以主键尽可能较短，可减少索引的磁盘占用，提高索引效率

2. BTREE索引与HASH索引

   HASH索引

   只用于=或<=>操作符的等式比较；

   优化器不能使用hash索引来加速order by 

   只能使用整个关键字来搜索一行

   BTREE索引

   使用>、<、>=、<=、BETWEEN、!=或者<>，或者LIKE 'pattern'（pattern不以通配符开始）时，都可以使用列上索引

#### SQL注入

1. 绑定变量（**Java驱动**中采用PreparedStatement语句实现）
2. 程序接口提供的对特殊字符处理的转换函数
3. 自定义函数进行校验

#### MySQL分区

优点：1.和单个磁盘或文件系统分区相比，可存储更多的数据；2.优化查询；3.对于过期或不需要保存的数据，可通过删除这些数据相关的分区来快速删除；4.跨多个磁盘分散数据查询，提高吞吐量

##### 分区概述

分区键。分区键用于根据某个区间值、特定值列或hash函数值执行数据的聚合

##### 分区类型

RANGE分区：基于连续区间范围，将数据分配到不同分区

LIST分区：类似RANGE分区。LIST基于枚举值分区

HASH分区：基于给定分区个数

KEY分区：

Columns分区：解决5.5之前RANGE和LIST只支持整数分区

#### SQL优化

##### 定位执行效率较低的SQL

慢查询日志

***show processlist***，实时查看SQL执行情况（线程状态，锁表）

![image-20210712143701249](C:\Users\dell\Desktop\task\md\MySQL.assets\image-20210712143701249.png)

##### EXPLAIN分析低效SQL执行计划

![image-20210712144110837](C:\Users\dell\Desktop\task\md\MySQL.assets\image-20210712144110837.png)

select_type：select的类型。常见simple（简单表）、primary（主查询）、union、subquery

table：输出数据集的表

type：表中找到所需行的方式，或者叫访问类型

​	all -> index -> range -> ref -> eq_ref -> const,system -> null

​	从左到右，性能由低到高

​	all：全表扫描，遍历全表

​	index：索引全扫描，遍历整个索引

​	range：索引范围扫描，如<、<=、>、>=、between

​	ref：使用非唯一索引扫描或唯一索引的前缀扫描，返回匹配某个单独值的记录行

​	eq_rep：使用索引是唯一索引

​	const,system：单表中最多有一个匹配行

​	null：不访问表或索引，直接就能得到结果

possible_keys：查询时可能用到的索引

key：实际使用的索引

key_len：索引的长度

rows：扫描行的数量

Extra：执行情况的说明和信息

explain extended + show warnings：SQL被执行前优化器做了哪些改进

```sql
EXPLAIN EXTENDED SELECT * from payment WHERE customer_id >=300 and customer_id <=360;
show warnings;
```

```sql
select count(*) FROM payment;

show PROFILES;
# sql执行过程
show PROFILE FOR QUERY 142;
```

##### 索引问题

###### 能够使用索引的场景

​	1.匹配全值，对索引中的列都有等值匹配的条件；

​	2.匹配值的范围查询，对索引值进行范围查询；

​	3.匹配最左前缀，B-Tree索引使用的首要原则；

​	4.进对索引进行查询，查询列都在索引字段；

​	5.匹配列前缀；

​	6.索引部分精确而其他部分范围匹配；

###### 存在索引但不使用索引的场景

​	1.以%开头的like查询不能利用B-Tree索引；

```sql
EXPLAIN select * from actor where last_name like '%NI%'
# 修改
EXPLAIN select * from (select actor_id from actor where last_name like '%NI%')a,actor b where a.actor_id = b.actor_id
```

​	2.数据类型出现隐式转换（特别列类型是字符串，where后应该用引号引起来）；

​	3.使用复合索引，查询条件不包含索引列最左边部分（不满足最左匹配原则）；

​	4.用or分隔开的条件，如or前有索引列，or后没索引列，那么索引都不会用到；

###### 实用优化方法

​	定期分析表和检查表

```sql
# 分析表
ANALYZE TABLE actor
# 检查表
CHECK TABLE actor
```

​	定期优化表（删除表大部分或表的列进行很多更改）

```sql
OPTIMIZE table actor
```

#### 优化数据库对象

##### 通过拆分提高表的访问效率

垂直拆分：主码和一些列放到一个表，然后主码和另外的列放到另一个表

如一些列常用，一些不常用，可垂直拆分；使数据行变小，一个数据页存放更多数据，查询时减少I/O，缺点管理冗余，查询所有数据需联合操作

水平拆分：根据一列或多列的值把数据行放到2个独立的表中

如表很大，分割后降低查询时的读的数据和索引页数，降低索引层数；表数据本来就有独立性，记录地区，不同时期的数据；数据存到多个介质

##### 逆规范化

增加冗余列，增加派生列

##### 使用中间表

#### 读写分离

读写分离则是根据SQL语义的分析，将读操作和写操作分别路由至主库与从库。

通过一主多从的配置方式，可以将查询请求均匀的分散到多个数据副本，能够进一步的提升系统的处理能力。 使用多主多从的方式，不但能够提升系统的吞吐量，还能够提升系统的可用性，可以达到在任何一个数据库宕机，甚至磁盘物理损坏的情况下仍然不影响系统的正常运行。

读写分离虽然可以提升系统的吞吐量和可用性，但同时也带来了==数据不一致==的问题。 这包括多个**主库之间**的数据一致性，以及**主库与从库**之间的数据一致性的问题

##### 核心概念

主库

添加、删除、更新数据操作

从库

查询数据操作

主从同步

主库的数据异步的同步到从库的操作。由于主从同步的异步性，从库与主库的数据会短时间内不一致

负载均衡策略

负载均衡策略将查询请求疏导至不同从库
