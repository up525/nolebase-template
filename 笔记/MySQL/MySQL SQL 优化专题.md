---
comment: true
tags:
  - 知识领域/文档工程
titleTemplate: 记录回忆，知识和畅想的地方
---
# MySQL SQL 优化专题

## 1. 插入数据优化

```sql
-- 普通插入（不推荐）
INSERT INTO tb_user VALUES(1,'tom');
INSERT INTO tb_user VALUES(2,'cat');
INSERT INTO tb_user VALUES(3,'jerry');

-- 优化方案1：批量插入（推荐，不建议超过1000条，500-1000较为合适）
INSERT INTO tb_user VALUES(1,'tom'), (2,'cat'), (3,'jerry');

-- 优化方案2：手动事务提交（适用于大数据量）
start transaction;
INSERT INTO tb_user VALUES(1,'tom');
INSERT INTO tb_user VALUES(2,'cat');
commit;

-- 优化方案3：主键顺序插入（减少页分裂）
-- 有序ID：1,2,3,4... 
-- 无序ID：3,1,4,2...

-- 优化方案4：LOAD命令（百万级数据）
-- 客户端连接服务端时，加上参数  -–local-infile
mysql –-local-infile  -u  root  -p
-- 设置全局参数local_infile为1，开启从本地加载文件导入数据的开关
set  global  local_infile = 1;
-- 执行load指令将准备好的数据，加载到表结构中
-- 语法：LOAD DATA LOCAL INFILE '文件路径' INTO TABLE 表名; 
load  data  local  infile  '/root/sql1.log'  into  table  tb_user  fields  terminated  by  ','  lines  terminated  by  '\n' ; 
```

**原理说明**：

- 批量插入减少事务提交次数
- 顺序插入可减少页分裂概率
- LOAD指令比INSERT快约20倍

## 2. 主键优化

**（1）数据组织方式**：

在InnoDB存储引擎中，表数据都是根据主键顺序组织存放的，这种存储方式的表称为索引组织表 (index organized table IOT)。

- InnoDB采用B+树索引，数据存储在叶子节点
- 页分裂（离散插入导致）和页合并（删除数据后触发）

