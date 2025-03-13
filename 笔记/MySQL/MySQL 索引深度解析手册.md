# MySQL 索引深度解析手册

## 一、索引核心概念

### 1.1 什么是索引？

索引（index）是帮助MySQL**高效获取数据的数据结构(有序)**。在数据之外，数据库系统还维护着满足 特定查找算法的数据结构，这些数据结构以某种方式引用（指向）数据， 这样就可以在这些数据结构 上实现高级查找算法，这种数据结构就是索引。

- **本质**：类似书籍目录的`高效数据定位结构`
- **存储特征**：独立的数据结构（B+Tree为主），与数据分离存储
- **核心价值**：将随机IO变为`顺序IO`，减少磁盘扫描范围

### 1.2 演示：

假如我们要执行的SQL语句为 ： select * from user where age = 45;

1). 无索引情况
![](https://i-blog.csdnimg.cn/direct/e3241eb44ef643ffb5b1c72b2a17e54f.png)




在无索引情况下，就需要从第一行开始扫描，一直扫描到最后一行，我们称之为**全表扫描，性能很低**。

2). 有索引情况

如果我们针对于这张表建立了索引，假设索引结构就是二叉树，那么也就意味着，会对age这个字段建 立一个二叉树的索引结构。

```sql
-- 创建索引示例
CREATE INDEX idx_user_age ON user(age);  -- 年龄字段建立常规索引
```
![](https://i-blog.csdnimg.cn/direct/5fd7266d7d03429d830dca95549eef51.png)



此时我们在进行查询时，**只需要扫描三次就可以找到数据了**，极大的提高的查询的效率。

<!--备注： 这里只是假设索引的结构是二叉树，介绍一下索引的大概原理，只是一个示意图，并 不是索引的真实结构，索引的真实结构，后面会详细介绍。-->

### 1.3特点
![](https://i-blog.csdnimg.cn/direct/f688ffa0d75942e6a377e56264bd2add.png)


解析：

我们现在的磁盘很便宜，所以这个索引占用空间这个问题不大。

插入、删除操作不过要对数据操作，还要维护索引的结构，所以会影响效率。不过实际项目里主要是进行查询操作，所以这个缺点也可以忽略。

## 二、索引结构深度对比

### 索引结构概述：

MySQL的索引是在存储引擎层实现的，不同的存储引擎有不同的索引结构，主要包含以下几种：

![](https://i-blog.csdnimg.cn/direct/c143e291b9a84d7b9d62478e75b09303.png)


补充：

1.像InnoDB、MyISMA，Memory都支持B+Tree

2.hash索引性能高，但不支持范围查询

3.R-tree用的少，了解一下

4.Full-text用的少，了解

不同的存储引擎对于索引结构的支持情况。
![](https://i-blog.csdnimg.cn/direct/5eb030e9b80c447581e01c5aaae5116f.png)

**注意： 我们平常所说的索引，如果没有特别指明，都是指B+树结构组织的索引。**

### 2.1 B+Tree（InnoDB默认）

####  二叉树

假如说MySQL的索引结构采用二叉树的数据结构，比较理想的结构如下：

![](https://i-blog.csdnimg.cn/direct/577f63339d044ed4a36c037b1f01b80e.png)


如果主键是顺序插入的，则会形成一个单向链表，结构如下：

![](https://i-blog.csdnimg.cn/direct/745aa5c96f384a46a16f88d117b03a2c.png)


所以，如果选择二叉树作为索引结构，会存在以下缺点： 

**顺序插入时，会形成一个链表，查询性能大大降低。 **

 **大数据量情况下，层级较深，检索速度慢。**

此时大家可能会想到，我们可以选择红黑树，红黑树是一颗自平衡二叉树，那这样即使是顺序插入数据，最终形成的数据结构也是一颗平衡的二叉树,但是，即使如此，由于红黑树也是一颗二叉树，所以也会存在一个缺点：**大数据量情况下，层级较深，检索速度慢。**所以，在MySQL的索引结构中，并没有选择二叉树或者红黑树，而选择的是B+Tree，那么什么是 B+Tree呢？在详解B+Tree之前，先来介绍一个B-Tree。

###  B-Tree

B-Tree，B树是一种多叉路衡查找树，相对于二叉树，B树每个节点可以有多个分支，即多叉。

 以一颗最大度数(max-degree)为5(5阶)的b-tree为例，那这个B树每个节点最多存储4个key，5 个指针：

![](https://i-blog.csdnimg.cn/direct/5aa5d4891f8c44b5ad5ea20e45ae46bc.png)


知识小贴士: 树的度数指的是一个节点的子节点个数。

B树可视化网站：https://www.cs.usfca.edu/~galles/visualization/BTree.html

特点：  5阶的B树，每一个节点最多存储4个key，对应5个指针。

​	 		一旦节点存储的key数量到达5，就会裂变，中间元素向上分裂。 

​			在B树中，非叶子节点和叶子节点都会存放数据。

### B+Tree

B+Tree是B-Tree的变种，我们以一颗最大度数（max-degree）为4（4阶）的b+tree为例，来看一 下其结构示意图：

![](https://i-blog.csdnimg.cn/direct/540c4d3d883144bb97ec8f257ed7bb6b.png)


我们可以看到，两部分：

 绿色框框起来的部分，是索引部分，仅仅起到索引数据的作用，不存储数据。

 红色框框起来的部分，是数据存储部分，在其叶子节点中要存储具体的数据。

B+Tree可视化：https://www.cs.usfca.edu/~galles/visualization/BPlusTree.html

B+Tree 与 B-Tree相比，主要有以下三点区别： 

所有的数据都会出现在叶子节点。

叶子节点形成一个单向链表。

非叶子节点仅仅起到索引数据作用，具体的数据都是在叶子节点存放的。



### 2.2 Hash索引（Memory引擎）

MySQL中除了支持B+Tree索引，还支持一种索引类型---Hash索引。

```sql
-- 创建Hash索引
CREATE TABLE test_hash (
    id INT PRIMARY KEY,
    code VARCHAR(20),
    INDEX idx_code USING HASH (code)
) ENGINE=MEMORY;   
```

1). 结构 

哈希索引就是采用一定的hash算法，将键值换算成新的hash值，映射到对应的槽位上，然后存储在 hash表中。

**如果两个(或多个)键值，映射到一个相同的槽位上，他们就产生了hash冲突（也称为hash碰撞），可 以通过链表来解决。**

![](https://i-blog.csdnimg.cn/direct/a04cd300d9e74ac4b0f066ebc1860ed6.png)



2）.特征：

- O(1)时间复杂度查找，查询效率高
- 仅支持`=, <=>`等值查询，不支持范围查询（（between，>，< ，...）
- 无排序能力（ORDER BY无效）

3). 存储引擎支持 在MySQL中，支持hash索引的是Memory存储引擎。 而InnoDB中具有自适应hash功能，hash索引是 InnoDB存储引擎根据B+Tree索引在指定条件下自动构建的。

### 2.3 结构选择建议

| 场景         | 推荐结构 | 案例说明           |
| ------------ | -------- | ------------------ |
| 电商商品搜索 | B+Tree   | 需支持价格范围查询 |
| 用户登录认证 | Hash     | 精确匹配用户名密码 |
| 日志时间查询 | B+Tree   | 按时间范围快速检索 |

#### **面试题： 为什么InnoDB存储引擎选择使用B+tree索引结构?**

A. 相对于二叉树，层级更少，搜索效率高；

B. **对于B-tree，无论是叶子节点还是非叶子节点，都会保存数据，这样导致一页中存储的键值减少，指针跟着减少，要同样保存大量数据，只能增加树的高度，导致性能降低；** 

（一页可以存放的数据大小是固定的，如果要保存数据，那么必然会减少存储键值和指针的空间大小，相应的就会导致树的层级变高，进而影响查询效率。）

C. 相对Hash索引，B+tree支持范围匹配及排序操作；

## 三、索引分类详解

### 3.1 功能分类

![](https://i-blog.csdnimg.cn/direct/e46080e50d374352920d82408b7a7285.png)


```sql
-- 主键索引（自动创建）
ALTER TABLE user ADD PRIMARY KEY (id); 

-- 唯一索引
CREATE UNIQUE INDEX idx_user_email ON user(email);

-- 全文索引（需MyISAM引擎）
CREATE FULLTEXT INDEX idx_content ON article(content);

-- 空间索引（GIS数据）
CREATE SPATIAL INDEX idx_gps ON locations(coord);
```

### 3.2 物理存储分类（重点理解）

| 类型                                         | 特征       | 数据存储方式                                                 |
| -------------------------------------------- | ---------- | ------------------------------------------------------------ |
| 聚簇索引(Clustered Index)                    | 主键索引   | 将数据存储与索引放到了一块，索引结构的叶子 节点保存了行数据（必须有，且只有一个） |
| 二级索引(Secondary Index辅助索引/非聚集索引) | 非主键索引 | 将数据与索引分开存储，索引结构的叶子节点关联的是对应的主键（可以有多个） |

#### 聚集索引选取规则:

 如果存在主键，主键索引就是聚集索引。 

如果不存在主键，将使用第一个唯一（UNIQUE）索引作为聚集索引。

 如果表没有主键，或没有合适的唯一索引，则InnoDB会自动生成一个rowid作为隐藏的聚集索引。



聚集索引和二级索引的具体结构如下： 
![](https://i-blog.csdnimg.cn/direct/68c11aa10c3f4b289cabdc5c830cd33f.png)




 聚集索引的叶子节点下挂的是**这一行的数据** 。

 二级索引的叶子节点下挂的是**该字段值对应的主键值**。



接下来，我们来分析一下，当我们执行如下的SQL语句时，具体的查找过程是什么样子的。

![](https://i-blog.csdnimg.cn/direct/cff1704442144bd3ac758ff888875015.png)


 具体过程如下: 
 ①. 由于是根据name字段进行查询，所以先根据name='Arm'到name字段的二级索引中进行匹配查 找。但是在二级索引中只能查找到 Arm 对应的主键值 10。

 ②. 由于查询返回的数据是*，所以此时，还需要根据主键值10，到聚集索引中查找10对应的记录，最 终找到10对应的行row。 

 ③. 最终拿到这一行的数据，直接返回即可。

> 回表查询：这种先到二级索引中查找数据，找到主键值，然后再到聚集索引中根据主键值，获取 数据的方式，就称之为回表查询。



思考题： 以下两条SQL语句，那个执行效率高? 为什么? 

A. select * from user where id = 10 ;  

B. select * from user where name = 'Arm' ; 

备注: id为主键，name字段创建的有索引；

解答： A 语句的执行性能要高于B 语句。  

​			因为A语句**直接走聚集索引，直接返回数据**。 而B语句需要**先查询name字段的二级索引找到id值，然后再根据id值查询聚集索引**，也就是需要进行回表查询。



思考题：InnoDB主键索引的B+tree高度为多高呢?

![](https://i-blog.csdnimg.cn/direct/d91139fea1774bc580b071378101ce3f.png)




假设: 

一行数据大小为1k，一页中可以存储16行这样的数据。InnoDB的指针占用6个字节的空间，主键即使为bigint，占用字节数为8。 

高度为2： 

n * 8 + (n + 1) * 6 = 16*1024 , 算出n约为 1170 （n：指代key的数量，n+1指代指针的数量）

1171* 16 = 18736  

也就是说，如果树的高度为2，则可以存储 18000 多条记录。

 高度为3：

 1171 * 1171 * 16 = 21939856

 也就是说，如果树的高度为3，则可以存储 2200w 左右的记录。

## 四、索引操作全指南

### 4.1 索引语法

1). 创建索引  

**CREATE  [ UNIQUE | FULLTEXT ]  INDEX**  index_name  ON  table_name  (  index_col_name,... ) ; 

2). 查看索引

**SHOW  INDEX**  FROM  table_name ;

 3). 删除索引 

**DROP  INDEX ** index_name  ON  table_name ;

## 五、 SQL性能分析

SQL执行频率

通过 show [session|global] status 命令可以提供服务器状态信 息。通过如下指令，可以查看当前数据库的INSERT、UPDATE、DELETE、SELECT的访问频次：

```sql 
--session 是查看当前会话 ;
-- global 是查询全局数据 ; 
SHOW  GLOBAL STATUS LIKE  'Com_______'; 
```

![](https://i-blog.csdnimg.cn/direct/7372d02834484ff2b861ddf8d450a5c7.png)


Com_delete: 删除次数

 Com_insert: 插入次数

 Com_select: 查询次数 

Com_update: 更新次数

那么通过查询SQL的执行频次，我们就能够知道当前数据库到底是增删改为主，还是查询为主。 那假 如说是以查询为主，我们又该如何定位针对于那些查询语句进行优化呢？ 次数我们可以借助于慢查询 日志。

> 通过上述指令，我们可以查看到当前数据库到底是以查询为主，还是以增删改为主，从而为数据库优化提供参考依据。 **如果是以增删改为主，我们可以考虑不对其进行索引的优化。**如果是以查询为主，那么就要考虑对数据库的索引进行优化了。

### 5.1 EXPLAIN全解析

EXPLAIN 或者 DESC命令获取 MySQL 如何执行 SELECT 语句的信息，包括在SELECT语句执行过程中表如何连接和连接的顺序。

语法:

```sql
-- 直接在select语句之前加上关键字 explain / desc (效果是一样的)
 SELECT   字段列表   FROM   表名   WHERE  条件 ;
 EXPLAIN SELECT * FROM user WHERE age > 25 ORDER BY create_time;  
```

![](https://i-blog.csdnimg.cn/direct/6d1128ee96a840ad9377ee5a40584820.png)


 Explain 执行计划中各个字段的含义:

![](https://i-blog.csdnimg.cn/direct/eabb77bb137b44edb65c7632a072c504.png)


- **id值越大越先执行，id值相同从上至下**。

- select_type:simple, union,sunquery

- type:NULL(性能最高)，system（访问系统表）、const（根据主键访问）、eq_ref、ref（非唯一性的索引 例如：age一张表中出现相同的年龄非常的正常）、range、index（用了索引，但会遍历整个索引树）、all(全表扫描，性能最差)。优化时尽量把type向前进行优化。（但对于业务系统的sql一般不太可能出现null，一般查询空表才会出现null）

- possible_key：显示可能用到的索引

- key：实际用到的索引，没有展示NULL

- key_len:标识索引的使用的字节数，越短越好。

- rows：MySQL认为必须要执行查询的行数，在innodb引擎的表中，是一个估计值，
  可能并不总是准确的。

- filtered：表示返回结果的行数占需读取行数的百分比，filtered 的值越大越好。

- Extra：额外信息，查询过程没展示的字段

  **需要重点关注的信息：type，possible_key，key，key_len**

  ### 5.2 慢查询日志实战

   接下来，我们就来介绍一下MySQL中的**慢查询日志**。

  慢查询日志记录了所有执行时间超过指定参数（long_query_time，单位：秒，默认10秒）的所有 SQL语句的日志。（只要查询时间超过10秒，我们就认为这是一条慢sql需要进行优化。）

   MySQL的慢查询日志默认没有开启，我们可以查看一下系统变量 slow_query_log。

  (**慢查询日志**：专门用来定位哪些sql语句查询效率低，这样可以有针对性对sql语句进行优化。)

```sql
--查看当前是否开启慢查询日志
show variables like 'slow_query_log';

--如果要开启慢查询日志，需要在MySQL的配置文件（/etc/my.cnf）中配置如下信息：（可以使用vi器）
# 开启MySQL慢日志查询开关
slow_query_log=1
# 设置慢日志的时间为2秒，SQL语句执行时间超过2秒，就会视为慢查询，记录慢查询日志
long_query_time=2
--配置完毕之后，通过以下指令重新启动MySQL服务器进行测试，查看慢日志文件中记录的信息 /var/lib/mysql/localhost-slow.log
--重启mysql
systemctl restart mysqld
--再次查看就会发现已开启慢查询日志
```

### 5.3 性能优化三剑客

使用慢查询可能出现查询查询时间花费两秒以上的sql，但是遗漏了1.99秒的sql，为了避免这种情况，我们可以使用show profiles。

show profiles 能够在做SQL优化时帮助我们了解**时间都耗费到哪里**去了。

```sql
--通过have_profiling 参数，能够看到当前MySQL是否支持profile操作：yes支持 
SELECT  @@have_profiling;
--MySQL是支持profile操作的，但是开关是关闭的。可以通过set语句在session/global级别开启profiling
SET  profiling = 1;
-- 1. 查看每一条SQL的耗时基本情况
SHOW PROFILES;
+----------+------------+---------------------------------------+
| Query_ID | Duration   | Query                                 |
+----------+------------+---------------------------------------+
|        1 | 0.00036200 | SELECT * FROM user WHERE id=100       |
|        2 | 1.23456700 | SELECT * FROM orders WHERE amount>1000| 
-- （如果某个sql查询时间特别长，想要查看时间都花在上面地方）
-- 查看指定query_id的SQL语句各个阶段的耗时情况
show profile  for  query query_id;
-- 查看指定query_id的SQL语句CPU的使用情况
show profile  cpu for  query query_id;
```



```sql
-- 2. 分析IO消耗
SET profiling = 1;
SELECT * FROM large_table;
SHOW PROFILE CPU, BLOCK IO FOR QUERY 1;
```



```sql
-- 3. 实时监控
SELECT * FROM sys.schema_unused_indexes;  -- 查看未使用索引
```

## 六、索引的使用

### 6.1验证索引效率

```sql
-- G：表示反转查询
select * from tb_sku where id = 1\G;

--在未建立索引之前，执行如下SQL语句，查看SQL的耗时。
SELECT * FROM tb_sku WHERE sn = '100000003145001';

--针对字段创建索引
create index idx_sku_sn on tb_sku(sn) ;

--然后再次执行相同的SQL语句，再次查看SQL的耗时。
SELECT * FROM tb sku WHER sn ='100000003145001';
```

1.即使有1000w的数据,根据id进行数据查询,性能依然很快，因为**主键id是默认有索引**的。

2.到根据sn字段进行查询，查询返回了一条数据，结果耗时 20.78sec，就是因为sn没有索引，而造成查询效率很低。

3.sn字段建立了索引之后0.01sec，查询性能大大提升。建立索引前后，查询耗时都不是一个数量级的。



### 6.2索引使用原则

如果索引了多列（联合索引），要遵守最左前缀法则。

最左前缀法则指的是查询**从索引的最左列开始， 并且不跳过索引中的列。** **如果跳跃某一列，索引将会部分失效**(后面的字段索引失效)。

1. **左前缀原则**：联合索引`(a,b,c)`有效组合：

   - ✅ WHERE a=1 AND b>2
   - ✅ WHERE a=1 ORDER BY b
   - ❌ WHERE b=2 AND c=5（**最左列为空，索引全部失效**）
   - ❌ WHERE a=1 AND c=3（**跳过中间，索引部分失效**）

2. **字段顺序策略**：

   - 高区分度字段在前（如用户ID）
   - 等值查询字段在前，范围字段在后
   - 常用排序字段可放在索引末尾

   

   ### 面试题： 

   当执行SQL语句:

    explain select * from tb_user where age = 31 and  status = '0' and profession = '软件工程';时，是否满足最左前缀法则，走不走上述的联合索引，索引长度？（在 tb_user 表中，有一个联合索引，这个联合索引涉及到三个字段，顺序分别为：profession， age，status。）

   是完全满足最左前缀法则的，索引长度54，联合索引是生效的。

   **注意 ： 最左前缀法则中指的最左边的列，是指在查询时，联合索引的最左边的字段(即是 第一个字段)必须存在，与我们编写SQL时，条件编写的先后顺序无关。**（与顺序无关，只看最左字段是否存在）

   

   

### 6.3 常见失效场景

 （1）范围查询 (**范围查询右边的列将会失效**)

```sql
--联合索引中，出现范围查询(>,<)，范围查询右侧的列索引失效。
explain select * from tb_user where profession = '软件工程' and age > 30 and status  = '0';
```

当范围查询使用> 或 < 时，走联合索引了，但是索引的长度为49，就说明范围查询右边的status字段是没有走索引的。(**范围查询右边的列将会失效**)

规避方案：在业务允许的情况下，尽可能的使用类似于 >= 或 <= 这类的范围查询，而避免使用 > 或 <

（加上等号可以具体定位到叶子链表中的具体节点，然后顺着往后遍历就可以了，不加等号，如果后面列索引不失效会出现很多重复的情况，遍历耗时更多）



（2）索引列运算（在索引列上进行运算操作， 索引将失效）

```sql
--不要在索引列上进行运算操作， 否则索引将失效。
--phone字段为单列索引。
--A. 当根据phone字段进行等值匹配查询时, 索引生效。
 explain select * from tb_user where phone = '17799990015';
-- 当根据phone字段进行函数运算操作之后，索引失效。
 explain  select  *  from  tb_user  where  substring(phone,10,2) = '15';
```



（3） 字符串不加引号

字符串类型字段使用时，不加引号，索引将失效。

```sql
-- 案例：隐式类型转换
SELECT * FROM user WHERE phone = 13800138000; -- phone是varchar类型
```

如果字符串不加单引号，对于查询结果，没什么影响，但是数据库存在隐式类型转换，索引将失效。



 (4)模糊查询

如果仅仅是尾部模糊匹配，索引不会失效。如果是头部模糊匹配，索引失效。

```sql
--生效
explain  select  *  from  tb_user  where  profession like '软件%';
--不生效
explain  select  *  from  tb_user  where  profession like '%工程';
--不生效
explain  select  *  from  tb_user  where  profession like '%工%';
```

在like模糊查询中，在关键字后面加%，索引可以生效。而如果在关键字 前面加了%，索引将会失效。

 （5）or连接条件

用or分割开的条件， 如果**or前的条件中的列有索引，而后面的列中没有索引**，那么涉及的索引都不会被用到。

```sql
-- 案例：OR混合条件
SELECT * FROM product WHERE price=99 OR stock>100; -- stock无索引,整体索引不生效
```

（6）数据分布影响

  如果MySQL评估使用索引比全表更慢，则不使用索引。

```sql
select * from tb_user where phone >= '17799990005';
select * from tb_user where phone >= '17799990015';
```

经过测试我们发现，相同的SQL语句，只是传入的字段值不同，最终的执行计划也完全不一样，这是为什么呢？

**这是因为MySQL在查询时，会评估使用索引的效率与走全表扫描的效率，如果走全表扫描更快，则放弃索引，走全表扫描。因为索引是用来索引少量数据的，如果通过索引查询返回大批量的数据，则还不如走全表扫描来的快，此时索引就会失效。**

```sql
explain select * from tb_user where profession is null;--走索引
explain select * from tb_user where profession is not null;--走全表
```

我们做一个操作将profession字段值全部更新为null。

然后，再次执行上述的两条SQL，查看SQL语句的执行计划。 

最终我们看到，一模一样的SQL语句，先后执行了两次，结果查询计划是不一样的，为什么会出现这种现象，这是和数据库的数据分布有关系。查询时MySQL会评估，走索引快，还是全表扫描快，如果全表扫描更快，则放弃索引走全表扫描。 因此，**is null 、is not null是否走索引，得具体情况具体分析，并不是固定的**。

（7）SQL提示

SQL提示，是优化数据库的一个重要手段，简单来说，就是在SQL语句中加入一些人为的提示来达到优 化操作的目的。 

1). use index ： 建议MySQL使用哪一个索引完成此次查询（仅仅是建议，mysql内部还会再次进行评估）。 explain select * from tb_user use index(idx_user_pro) where profession = '软件工程'; 

2). ignore index ： 忽略指定的索引。 

explain select * from tb_user ignore index(idx_user_pro) where profession = '软件工程'; 

3). force index ： 强制使用索引。 

explain select * from tb_user force index(idx_user_pro) where profession = '软件工程';

## 七、高级索引策略

### 7.1 覆盖索引

尽量使用覆盖索引，减少select *。 那么什么是覆盖索引呢？ 

覆盖索引是指查询使用了索引，并且需要返回的列，在该索引中已经全部能够找到 。

```sql
-- 原始SQL（需回表）
SELECT * FROM user WHERE age BETWEEN 20 AND 30;

-- 优化为覆盖索引
ALTER TABLE user ADD INDEX idx_age_name (age, name);
SELECT name, age FROM user WHERE age BETWEEN 20 AND 30; -- Using index
```
![](https://i-blog.csdnimg.cn/direct/a35a26ff587e4ce4be7cac87139c433f.png)



回表：先从二级索引去查，拿到id后再根据id去聚集索引加载具体的数据的过程。
![](https://i-blog.csdnimg.cn/direct/bd77cd88b7a24c9786ee8ad8701a49e6.png)
![](https://i-blog.csdnimg.cn/direct/32bf459551054bfeb3af71ab4f9cae53.png)
![](https://i-blog.csdnimg.cn/direct/5865ee5954e840d0aae942c9fc61279c.png)


这就是为什么要尽量减少使用select *，因为使用selcet *很容易造成回表查询，导致查询性能降低。

### 思考题：

一张表, 有四个字段(id, username, password, status), 由于数据量大, 需要对 以下SQL语句进行优化, 

该如何进行才是**最优方案**: 

select id,username,password from tb_user where username =  'itcast'; 

答案: 针对于 **username, password建立联合索引**（id，在查二级表时可以获取。）, sql为:

 create index  idx_user_name_pass on tb_user(username,password);  

这样可以避免上述的SQL语句，在查询的过程中，出现回表查询。

### 7.2 前缀索引

当字段类型为字符串（varchar，text，longtext等）时，有时候需要索引**很长的字符串**，这会让 索引变得很大，查询时，浪费大量的磁盘IO， 影响查询效率。此时可以只将字符串的一部分前缀，建 立索引，这样可以大大节约索引空间，从而提高索引效率。

```sql
--语法 n指定字符串的前几个字符建立索引
 create index  idx_xxxx on table_name(column(n)) ;
```

- 前缀长度

可以根据索引的选择性来决定，而选择性是指不重复的索引值（基数）和数据表的记录总数的比值， 索引选择性越高则查询效率越高， 唯一索引的选择性是1，这是最好的索引选择性，性能也是最好的。

```sql
select  count(distinct email) / count(*)   from  tb_user ;
select  count(distinct substring(email,1,5)) / count(*)  from  tb_user ;
```

 3). 前缀索引的查询流程

![](https://i-blog.csdnimg.cn/direct/c903ac719e05406e9590dd2d30b41975.png)


### 7.2 单列索引与联合索引（组合索引）

单列索引：即一个索引只包含单个列。

联合索引：即一个索引包含了多个列。

我们先来看看 tb_user 表中目前的索引情况: 
![](https://i-blog.csdnimg.cn/direct/7738ba2019c54d93b859ca926511be1d.png)



在查询出来的索引中，既有单列索引，又有联合索引。

接下来，我们来执行一条SQL语句，看看其执行计划：
![](https://i-blog.csdnimg.cn/direct/a574c7853a4b459e9d8cd73524591f02.png)



通过上述执行计划我们可以看出来，在and连接的两个字段 phone、name上都是有单列索引的，但是 最终mysql只会选择一个索引，也就是说，只能走一个字段的索引，此时是会回表查询的。 紧接着，我们再来创建一个phone和name字段的联合索引来查询一下执行计划。 

```sql
 create unique index idx_user_phone_name on tb_user(phone,name);
```

![](https://i-blog.csdnimg.cn/direct/ed09d14f42ae48388039f8f6a2ba5a29.png)


**在业务场景中，如果存在多个查询条件，考虑针对于查询字段建立索引时，建议建立联合索引， 而非单列索引。**

如果查询使用的是联合索引，具体的结构示意图如下：

![](https://i-blog.csdnimg.cn/direct/485b2ba163a147fbb37f450c2af7ce69.png)


先按phone进行排序，然后再根据name字段进行排序。

单列索引很容易造成回表查询。

## 八、索引设计原则

### 8.1索引设计原则（重点）

1). 针对于**数据量较大**，且**查询比较频繁**的表建立索引。

2). 针对于常作为**查询条件（where）、排序（order by）、分组（group by）**操作的字段建立索引。

3). 尽量选择**区分度高的列作为索引**，尽量建立唯一索引，区分度越高，使用索引的效率越高。 

4). 如果是字符串类型的字段，**字段的长度较长**，可以针对于字段的特点，建立**前缀索引**。

5). **尽量使用联合索引，减少单列索引**，查询时，联合索引很多时候可以覆盖索引，节省存储空间，避免回表，提高查询效率。 

6). 要控制索引的数量，**索引并不是多多益善**，索引越多，维护索引结构的代价也就越大，会影响增删改的效率。 7).**如果索引列不能存储NULL值，请在创建表时使用NOT NULL约束它**。当优化器知道每列是否包含 NULL值时，它可以更好地确定哪个索引最有效地用于查询。

### 8.2 索引选择策略

| 场景         | 推荐方案                 | 案例说明             |
| ------------ | ------------------------ | -------------------- |
| 频繁范围查询 | 联合索引（范围字段在后） | WHERE a=1 AND b>10   |
| 多字段排序   | 联合索引顺序匹配排序     | ORDER BY c,b,a       |
| 高并发写入   | 减少索引数量             | 日志型表保留必要索引 |

## 九、实战优化案例

### 案例：电商订单查询优化

```sql
-- 原始SQL（执行时间2.1s）
SELECT * FROM orders 
WHERE user_id=1001 
  AND status='paid'
  AND create_time BETWEEN '2023-01-01' AND '2023-06-30'
ORDER BY amount DESC
LIMIT 10;

-- 优化步骤：
1. 添加联合索引：
ALTER TABLE orders ADD INDEX idx_user_status_time (user_id, status, create_time);

2. 改写SQL：
SELECT * FROM orders 
WHERE user_id=1001 
  AND status='paid'
  AND create_time >= '2023-01-01' 
  AND create_time < '2023-07-01'  -- 避免函数转换
ORDER BY amount DESC
LIMIT 10;

-- 执行时间降至0.03s  
```

### 附录：索引管理命令速查

```sql
-- 查看索引大小
SELECT 
  table_name AS `Table`,
  index_name AS `Index`,
  ROUND(stat_value * @@innodb_page_size / 1024 / 1024, 2) `Size(MB)`
FROM 
  mysql.innodb_index_stats
WHERE 
  database_name = 'your_db'
  AND stat_name = 'size';

-- 监控索引使用
SELECT * FROM sys.schema_index_statistics;   
```

## 总结

1.索引概述

索引是高效获取数据的数据结构;

2.索引结构

B+Tree
Hash

3．索引分类

主键索引、唯一索引、常规索引、全文索引
聚集索引、二级索引

4.索引语法

create [unique ]index <xxx on xxx(xxx);
show index from xxxx ;
drop index xxx on xxxx;

5．SQL性能分析

执行频次、慢查询日志、profile、explain

6．索引使用

联合索引
索引失效
SQL提示
覆盖索引
前缀索引
单列/联合索引

7.索引设计原则

表
字段
索引

> **最佳实践总结**：
>
> 1. 理解B+Tree的`分层存储`特性，合理设计索引顺序
> 2. 善用`EXPLAIN`分析执行计划，重点关注type和Extra字段
> 3. 对高频查询优先考虑`覆盖索引`，减少回表操作
> 4. 定期使用`慢查询日志`定位性能瓶颈
> 5. 遵循`最左前缀原则`设计联合索引
> 6. 注意隐式的`类型转换`和`函数调用`导致索引失效
> 7. 在`查询性能`与`写入开销`之间找到平衡点
