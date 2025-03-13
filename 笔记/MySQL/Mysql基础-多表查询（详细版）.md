## 一、表的关系类型与适用场景
### 1. 一对一关系

**场景**：一个表的记录对应另一个表的唯一记录  
**案例**：用户表 + 用户详情表  

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE user_details (
    user_id INT PRIMARY KEY,
    address VARCHAR(100),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

### 2. 一对多关系

**场景**：主表的一条记录对应从表的多条记录
 **案例**：部门表 + 员工表

```sql
CREATE TABLE departments (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES departments(id)
);
```

### 3. 多对多关系

**场景**：两个表的记录可以相互对应多条记录
 **案例**：学生表 + 课程表（通过中间表实现）

```sql
CREATE TABLE students (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE courses (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE student_courses (
    student_id INT,
    course_id INT,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (course_id) REFERENCES courses(id)
);
    
```

------

## 二、连接方式与使用场景

### 1. 内连接（INNER JOIN）

**场景**：需要两表**同时存在匹配记录**的数据，相当于查询的是两张表的交集，不能查空。

```sql
-- 查询所有有部门的员工信息
SELECT e.name, d.name AS dept_name
FROM employees e
INNER JOIN departments d 
ON e.dept_id = d.id;

--等价写法(这种写法平时项目里用的更多)
SELECT e.name, d.name AS dept_name
FROM employees e，departments d 
ON e.dept_id = d.id;  
```

### 2. 左外连接（LEFT JOIN）

**场景**：保留左表所有记录，右表无匹配时显示NULL（相比右外，实际开发用的更多）

相当于查询的是两张表的交集，但是能查空

```sql
-- 查询所有员工（包括未分配部门的） 
SELECT e.name, d.name AS dept_name
FROM employees e
LEFT JOIN departments d 
ON e.dept_id = d.id;
--两表出现相同字段要起别名
```

### 3. 右外连接（RIGHT JOIN）

**场景**：保留右表所有记录，左表无匹配时显示NULL

```sql
-- 查询所有部门（包括没有员工的）
SELECT d.name AS dept_name, e.name
FROM employees e
RIGHT JOIN departments d 
ON e.dept_id = d.id;
```

#### 补充对比示例说明：

假设有以下两个表：

#### 员工表 (employees)

| id   | name | dept_id |
| ---- | ---- | ------- |
| 1    | 张三 | 101     |
| 2    | 李四 | NULL    |

#### 部门表 (departments)

| id   | dept_name |
| ---- | --------- |
| 101  | 技术部    |
| 102  | 市场部    |

------

#### 不同连接的结果差异：

```sql
-- 内连接（INNER JOIN）
SELECT e.name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.dept_id = d.id;
-- 结果：只有张三 + 技术部

    
-- 左连接（LEFT JOIN）
SELECT e.name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.dept_id = d.id;

-- 结果：
-- 张三 + 技术部
-- 李四 + NULL
```

------

### ❗ 关键区别：

- | 连接类型   | 是否要求右表有数据 | 是否保留左表所有数据 | 典型场景                                  |
  | ---------- | ------------------ | -------------------- | ----------------------------------------- |
  | INNER JOIN | **必须**           | 否                   | 查询「**完整关联信息**」                  |
  | LEFT JOIN  | 不必须             | **是**               | 查询「**左表全部**+**右表能关联的部分**」 |

------

### 🧠 易错点提醒：

1. **不要混淆「存在数据」和「匹配条件」**

   - 即使两表都有数据，但若 **不满足连接条件**，内连接也会过滤掉
   - 例如：员工表有 `dept_id=100`，部门表没有 `id=100` 的记录时，该员工不会出现在内连接结果中

2. **默认 JOIN 行为差异**

   ```sql
   -- 以下两种写法等价
   SELECT * FROM A INNER JOIN B ON A.id = B.a_id;--显式内连接
   SELECT * FROM A, B WHERE A.id = B.a_id; -- 隐式内连接
   ```

### 4. 全外连接/联合查询（FULL OUTER JOIN） 

**场景**：同时保留两表所有记录（MySQL会用到关键字 union或union all）

对于union查询，就是把多次查询的结果合并起来，形成一个新的查询结果集。

```sql
--将薪资低于5000的员工，和年龄大于50的员工全部查询出来。
--union all 包含重复数据
select * from emp where salary < 5000
union all
select * from emp where age > 50;
--union  去除重复数据
select*fromemp where salary< 5000
union
select * from emp where age > 50;
```

**tip：**	对于联合查询的多张表的列数必须保持一致，字段类型也需要保持一致。
			 union all会将全部的数据直接合并在一起，union会对合并之后的数据去重。

### 5. 交叉连接（CROSS JOIN）

**场景**：生成笛卡尔积，常用于组合场景

```sql
-- 生成颜色与尺寸的所有组合
SELECT colors.name, sizes.name
FROM colors
CROSS JOIN sizes;
```

### 6. 自连接（SELF JOIN）

**场景**：同一表内数据关联查询

**tip：自连接一定要起别名**

```sql
-- 查找员工的上级经理
SELECT e.name AS employee, m.name AS manager
FROM employees e
LEFT JOIN employees m 
ON e.manager_id = m.id; 
```

------

## 三、易错点与注意事项

1. **忘记关联条件导致笛卡尔积**

   ```sql
   -- 错误！缺少ON条件，将产生百万级数据
   SELECT * FROM employees, departments;  
   ```

2. **NULL值处理问题**

   ```sql
   -- 外连接后过滤条件应放在ON子句
   SELECT * 
   FROM A 
   LEFT JOIN B 
   ON A.id = B.a_id AND B.status = 1; -- ✔ 正确写法
   ```

3. **多次连接时的别名冲突**

   ```sql
   -- 必须为每个表指定唯一别名
   SELECT o.order_no, c1.name AS city_from, c2.name AS city_to
   FROM orders o
   LEFT JOIN cities c1 ON o.from_city = c1.id
   LEFT JOIN cities c2 ON o.to_city = c2.id;    
   ```

4. **连接顺序影响性能**

   ```sql
    -- 大表在前可能导致性能问题
   SELECT * 
   FROM huge_table  -- ✘ 大表在前
   INNER JOIN small_table ON ...  
   ```

------

## 四、总结

| 连接类型   | 适用场景              | 特点说明                 |
| ---------- | --------------------- | ------------------------ |
| INNER JOIN | 需要严格匹配的数据    | 结果集最小，性能最好     |
| LEFT JOIN  | 保留左表全部数据      | 常用于主表查询           |
| RIGHT JOIN | 保留右表全部数据      | 可用LEFT JOIN替代        |
| FULL JOIN  | 需要两表所有数据      | MySQL需用UNION模拟       |
| CROSS JOIN | 生成组合数据          | 谨慎使用，易产生大数据量 |
| SELF JOIN  | 层级关系/树形结构查询 | 必须使用别名             |

**最佳实践建议**：

1. 优先使用INNER JOIN，需要保留全部数据时再用外连接
2. 多表连接时，按数据量从小到大排列连接顺序
3. 始终为连接的表指定明确的别名
4. 复杂查询建议分步调试，先验证单表结果再组合
5. 超过3个表连接时，建议使用EXPLAIN分析执行计划  

# MySQL 子查询全面指南

## 目录

- [一、子查询类型与使用场景](#一子查询类型与使用场景)
- [二、不同子查询的SQL示例](#二不同子查询的sql示例)
- [三、易错点与注意事项](#三易错点与注意事项)
- [四、总结与最佳实践](#四总结与最佳实践)

---

## 一、子查询类型与使用场景
概念：SQL语句中嵌套SELECT语句，称为嵌套查询，又称子查询。

```sql
SELECT*FROM t1 WHERE column1=（SELECT column1 FROM t2);
```

**子查询外部的语句可以是INSERT/UPDATE/DELETE/SELECT的任何一个。**
根据子查询结果不同，分为：

> 1.标量子查询（子查询结果为单个值）
> 2.列子查询(子查询结果为一列)
> 3.行子查询(子查询结果为一行)
> 4.表子查询（子查询结果为多行多列)

**根据子查询位置，分为：WHERE之后、FROM之后、SELECT之后。**

### 1. 标量子查询

**特征**：子查询返回的结果是单个值（数字、字符串、日期等），最简单的形式，返回单个值（一行一列），这种子查询成为标量子查询。
**常用的操作符**：= <> > >= < <=
**场景**：在WHERE/SELECT/HAVING等位置作为条件值使用  

```sql
-- 查询高于平均工资的员工
--a.查询平均员工的工资（一行一列，即只有一条数据，所以称为标量子查询）
SELECT AVG(salary) FROM employees;
--b.查询高于平均员工工资的员工信息(>)
SELECT name, salary 
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees); -- 返回单个数值 
```

### 2. 列子查询

**特征**：返回单列多行数据
**常用的操作符**：IN、NOTIN、ANY、SOME、ALL
 **场景**： 子查询返回的结果是一列（可以是多行），这种子查询称为列子查询。
![](https://i-blog.csdnimg.cn/direct/5afb6db019e64bf0869a8be95061516e.png)
```sql
--1.查询销售部和市场部的所有员工信息
--a.查询销售部和市场部的部门ID
select id from dept where name='销售部' or name='市场部';
--b.根据部门id查询员工信息
select * from emp where dept_id in (select id from dept where name='销售部' or name='市场部');

--2.查询比财务部所有人工资都高的员工信息
--法一（使用max()函数）
--a. 查询财务部所有人员工资
--1.首先拿到财务部的id
select id from dept where name='财务部';
--2.根据财务部id，在员工表查询最高的财务部员工的工资
select max(salary) from emp where dept_id=(select id from dept where name='财务部');
--b.在员工表查询比比财务部所有人工资都高的员工信息
select * from emp where salary>(select max(salary) from emp where dept_id=(select id from dept where name='财务部'));

--法二（使用all关键字）
--a. 查询财务部所有人员工资
--1.首先拿到财务部的id
select id from dept where name='财务部';
--2.根据财务部id，在员工表查询财务部员工的工资
select salary from emp where dept_id=(select id from dept where name='财务部');
--b.在员工表查询比比财务部所有人工资都高的员工信息
select * from emp where salary>all(select salary from emp where dept_id=(select id from dept where name='财务部'));

--3.查询比研发部其中任意一人工资高的员工信息(相当于只要比工资的最小值大就可以,这里可以用some，any)
--a.查询研发部所有人工资
--1.首先拿到研发部的id
select id from dept where name='研发部';
--2.根据研发部id，在员工表查询研发部员工的工资
select salary from emp where dept_id=(select id from dept where name='研发部');
--b.比研发部其中任意一人工资高的员工信息
select * from emp where salary>any(select salary from emp where dept_id=(select id from dept where name='研发部'));
```

### 3. 行子查询
**特征**：子查询返回的结果是一行（可以是多列），这种子查询称为行子查询。
 **场景**：多条件同时比较 (c1,c2)=(c3,c4)  一行多列=一行多列（注意这里是单行数据所以才可以用=）
 **常用的操作符**：=、<>、IN、NOT IN
```sql
-- 查询与张三同部门同职位的员工
--a.查询张三的部门和职位信息
SELECT dept_id, position 
    FROM employees 
    WHERE name = '张三'
;
--b.查询与张三同部门同职位的员工
SELECT name 
FROM employees
WHERE (dept_id, position) = (
    SELECT dept_id, position 
    FROM employees 
    WHERE name = '张三'
);
```

### 4. 表子查询

**特征**：返回多行多列结果集，所以用关键字in(查询语句)
 **场景**：作为临时表参与连接查询

```sql
-- 查询各部门最高薪员工
--a.查询各个部门的最高薪水，并作为一张临时表存在
    SELECT dept_id, MAX(salary) AS max_salary
    FROM employees
    GROUP BY dept_id;
--b.查询各部门最高薪的员工信息
SELECT e.dept_id, e.name, e.salary
FROM employees e
INNER JOIN (
    SELECT dept_id, MAX(salary) AS max_salary
    FROM employees
    GROUP BY dept_id
) AS tmp 
ON e.dept_id = tmp.dept_id AND e.salary = tmp.max_salary; 

--1．查询与“鹿杖客”，“宋远桥”的职位和薪资相同的员工信息
--a．查询“鹿杖客”，"宋远桥”的职位和薪资
select job,salary from emp where name='鹿杖客'or name='宋远桥';
--b.查询与“鹿杖客”,“宋远桥”的职位和薪资相同的员工信息(多行多列所以这里要用in，不能用=)
select * from emp where(job,salary) in (select job,salary from emp where name='鹿杖客'orname='宋远桥');

--查询入职日期是"2006-01-01”之后的员工信息，及其部门信息
--a.入职日期是"2006-01-01”之后的员工信息,并作为一张临时表存在
select * from emp where entrydate >'2006-01-01';
--b．查询这部分员工，对应的部门信息;
select e.*，d.* from (select * from emp where entrydate >'2006-01-01') e left join dept d on e.dept_id =d.id;
```

### 5. 相关子查询

**特征**：子查询引用外层查询的字段
 **场景**：逐行处理关联数据

```sql
-- 查询工资高于部门平均的员工
SELECT name, salary, dept_id
FROM employees e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.dept_id = e1.dept_id  -- 引用外层字段
);  
```

### 6. EXISTS/NOT EXISTS

**特征**：检查子查询是否存在结果
 **场景**：存在性验证

```sql
-- 查询从未下单的客户
SELECT name 
FROM customers c
WHERE NOT EXISTS (
    SELECT 1 
    FROM orders o 
    WHERE o.customer_id = c.id
);   
```

------

## 二、不同子查询的SQL示例

### 1. 在SELECT中使用

```sql
-- 显示员工及其部门人数
SELECT 
    name,
    dept_id,
    (SELECT COUNT(*) 
     FROM employees e2 
     WHERE e2.dept_id = e1.dept_id) AS dept_total
FROM employees e1;
```

### 2. 在UPDATE中使用

```sql
-- 将技术部员工薪资提高10%
UPDATE employees
SET salary = salary * 1.1
WHERE dept_id = (
    SELECT id 
    FROM departments 
    WHERE dept_name = '技术部'
);
   
```

### 3. 在HAVING中使用

```sql
-- 查询订单数超过平均值的客户
SELECT customer_id, COUNT(*) AS order_count
FROM orders
GROUP BY customer_id
HAVING COUNT(*) > (
    SELECT AVG(order_count) 
    FROM (
        SELECT COUNT(*) AS order_count
        FROM orders
        GROUP BY customer_id
    ) tmp
);
   
```

------

## 三、易错点与注意事项

1. **性能陷阱**

   ```sql
   -- 错误：每行执行子查询导致性能低下
   SELECT name, 
       (SELECT COUNT(*) FROM orders 
        WHERE customer_id = c.id) AS order_count
   FROM customers c;
   -- ✅ 应改用LEFT JOIN优化
   
       
   ```

2. **NULL值问题**

   ```sql
   -- 当子查询可能返回NULL时
   SELECT * 
   FROM products
   WHERE price > (SELECT MAX(price) FROM discontinued_products);
   -- 如果子查询结果为NULL，整个WHERE条件会失效
       
   ```

3. **多行比较错误**

   ```sql
   -- 错误：标量子查询返回多行
   SELECT name 
   FROM employees
   WHERE salary = (
       SELECT salary 
       FROM employees 
       WHERE dept_id = 2
   );
   -- ✅ 应改用IN或LIMIT 1
       
   ```

4. **列不匹配错误**

   ```sql
   -- 错误：行子查询列数不匹配
   SELECT * 
   FROM tableA 
   WHERE (col1, col2) = (
       SELECT col1 
       FROM tableB
   );
      
   ```

------
## 四、实战练习
![](https://i-blog.csdnimg.cn/direct/72abdcb6e3914242a4b390ff23b19207.png)

![](https://i-blog.csdnimg.cn/direct/a402013741974969806a900a6a2e8bee.png)
```sql
# 查询年龄为男，并且年龄在20-40(含)以内的姓名为三个字的员工。
select * from emp where gender='男' and age>20 and age<=40 and name like '___';

# 查询年龄为20，21，22，23岁的员工信息。
select * from emp where age in (20,21,22,23);

# 统计员工表中，年龄小于60岁的，男性员工和女性员工的人数。
select gender,count(*) from emp where age<60 group by gender;

# 查询所有年龄小于等于35岁员工的姓名和年龄，并对查询结果按年龄升序排序，如果年龄相同按入职时间降序排序。
select name,age from emp where age<=35 order by age asc,entrydate desc;

# 查询性别为男，且年龄在20-40(含)以内的前5个员工信息，对查询的结果按年龄升序排序，年龄相同按入职时间升序排序。
select * from emp where gender='男' and age>20 and age<=40 order by age asc,entrydate asc limit 5;

# 由于业务需求变更，企业员工的工号，统一为5位数，目前不足5位数的全部在前面补0。比如：1号员工的工号应该为00001。
update emp set workno=lpad(workno,5,'0');

# 查询emp表的员工姓名和工作地址（北京/上海->一线城市，其他->二线城市）
select name, if(workaddress in ('北京','上海'),'一线城市','二线城市') from emp;
# 法2
select name, case when workaddress in ('北京','上海') then '一线城市' else '二线城市' end from emp;

# 案例：统计班级各个学员的成绩，展示的规则如下：
# >=85，展示优秀
# >=60，展示及格
# 否则，展示不及格
# 法1
select
    name,
    case when math>=85 then '优秀'
         when math>=60 then '及格'
         else '不及格'
    end as math,
    case when english>=85 then '优秀'
         when english>=60 then '及格'
         else '不及格'
    end as english,
    case when chinese>=85 then '优秀'
         when chinese>=60 then '及格'
         else '不及格'
    end as chinese
from score;
# 法2
select
    name,
    if(math>=85,'优秀',if(math>=60,'及格','不及格')) as math,
    if(english>=85,'优秀',if(english>=60,'及格','不及格')) as english,
    if(chinese>=85,'优秀',if(chinese>=60,'及格','不及格')) as chinese
from score;

# 数据库表中，存储的是入职日期，如2000-11-12，如何快速计算入职天数？？？
select name,datediff(now(),entrydate) as entrydays from emp;

# 如果设置了主键自增，那么在插入数据时user(里要指明插入的字段)，如果单单user默认向所有字段去插入，会向主键自增id去插入数据，那么就会报错。
insert into user(name, age, status, gender)values
    ('张三',20,1,'男'),
    ('李四',20,1,'男');

# 1．查询每一个员工的姓名，及关联的部门的名称（隐式内连接实现）
select
    e.name,d.name as dept_name
    from emp e inner join dept d
    where e.dept_id=d.id;


select id from dept where name='财务部';
select max(salary) from emp where dept_id=(select id from dept where name='财务部');
select * from emp where salary>(select max(salary) from emp where dept_id=(select id from dept where name='财务部'));
select * from emp where salary>all(select salary from emp where dept_id=(select id from dept where name='财务部'));

#1.查询员工的姓名、年龄、职位、部门信息。(隐式内连接)
-- 表 emp,dept
-- 连接条件 e.dept_id=d.id
select e.name,e.age,e.job,d.name as dept_name from emp e ,dept d where e.dept_id=d.id;
#2.查询年龄小于30岁的员工姓名、年龄、职位、部门信息。（显示内连接）
-- 表 emp,dept
-- 连接条件 e.dept_id=d.id
select e.name,e.age,e.job,d.name as dept_name from emp e inner join dept d on d.id = e.dept_id where age < 30;
#3.查询拥有员工的部门ID、部门名称。
 -- 表 emp,dept
-- 连接条件 e.dept_id=d.id
# a.查询拥有员工的部门ID
    select emp.dept_id from emp
    group by emp.dept_id
    having count(emp.id)>0;
# b.查询拥有员工的部门ID、部门名称
select d.id,d.name
from dept d  where d.id in (select emp.dept_id from emp
    group by emp.dept_id
    having count(emp.id)>0);
-- 法二 使用distinct关键字
 select distinct d.id,d.name
from dept d, emp e where d.id = e.dept_id;
#4.查询所有年龄大于40岁的员工，及其归属的部门名称；如果员工没有分配部门，也需要展示出来。
-- 表 emp,dept
-- 连接条件 e.dept_id=d.id
-- 外连接
-- 法一
-- a.查询所有年龄大于40岁的员工
select name from emp where age>40;
-- b.查询所有年龄大于40岁的员工，及其归属的部门名称,如果员工没有分配部门，也需要展示出来。
select e.*,d.name from (select * from emp where age>40) e left join dept d on e.dept_id=d.id;
-- 法二
select e.*,d.name from emp e left join dept d on e.dept_id=d.id where e.age>40;
#5.查询所有员工的工资等级。(不同等级对应的最低最高工资是不同的)
-- 表 emp,salgrade
-- 连接条件 emp.salary>=salgrade.losal and emp.salary<=salgrade.hisal
select e.*,s.grade from emp e,salgrade s where e.salary>=s.losal and e.salary<=s.hisal;
-- 简写
select e.*,s.grade from emp e,salgrade s where e.salary between s.losal and s.hisal;
#6.查询"研发部”所有员工的信息及工资等级。
-- 表 emp,dept,salgrade
-- 连接条件 emp.dept_id=dept.id and emp.salary between salgrade.losal and salgrade.hisal
-- 查询条件 dept.name='研发部'
select e.*, s.grade
from emp e,
     dept d,
     salgrade s
where e.dept_id = d.id
  and e.salary between s.losal and s.hisal
  and d.name = '研发部';
#7.查询"研发部”员工的平均工资。
# a.查询"研发部”的部门id
select id from dept where name='研发部';
# b.查询"研发部”员工的平均工资
select avg(salary) from emp where dept_id=(select id from dept where name='研发部');
#8.查询工资比"灭绝”高的员工信息。
# a.查询"灭绝”的工资
select salary from emp where name='灭绝';
# b.查询工资比"灭绝”高的员工信息
select * from emp where salary>(select salary from emp where name='灭绝');
#9.查询比平均薪资高的员工信息。
# a.查询平均薪资
select avg(salary) from emp;
# b.查询比平均薪资高的员工信息
select * from emp where salary>(select avg(salary) from emp);
#10.查询低于本部门平均工资的员工信息。
# a.查询本部门平均工资
select avg(salary) from emp where dept_id=(select dept_id from emp where name='灭绝');
# b.查询低于本部门平均工资的员工信息
select * from emp where salary<(select avg(salary) from emp where dept_id=(select dept_id from emp where name='灭绝'));
#11.查询所有的部门信息，并统计部门的员工人数。
# a.查询所有部门的信息
select * from dept;
# b.统计部门的员工人数
-- 法一 分组
select d.* ,count(e.id) as emp_count from dept d left join emp e on d.id=e.dept_id group by d.id;
-- 法二 子查询
select d.*,(select count(e.id) from emp e where e.dept_id=d.id) as emp_count from dept d;
#12.查询所有学生的选课情况，展示出学生名称，学号，课程名称
-- 表 student,student_course,course 涉及三张表，需要个个条件消除笛卡尔积
-- 连接条件 sc.studentid=s.id and sc.courseid=c.id
-- a.根据学生课程中间表关联学生表查询学生名称，学号
select s.name,s.no from student s left join student_course sc on s.id=sc.studentid;
-- b.根据学生课程中间表关联课程表查询课程名称
select s.name,s.no,c.name from student s left join student_course sc on s.id=sc.studentid left join course c on sc.courseid=c.id;
```

## 五、总结与最佳实践

| 子查询类型 | 适用场景         | 性能建议               |
| ---------- | ---------------- | ---------------------- |
| 标量子查询 | 单值比较         | 优先用于简单条件       |
| EXISTS     | 存在性检查       | 比COUNT(*)效率高       |
| 相关子查询 | 逐行依赖外层数据 | 避免在大数据量场景使用 |
| 表子查询   | 复杂数据过滤     | 考虑改用临时表或视图   |


**1.多表关系**

> 一对多：在多的一方设置外键，关联一的一方的主键 
> 多对多：建立中间表，中间表包含两个外键，关联两张表的主键
> 一对一：用于表结构拆分，在其中任何一方设置外键(UNIQUE)，关联另一方的主键

**2．多表查询**
内连接

> 隐式：SELECT ... FROM 表A，表B WHERE 条件 ... 
>  显式：SELECT ... FROM 表A INNER JOIN 表B ON 条件 ... 

  外连接

> 左外：SELECT...FROM 表A LEFT JOIN 表B ON 条件... 
> 右外：SELECT ...FROM 表A RIGHT JOIN 表B ON 条件 ... 
> 自连接：SELECT..FROM 表A 别名1，表A 别名2 WHERE 条件...
> 子查询：标量子查询、列子查询、行子查询、表子查询