## 1.Redis简单介绍

Redis是一种键值型的NoSql数据库，这里有两个关键字：

- 键值型
    
- NoSql
    

其中**键值型**，是指Redis中存储的数据都是以key.value对的形式存储，而value的形式多种多样，可以是字符串.数值.甚至json：
![](assets/Pasted%20image%2020250312200859.png)而NoSql则是相对于传统关系型数据库而言，有很大差异的一种数据库。

对于存储的数据，没有类似Mysql那么严格的约束，比如唯一性，是否可以为null等等，所以我们把这种松散结构的数据库，称之为NoSQL数据库。
## 初始Redis

### 3.1.认识NoSQL

**NoSql**可以翻译做Not Only Sql（不仅仅是SQL），或者是No Sql（非Sql的）数据库。是相对于传统关系型数据库而言，有很大差异的一种特殊的数据库，因此也称之为**非关系型数据库**。

#### 3.1.1.结构化与非结构化

传统关系型数据库是结构化数据，每一张表都有严格的约束信息：字段名.字段数据类型.字段约束等等信息，插入的数据必须遵守这些约束：
![](https://i.imgur.com/4tUgFo6.png)
而NoSql则对数据库格式没有严格约束，往往形式松散，自由。

可以是键值型：

![](https://i.imgur.com/GdqOSsj.png)

也可以是文档型：

![](https://i.imgur.com/zBBQfcc.png)

甚至可以是图格式：

![](https://i.imgur.com/zBnKxWf.png)
#### 3.1.2.关联和非关联

传统数据库的表与表之间往往存在关联，例如外键：

![](https://i.imgur.com/tXYSl5x.png)

而非关系型数据库不存在关联关系，要维护关系要么靠代码中的业务逻辑，要么靠数据之间的耦合：
```json
{  
  id: 1,  
  name: "张三",  
  orders: [  
    {  
       id: 1,  
       item: {  
     id: 10, title: "荣耀6", price: 4999  
       }  
    },  
    {  
       id: 2,  
       item: {  
     id: 20, title: "小米11", price: 3999  
       }  
    }  
  ]  
}
```
此处要维护“张三”的订单与商品“荣耀”和“小米11”的关系，不得不冗余的将这两个商品保存在张三的订单文档中，不够优雅。还是建议用业务来维护关联关系。
#### 3.1.3.查询方式

传统关系型数据库会基于Sql语句做查询，语法有统一标准；

而不同的非关系数据库查询语法差异极大，五花八门各种各样。

![](https://i.imgur.com/AzaHOTF.png)

#### 3.1.4.事务

传统关系型数据库能满足事务ACID的原则。

![](https://i.imgur.com/J1MqOJM.png)

而非关系型数据库往往不支持事务，或者不能严格保证ACID的特性，只能实现基本的一致性。

#### 3.1.5.总结

除了上述四点以外，在存储方式.扩展性.查询性能上关系型与非关系型也都有着显著差异，总结如下：

![](https://i.imgur.com/kZP40dQ.png)

- 存储方式
    
    - 关系型数据库基于磁盘进行存储，会有大量的磁盘IO，对性能有一定影响
        
    - 非关系型数据库，他们的操作更多的是依赖于内存来操作，内存的读写速度会非常快，性能自然会好一些
        

- 扩展性
    
    - 关系型数据库集群模式一般是主从，主从数据一致，起到数据备份的作用，称为垂直扩展。
        
    - 非关系型数据库可以将数据拆分，存储在不同机器上，可以保存海量数据，解决内存大小有限的问题。称为水平扩展。
        
    - 关系型数据库因为表之间存在关联关系，如果做水平扩展会给数据查询带来很多麻烦
        

### 3.2.认识Redis

Redis诞生于2009年全称是**Re**mote **D**ictionary **S**erver 远程词典服务器，是一个基于内存的键值型NoSQL数据库。

**特征**：

- 键值（key-value）型，value支持多种不同数据结构，功能丰富
    
- 单线程，每个命令具备原子性
    
- 低延迟，速度快（基于内存.IO多路复用.良好的编码）。
    
- 支持数据持久化
    
- 支持主从集群.分片集群
    
- 支持多语言客户端
    

**作者**：Antirez

Redis的官方网站地址：[https://redis.io/](https://redis.io/)
### 3.3.安装Redis

大多数企业都是基于Linux服务器来部署项目，而且Redis官方也没有提供Windows版本的安装包。因此课程中我们会基于Linux系统来安装Redis.

此处选择的Linux版本为CentOS 7.

#### 3.3.1.依赖库

Redis是基于C语言编写的，因此首先需要安装Redis所需要的gcc依赖：

yum install -y gcc tcl

#### 3.3.2.上传安装包并解压

然后将课前资料提供的Redis安装包上传到虚拟机的任意目录：

![](https://i.imgur.com/SyjanS5.png)

例如，我放到了/usr/local/src 目录：

![](https://i.imgur.com/01DTNCf.png)

解压缩：

tar -xzf redis-6.2.6.tar.gz

解压后：

![image-20211211080339076](https://i.imgur.com/8V6zvCD.png)

进入redis目录：

cd redis-6.2.6

运行编译命令：

make && make install

如果没有出错，应该就安装成功了。

默认的安装路径是在 `/usr/local/bin`目录下：

![](https://i.imgur.com/YSxkGm7.png)

该目录已经默认配置到环境变量，因此可以在任意目录下运行这些命令。其中：

- redis-cli：是redis提供的命令行客户端
    
- redis-server：是redis的服务端启动脚本
    
- redis-sentinel：是redis的哨兵启动脚本
    

#### 3.3.3.启动

redis的启动方式有很多种，例如：

- 默认启动
    
- 指定配置启动
    
- 开机自启
    

#### 3.3.4.默认启动

安装完成后，在任意目录输入redis-server命令即可启动Redis：

```bash
redis-server
```

如图：

![](https://i.imgur.com/v7xWsqC.png)

这种启动属于`前台启动`，会阻塞整个会话窗口，窗口关闭或者按下`CTRL + C`则Redis停止。不推荐使用。
#### 3.3.5.指定配置启动

如果要让Redis以`后台`方式启动，则必须修改Redis配置文件，我们先将这个配置文件备份一份：
```bash
cp redis.conf redis.conf.bck
```
然后修改redis.conf文件中的一些配置：
```properties
# 允许访问的地址，默认是127.0.0.1，会导致只能在本地访问。修改为0.0.0.0则可以在任意IP访问，生产环境不要设置为0.0.0.0
bind 0.0.0.0
# 守护进程，修改为yes后即可后台运行
daemonize yes 
# 密码，设置后访问Redis必须输入密码
requirepass 123321
```
Redis的其它常见配置：
```properties
# 监听的端口
port 6379
# 工作目录，默认是当前目录，也就是运行redis-server时的命令，日志.持久化等文件会保存在这个目录
dir .
# 数据库数量，设置为1，代表只使用1个库，默认有16个库，编号0~15
databases 1
# 设置redis能够使用的最大内存
maxmemory 512mb
# 日志文件，默认为空，不记录日志，可以指定日志文件名
logfile "redis.log"
```
启动Redis：
```sh
# 进入redis安装目录 
cd /usr/local/src/redis-6.2.6
# 启动
redis-server redis.conf
```
停止服务：
```sh
# 利用redis-cli来执行 shutdown 命令，即可停止 Redis 服务，
# 因为之前配置了密码，因此需要通过 -u 来指定密码
redis-cli -u 123321 shutdown
```
#### 3.3.6.开机自启

我们也可以通过配置来实现开机自启。

首先，新建一个系统服务文件：
```sh
vi /etc/systemd/system/redis.service
```
内容如下：
```conf
[Unit]
Description=redis-server
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/bin/redis-server /usr/local/src/redis-6.2.6/redis.conf
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```
然后重载系统服务：
```sh
systemctl daemon-reload
```
现在，我们可以用下面这组命令来操作redis了：
```sh
# 启动
systemctl start redis
# 停止
systemctl stop redis
# 重启
systemctl restart redis
# 查看状态
systemctl status redis
```
执行下面的命令，可以让redis开机自启：
```sh
systemctl enable redis
```
### 4.2 Redis 通用命令

通用指令是部分数据类型的，都可以使用的指令，常见的有：

- KEYS：查看符合模板的所有key
    
- DEL：删除一个指定的key
    
- EXISTS：判断key是否存在
    
- EXPIRE：给一个key设置有效期，有效期到期时该key会被自动删除
    
- TTL：查看一个KEY的剩余有效期
    

通过help [command] 可以查看一个命令的具体用法，例如：

![1652887865189](assets/1652887865189.png)