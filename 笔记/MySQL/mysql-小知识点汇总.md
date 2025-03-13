#  SQL通用语法

> 1). SQL语句可以单行或多行书写，以分号结尾。
> 
> 2). SQL语句可以使用空格/缩进来增强语句的可读性。
> 
> 3). MySQL数据库的SQL语句不区分大小写，关键字建议使用大写。
> 
> 4). 注释： 单行注释：--  注释内容 或 # 注释内容 多行注释：/* 注释内容 */

# SQL分类 

SQL语句，根据其功能，主要分为四类：DDL、DML、DQL、DCL。

![](https://i-blog.csdnimg.cn/direct/1f5fc69164e04219b42eaf120cb32776.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​编辑

#  数据类型

MySQL中的数据类型有很多，主要分为三类：数值类型、字符串类型、日期时间类型。

## （1）数值类型

|   |   |   |   |   |   |
|---|---|---|---|---|---|
|分类|类型|大小|有符号(SIGNED)范围|无符号(UNSIGNED)范围|描述|
|数值类型|TINYINT|1 byte|(-128，127)|(0，255)|小整数值|
|SMALLINT|2 bytes|(-32768，32767)|(0，65535)|大整数值|
|MEDIUMINT|3 bytes|(-8388608，8388607)|(0，16777215)|大整数值|
|INT或INTEGER|4 bytes|(-2147483648，2147483647)|(0，4294967295)|大整数值|
|BIGINT|8 bytes|(-2^63，2^63-1)|(0，2^64-1)|极大整数值|
|FLOAT|4 bytes|(-3.402823466 E+38，3.402823466351 E+38)|0 和 (1.175494351 E-38，3.402823466 E+38)|单精度浮点数值|
|DOUBLE|8 bytes|(-1.7976931348623157 E+308，1.7976931348623157 E+308)|0 和 (2.2250738585072014 E-308，1.7976931348623157 E+308)|双精度浮点数值|
|DECIMAL||依赖于M(精度)和D(标度)的值|依赖于M(精度)和D(标度)的值|小数值(精确定点数)|

![](https://i-blog.csdnimg.cn/direct/413918fe3aac464b8a53bbf011ded5dc.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​编辑

## （2）字符串类型

|   |   |   |   |   |   |
|---|---|---|---|---|---|
|**分类**|**类型**|****大小****|**描述**|||
|字符串类型|CHAR|0-255 bytes|定长字符串|char(10) -----------> 性能好|用户名  username  varchar(50)|
|VARCHAR|0-65535 bytes|变长字符串|varchar(10) ---------> 性能较差|性别  gender  char(1)|
|TINYBLOB|0-255 bytes|不超过255个字符的二进制数据|||
|TINYTEXT|0-255 bytes|短文本字符串|||
|BLOB|0-65 535 bytes|二进制形式的长文本数据|||
|TEXT|0-65 535 bytes|长文本数据|||
|MEDIUMBLOB|0-16 777 215 bytes|二进制形式的中等长度文本数据|||
|MEDIUMTEXT|0-16 777 215 bytes|中等长度文本数据|||
|LONGBLOB|0-4 294 967 295 bytes|二进制形式的极大文本数据|||
|LONGTEXT|0-4 294 967 295 bytes|极大文本数据|||

char 与 varchar 都可以描述字符串，char是定长字符串，指定长度多长，就占用多少个字符，和 字段值的长度无关 。而varchar是变长字符串，指定的长度为最大占用长度 。相对来说，char的性 能会更高些。

![](https://i-blog.csdnimg.cn/direct/524760ebd7754909be8e4e79f223e0a4.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​编辑

##  （3）日期类型

|   |   |   |   |   |   |
|---|---|---|---|---|---|
|****分类****|**类型**|****大小****|**范围**|**格式**|**描述**|
|日期类型|DATE|3|1000-01-01 至 9999-12-31|YYYY-MM-DD|日期值|
|TIME|3|-838:59:59 至 838:59:59|HH:MM:SS|时间值或持续时间|
|YEAR|1|1901 至 2155|YYYY|年份值|
|DATETIME|8|1000-01-01 00:00:00 至 9999-12-31 23:59:59|YYYY-MM-DD HH:MM:SS|混合日期和时间值|
|TIMESTAMP|4|1970-01-01 00:00:01 至 2038-01-19 03:14:07|YYYY-MM-DD HH:MM:SS|混合日期和时间值，时间戳|

![](https://i-blog.csdnimg.cn/direct/8204abb017d441208c25395b53bb3f6d.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​编辑

# 函数

MySQL中的函数主要分为以下四类： 字符串函数、数值函数、日期函数、流程函数。

## 字符串函数

MySQL中内置了很多字符串函数，常用的几个如下：

![](https://i-blog.csdnimg.cn/direct/cc08f8270bbc450b90ee83e26fc459cb.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​编辑

## 数值函数

常见的数值函数如下：

![](https://i-blog.csdnimg.cn/direct/7b9a2d9372d24efa82015b759d2f28b1.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​编辑

## 日期函数

常见的日期函数如下： 

![](https://i-blog.csdnimg.cn/direct/8df02609511c4a46a83feb992dc62321.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​编辑

流程函数

流程函数也是很常用的一类函数，可以在SQL语句中实现条件筛选，从而提高语句的效率。 

![](https://i-blog.csdnimg.cn/direct/4f0b80a541aa48839b6dd6e5731cb68f.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​编辑

# 约束

概念：约束是作用于表中字段上的规则，用于限制存储在表中的数据。

目的：保证数据库中数据的正确、有效性和完整性 。

分类:

![](https://i-blog.csdnimg.cn/direct/5a4989581286485f8b77516e6166a467.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​编辑

> **注意：约束是作用于表中字段上的，可以在创建表/修改表的时候添加约束。** 

## 外键约束

外键：用来让两张表的数据之间建立连接，从而保证数据的一致性和完整性。 

### 1). 添加外键

### ![](https://i-blog.csdnimg.cn/direct/e1fecc5104164396b121f715f14139b7.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​编辑

### 2). 删除外键

![](https://i-blog.csdnimg.cn/direct/59c1c8c1fa38423c9fc8fd706aec0515.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​编辑

### 3). 删除/更新行为

添加了外键之后，再删除父表数据时产生的约束行为，我们就称为删除/更新行为。具体的删除/更新行 为有以下几种：

![](https://i-blog.csdnimg.cn/direct/15e48e9ccf0842b98769109c801fa415.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​编辑

具体语法为:

# ![](https://i-blog.csdnimg.cn/direct/59933d9bf914422d969b9a0779c3d73e.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​编辑 聚合函数

##  1). 介绍

将一列数据作为一个整体，进行纵向计算 。

## 2). 常见的聚合函数

![](https://i-blog.csdnimg.cn/direct/4435893ffdaa4b0190aef0ab2ff31d11.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​编辑

## 3). 语法

SELECT 聚合函数(字段列表) FROM 表名 ;

注意 : NULL值是不参与所有聚合函数运算的。 

# 查询sql执行顺序 

![](https://i-blog.csdnimg.cn/direct/57e3f42be1db4a76b62c3542918ac6fe.png)![](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw== "点击并拖拽以移动")​编辑

  

​