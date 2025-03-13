# MySQL 存储引擎笔记

## 1. 简介

存储引擎是MySQL中**负责数据存储和检索的底层组件**。不同的存储引擎提供不同的特性（事务、锁机制、索引类型等），直接影响数据库的性能和功能。

```sql
--查询建表语句，默认存储引擎：InnoDB
show create table account;

-- 查看当前数据库支持的存储引擎
SHOW ENGINES;

-- 查看某张表的存储引擎
SHOW TABLE STATUS LIKE '表名';
```

## 2. InnoDB 存储引擎

### **介绍**

InnoDB是一种兼顾高可靠性和高性能的通用存储引擎，在 MySQL 5.5 之后，InnoDB是默认的  MySQL 存储引擎。

### 核心特性

- **事务支持**：完全符合ACID特性，支持事务；
- **行级锁**：提高并发访问性能；
- **外键约束**：支持外键约束，保证数据的完整性和正确性；

### 文件

1.xxx.ibd：xxx代表的是表名，**innoDB引擎的每张表都会对应这样一个表空间文件**，存储该表的表结 构（frm-早期的 、sdi-新版的）、数据和索引。 参数：innodb_file_per_table

2.`show variables  like 'innodb_file_per_table;`

3.可以在命令行中执行ibd2sdi score.ibd查看xxx.ibd文件

4.**innoDB**的逻辑存储结构
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/a1bb0494c5d847a2885e015d626b299e.png)



### 适用场景

- OLTP系统（高并发读写）
- 需要事务的场景（如金融交易）
- 数据完整性要求高的场景

```sql
-- 显式指定InnoDB引擎建表
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(50)
) ENGINE=InnoDB;  
```

⚠️ **易错注意**：

1. 默认配置下`autocommit=1`，每条SQL自动提交事务
2. 外键关联的表必须都是InnoDB引擎
3. 大事务可能导致undo log膨胀

------

## 3. MyISAM 存储引擎

### 介绍

MyISAM是MySQL早期的默认存储引擎。

### 核心特性

- **不支持事务，不支持外键** 
- **支持表锁**，不支持行锁
-  访问速度快

### 适用场景

- 日志系统（写入后很少修改）
- 数据仓库（大量读操作）
- 不需要事务的简单应用

### 文件

xxx.sdi：存储表结构信息 （json格式数据）

xxx.MYD: 存储数据

 xxx.MYI: 存储索引

```sql
-- MyISAM建表示例
CREATE TABLE logs (
    id INT PRIMARY KEY,
    content TEXT
) ENGINE=MyISAM;
```

⚠️ **易错注意**：

1. **不支持事务**，崩溃后恢复困难
2. 写操作会锁定整个表
3. 默认配置可能丢失最后一条插入记录（配置`delay_key_write`需谨慎）

------

## 4. Memory 存储引擎

### 介绍

Memory引擎的表数据时**存储在内存中**（访问速度比较快）的，由于受到硬件问题、或断电问题的影响，只能将这些表作为 临时表或缓存使用。

### 核心特性

- **内存存储**：重启后数据丢失
- **哈希索引**：**默认**使用哈希索引

### 适用场景

- 临时数据/缓存表
- 快速查找的维度表
- 中间结果暂存

```sql
-- 创建内存表
CREATE TABLE temp_data (
    key VARCHAR(32) PRIMARY KEY,
    value TEXT
) ENGINE=MEMORY;    
```

⚠️ **易错注意**：

1. **不支持BLOB/TEXT类型**（报错：The table 'xxx' is full）
2. 内存不足会导致表损坏
3. 最大受限于`max_heap_table_size`参数

------

## 5. 存储引擎选择策略

**面试题InnoDB和MyISAM的区别？**
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/33336ea42b954601a578157c874dd34b.png)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/04afd59422f94597ae2885168bf6b877.png)



------

### 5.存储引擎的选择

在选择存储引擎时，应该根据应用系统的特点选择合适的存储引擎。对于复杂的应用系统，还可以根据 实际情况选择多种存储引擎进行组合。 

InnoDB: 是Mysql的默认存储引擎，支持事务、外键。**如果应用对事务的完整性有比较高的要求，在并发条件下要求数据的一致性，数据操作除了插入和查询之外，还包含很多的更新、删除操作**，那么InnoDB存储引擎是比较合适的选择。(用的多)

MyISAM ： 如果应用是以**读操作和插入操作为主，只有很少的更新和删除操作，并且对事务的完 整性、并发性要求不是很高**，那么选择这个存储引擎是非常合适的。 MEMORY：将所有数据保存在内存中，访问速度快，通常用于临时表及缓存。(用的少，被mongodb这样的nosql数据库取代了)

MEMORY的缺陷就是对表的大小有限制，太大的表无法缓存在内存中，而且无法保障数据的安全性。（用的少，被Redis这样的nosql数据库取代了）

## 6. 小结

1．体系结构

连接层、服务层、引擎层、存储层

2．存储引擎简介

SHOW ENGINES;
CREATE TABLE XXXX(....) ENGINE=INNODB;

3．存储引擎特点

INNODB与MyISAM：事务、外键、行级锁

4．存储引擎应用

INNODB：存储业务系统中对于事务、数据完整性要求较高的核心数据。
MyISAM：存储业务系统的非核心事务。

**5.引擎选择黄金法则**：

1. **默认选择InnoDB**：除非有特殊需求
2. **读多写少考虑MyISAM**：但要做好备份方案
3. **临时数据用Memory**：注意数据易失特性