![](https://i-blog.csdnimg.cn/direct/c299061d650040fbb093edf440cee4ec.png)


行数据，都是存储在聚集索引的叶子节点上的。InnoDB的逻辑结构图：

![](https://i-blog.csdnimg.cn/direct/e471f66ddf994124950101b3a4da3a03.png)


在InnoDB引擎中，数据行是记录在逻辑结构 page 页中的，而每一个页的大小是固定的，默认16K。 那也就意味着， 一个页中所存储的行也是有限的，如果插入的数据行row在该页存储不小，将会存储 到下一个页中，页与页之间会通过指针连接。

（2). **页分裂**

页可以为空，也可以填充一半，也可以填充100%。每个页至少包含了2行数据（只有一行数据就等于退化成链表了）(如果一行数据过大，会行溢出)，根据主键排列。

A. 主键顺序插入效果

①. 从磁盘中申请页， 主键顺序插入

![](https://i-blog.csdnimg.cn/direct/528a5f71522d410bb592236915038aa9.png)


②. 第一个页没有满，继续往第一页插入

![](https://i-blog.csdnimg.cn/direct/fa22b007ed3b485092bcda61382e808d.png)


③. 当第一个也写满之后，再写入第二个页，页与页之间会通过指针连接

![](https://i-blog.csdnimg.cn/direct/b1752f6e4186444ea80655acb39a1ab2.png)


④. 当第二页写满了，再往第三页写入

![](https://i-blog.csdnimg.cn/direct/eaa506216fbf4d58b8130e68097e501d.png)




B. 主键乱序插入效果

①. 加入1#,2#页都已经写满了，存放了如图所示的数据

![](https://i-blog.csdnimg.cn/direct/fc753b4d5bd44d1da403ed6c487d0046.png)

②. 此时再插入id为50的记录，我们来看看会发生什么现象 ？

会再次开启一个页，写入新的页中吗？

![](https://i-blog.csdnimg.cn/direct/da25ecd0a4ac400f92269480f76396c2.png)


不会。因为，索引结构的叶子节点是有顺序的。按照顺序，应该存储在47之后。

![](https://i-blog.csdnimg.cn/direct/ffb0a2f091034283b308f584bdb567ba.png)


但是47所在的1#页，已经写满了，存储不了50对应的数据了。 那么此时会开辟一个新的页 3#。

![](https://i-blog.csdnimg.cn/direct/2df9e2ec29d8444e98892e3e5f31dc1c.png)


但是并不会直接将50存入3#页，而是会将1#页后一半的数据，移动到3#页，然后在3#页，插入50。

![](https://i-blog.csdnimg.cn/direct/6e970bf647de4228ada82b4f3b2c1bab.png)


移动数据，并插入id为50的数据之后，那么此时，这三个页之间的数据顺序是有问题的。 1#的下一个 页，应该是3#， 3#的下一个页是2#。 所以，此时，需要重新设置链表指针。（连接过程类似双向链表的插入过程）
![](https://i-blog.csdnimg.cn/direct/027b96faec0f4a7bbf24bfe211684e38.png)



**上述的这种现象，称之为 "页分裂"，是比较耗费性能的操作。**

3). 页合并

目前表中已有数据的索引结构(叶子节点)如下：
![](https://i-blog.csdnimg.cn/direct/178a00d8e003433b8c930b677f8b20a6.png)



当我们对已有数据进行删除时，具体的效果如下: 

当删除一行记录时，**实际上记录并没有被物理删除，只是记录被标记（flaged）**为删除并且它的空间 变得允许被其他记录声明使用。

![](https://i-blog.csdnimg.cn/direct/77ec1a7a7b2e41fabb2b01625590bfb5.png)


当我们继续删除2#的数据记录
![](https://i-blog.csdnimg.cn/direct/42bdccbfed534a208a7613ec1077a6a7.png)



**当页中删除的记录达到 MERGE_THRESHOLD（默认为页的50%）**，InnoDB会开始寻找最靠近的页（前或后）看看是否可以将两个页合并以优化空间使用。
![](https://i-blog.csdnimg.cn/direct/053a9abc81b842f48561da1ec1d89308.png)



删除数据，并将页合并之后，再次插入新的数据21，则直接插入3#页
![](https://i-blog.csdnimg.cn/direct/2a23ebe6324d487d9257cfc5842c0512.png)


这个里面所发生的合并页的这个现象，就称之为 "页合并"。

知识小贴士： MERGE_THRESHOLD（threshold：阈值）：合并页的阈值，可以自己设置，在创建表或者创建索引时指定。

**4). 主键设计原则**

1. 满足业务需求情况下，尽量`降低主键长度`
2. 插入数据时尽量选择`顺序插入`，使用`AUTO_INCREMENT`主键
3. 尽量不要使用`UUID`（无序，插入可能产生页分裂现象，影响性能）或其他`自然主键`（如身份证号：长度比较长，检索时会浪费大量的磁盘IO时间）
4. 避免对主键进行`修改`（修改主键还需要修改对应的索引）

## 3. ORDER BY 优化

MySQL的排序，有两种方式：

**Using filesort** : **通过表的索引或全表扫描**，读取满足条件的数据行，然后在排序缓冲区sort  buffer中完成排序操作，所有不是通过索引直接返回排序结果的排序都叫 FileSort 排序。

 **Using index** : 通过有序索引**顺序扫描直接返回**有序数据，这种情况即为 using index，不需要 额外排序，操作效率高。 

 对于以上的两种排序方式，**Using index的性能高，而Using filesort的性能低**，我们在优化排序 操作时，尽量要优化为 Using index。

```sql
-- 需要优化的查询（出现Using filesort）
explain select  id,age,phone from tb_user order by age ;
explain select  id,age,phone from tb_user order by age, phone ;
--由于 age, phone 都没有索引，所以此时再排序时，出现Using filesort， 排序性能较低。

-- 创建索引
CREATE INDEX idx_age_phone ON tb_user(age, phone);

--创建索引后，根据age, phone进行升序排序
-- 优化后查询（Using index）
explain select  id,age,phone from tb_user order by age, phone ;
--建立索引之后，再次进行排序查询，就由原来的Using filesort， 变为了 Using index，性能就是比较高的了。
```

```sql
--根据age, phone进行降序一个升序，一个降序
explain select  id,age,phone from tb_user order by age desc , phone desc ;
--因为创建索引时，如果未指定顺序，默认都是按照升序排序的，而查询时，一个升序，一个降序，此时
--就会出现Using filesort。
```

为了解决上述的问题，我们可以创建一个索引，这个联合索引中 age 升序排序，phone 倒序排序。

创建联合索引(age 升序排序，phone 倒序排序)

```sql
 create  index  idx_user_age_phone_ad  on  tb_user(age asc ,phone desc);
```

优化后查询（Using index）。

升序/降序联合索引结构图示:
![](https://i-blog.csdnimg.cn/direct/ddd4b65b2c2c4f0c9f56c1594c631624.png)

![](https://i-blog.csdnimg.cn/direct/40ce33d79f5742d4826359aeb9fc42f6.png)






```sql
--根据phone，age进行升序排序，phone在前，age在后。（易错细节）
explain select  id,age,phone from tb_user order by phone , age;
--排序时,也需要满足最左前缀法则,否则也会出现 filesort。因为在创建索引的时候， age是第一个
--字段，phone是第二个字段，所以排序时，也就该按照这个顺序来，否则就会出现 Using filesort。
```

**排序类型**：

- Using index：直接通过索引返回数据（性能最佳）
- Using filesort：需要将结果集加载到内存排序（需要优化）

**order by优化原则:**

 A. 根据排序字段建立合适的索引，多字段排序时，也遵循最左前缀法则。(**where 后的连表条件只要存在即可，无所谓顺序，但order by后面的书写有顺序要求**)

 B. 尽量使用覆盖索引。（减少使用select  * ，不用回表）

 C. 多字段排序, 一个升序一个降序，此时需要注意联合索引在创建时的规则（ASC/DESC）。

 D. 如果不可避免的出现filesort，大数据量排序时，可以适当增大排序缓冲区大小  sort_buffer_size(默认256k)。

## 4. GROUP BY 优化

```sql
-- 未优化（出现Using temporary）
EXPLAIN SELECT profession, COUNT(*) FROM tb_user 
GROUP BY profession;

-- 创建索引后优化
CREATE INDEX idx_pro_age_sta ON tb_user(profession,age,status);
EXPLAIN SELECT profession, COUNT(*) FROM tb_user 
GROUP BY profession; -- 使用索引
```

**优化方法**：

 A. 在分组操作时，可以通过索引来提高效率。

 B. 分组操作时，索引的使用也是满足最左前缀法则的。

## 5. LIMIT 优化

在数据量比较大时，如果进行limit分页查询，在查询时，越往后，分页查询效率越低。

我们一起来看看执行limit分页查询耗时对比：
![](https://i-blog.csdnimg.cn/direct/4165addd56d14807aa29da4dc7be9b94.png)



通过测试我们会看到，越往后，分页查询效率越低，这就是分页查询的问题所在。 因为，当在进行分页查询时，如果执行 limit 2000000,10 ，此时需要MySQL排序前2000010 记 录，仅仅返回 2000000 - 2000010 的记录，其他记录丢弃，查询排序的代价非常大 。

（因为叶子排序是双链表，要依次遍历，越向后时间越长。）

优化思路: 一般分页查询时，通过创建 覆盖索引 能够比较好地提高性能，可以通过**覆盖索引加子查询**形式进行优化。

```sql
explain   select  *  from  tb_sku  t  ,  (select  id  from  tb_sku  order  by  id 
limit  2000000,10)  a  where t.id  =  a.id;
```



```sql
-- 低效写法（耗时随偏移量增加）
SELECT * FROM tb_sku LIMIT 9000000,10;

-- 优化方案：记录上次查询的最大ID
SELECT * FROM tb_sku WHERE id > 9000000 LIMIT 10;

-- 子查询优化（需覆盖索引）
SELECT * FROM tb_sku t, 
(SELECT id FROM tb_sku ORDER BY id LIMIT 9000000,10) a 
WHERE t.id = a.id;
```

**优化原理**：

- 避免全表扫描，使用索引覆盖
- 使用ID分段查询替代大偏移量

## 6. COUNT 优化

```sql
 select  count(*)  from  tb_user ;
```

我们发现，如果数据量很大，在执行count操作时，是非常耗时的。

MyISAM 引擎把一个表的总行数存在了磁盘上，因此执行 count(*) 的时候会直接返回这个数，效率很高； 但是如果是带条件的count，MyISAM也慢。 

InnoDB 引擎就麻烦了，它执行 count(*) 的时候，需要把数据一行一行地从引擎里面读出来，然后累积计数 。

如果说要大幅度提升InnoDB表的count效率，主要的优化思路：

**自己计数**(可以借助于redis这样的数 据库进行,但是如果是带条件的count又比较麻烦了)。



####  count用法  

count() 是一个聚合函数，对于返回的结果集，一行行地判断，如果 count 函数的参数不是  NULL，累计值就加 1，否则不加，最后返回累计值。

 用法：count（*）、count（主键）、count（字段）、count（数字）
![](https://i-blog.csdnimg.cn/direct/82cc6b9098da4aa698c32c966c3b0666.png)



```sql
--按照效率排序的话，所以尽量使用count(*)，因为专门做了优化。
count(字段)(需要做判断是否为空)< count(主键 id) < count(1) ≈ count(*)

-- 统计有效数据条数
SELECT COUNT(1) FROM tb_user;  -- 推荐写法
SELECT COUNT(*) FROM tb_user;  -- 官方优化写法
```

**不同COUNT区别**：

- COUNT(字段)：统计不为NULL的记录数
- COUNT(主键)：遍历主键索引
- COUNT(1)：不取值直接累加1
- COUNT(*)：MySQL优化过的特殊计数器

## 7. UPDATE 优化

回忆：InnoDB的三大特性：事务，外键，行级锁

​			start transaction; 或者是begin来开启事务；

我们主要需要注意一下update语句执行时的注意事项。

```sql
update  course  set  name = 'javaEE'  where  id  =  1 ;
```
![](https://i-blog.csdnimg.cn/direct/5b47073ddcd74c028e80859bac43526b.png)



当我们在执行更新的SQL语句时，会锁定id为1这一行的数据，然后事务提交之后，行锁释放。

但是当我们在执行如下SQL时。

```sql
update course set name = 'SpringBoot' where name = 'PHP' ;
```
![](https://i-blog.csdnimg.cn/direct/eaf97b2bde06477f84b7def98c9840da.png)



**name这个字段没有索引，此时加的就不再是行锁了，而是表锁。一旦锁表了，我们的并发性能就会降低！！！**

当我们开启多个事务，在执行上述的SQL时，我们发现行锁升级为了表锁。 导致该update语句的性能大大降低。

**InnoDB的行锁是针对索引加的锁，不是针对记录加的锁 ,并且该索引不能失效，否则会从行锁 升级为表锁 。**

```sql
-- 使用索引字段更新（行级锁）
UPDATE tb_user SET name = 'zhangsan' WHERE id = 1;

-- 无索引更新（表级锁，需要避免！）
UPDATE tb_user SET name = 'lisi' WHERE name = 'wangwu';
```

**优化重点**：

- 更新条件必须`走索引`，避免行锁升级为表锁
- 事务要及时提交，减少锁持有时间

# 总结

| 优化类型     | 核心方法                                         | 典型案例                      |
| ------------ | ------------------------------------------------ | ----------------------------- |
| **插入优化** | 批量插入+手动事务提交+主键顺序插入               | 万级数据使用LOAD DATA         |
| **主键优化** | 自增主键+避免修改+尽量短                         | UUID导致页分裂问题            |
| **ORDER BY** | 尽量建立覆盖索引+避免filesort                    | 多字段排序注意索引顺序        |
| **GROUP BY** | 利用索引减少临时表（多字段分组满足最左前缀法则） | 分组字段建立联合索引          |
| **LIMIT**    | 覆盖索引+子查询（使用ID分段替代大偏移量）        | 百万级分页优化方案            |
| **COUNT**    | 优先使用COUNT(*)或COUNT(1)                       | 统计全表数据时避免COUNT(字段) |
| **UPDATE**   | WHERE条件必须走索引                              | 无索引更新导致表锁问题        |

通过以上优化手段，通常可以使MySQL查询性能提升1-3个数量级，特别是在大数据量场景下效果尤为显著。实际优化中需要结合EXPLAIN执行计划进行分析，针对性优化关键瓶颈点。
