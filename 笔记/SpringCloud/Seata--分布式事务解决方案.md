# Seata--分布式事务解决方案

## 简介

Seata 是阿里开源的分布式事务解决方案，提供 **AT**、**XA**、**TCC**、**Saga** 四种事务模式，支持微服务架构下的数据一致性。

官网：https://seata.apache.org/zh-cn/  

### 同类产品对比

| 方案                  | 核心特点                           |           适用场景           |
| --------------------- | ---------------------------------- | :--------------------------: |
| **Seata**             | 多模式支持，代码侵入性低，社区活跃 | 复杂业务场景，需灵活选择模式 |
| **阿里云 GTS**        | 商业版方案，功能全面，性能强       |        企业级付费场景        |
| **RocketMQ 事务消息** | 基于消息队列实现最终一致性         |        异步高吞吐场景        |
| **LCN**               | 基于代理模式，实现简单             |        轻量级快速接入        |

---

## 1 环境搭建

### 1.1 微服务

创建 Spring Cloud 项目，推荐使用以下组件：

```xml
<!-- Spring Cloud Alibaba 依赖 -->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
</dependency>
```

### 1.2 SQL

每个微服务数据库需创建 **undo_log** 表（AT模式必需）：

```sql
-- 每个业务数据库均需执行
CREATE TABLE IF NOT EXISTS `undo_log`
(
    `id`            BIGINT(20)   NOT NULL AUTO_INCREMENT,
    `branch_id`     BIGINT(20)   NOT NULL,
    `xid`           VARCHAR(100) NOT NULL,
    `context`       VARCHAR(128) NOT NULL,
    `rollback_info` LONGBLOB     NOT NULL,
    `log_status`    INT(11)      NOT NULL,
    `log_created`   DATETIME     NOT NULL,
    `log_modified`  DATETIME     NOT NULL,
    PRIMARY KEY (`id`),
    UNIQUE KEY `ux_undo_log` (`xid`, `branch_id`)
) ENGINE = InnoDB
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8;
```

### 1.3 seata-server

1. 下载 :https://seata.apache.org/zh-cn/download/seata-server

   ​			[📎apache-seata-2.1.0-incubating-bin.tar.gz](https://www.yuque.com/attachments/yuque/0/2025/gz/1613913/1736420650170-1143baab-fbe9-4114-b7e0-515349276444.gz)

​	2.解压并启动：seata-server.bat

​	3.控制台:http://127.0.0.1:7091/#/transaction/list

### 1.4 微服务配置

#### 1.4.1 依赖

除基础依赖外需添加：

```xml
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
</dependency>
```

#### 1.4.2 配置

每个微服务创建 `file.conf`文件，完整内容如下；

【微服务只需要复制 `service 块`配置即可】

```yaml
#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

transport {
  # tcp, unix-domain-socket
  type = "TCP"
  #NIO, NATIVE
  server = "NIO"
  #enable heartbeat
  heartbeat = true
  # the tm client batch send request enable
  enableTmClientBatchSendRequest = false
  # the rm client batch send request enable
  enableRmClientBatchSendRequest = true
   # the rm client rpc request timeout
  rpcRmRequestTimeout = 2000
  # the tm client rpc request timeout
  rpcTmRequestTimeout = 30000
  # the rm client rpc request timeout
  rpcRmRequestTimeout = 15000
  #thread factory for netty
  threadFactory {
    bossThreadPrefix = "NettyBoss"
    workerThreadPrefix = "NettyServerNIOWorker"
    serverExecutorThread-prefix = "NettyServerBizHandler"
    shareBossWorker = false
    clientSelectorThreadPrefix = "NettyClientSelector"
    clientSelectorThreadSize = 1
    clientWorkerThreadPrefix = "NettyClientWorkerThread"
    # netty boss thread size
    bossThreadSize = 1
    #auto default pin or 8
    workerThreadSize = "default"
  }
  shutdown {
    # when destroy server, wait seconds
    wait = 3
  }
  serialization = "seata"
  compressor = "none"
}
service {
  #transaction service group mapping
  vgroupMapping.default_tx_group = "default"
  #only support when registry.type=file, please don't set multiple addresses
  default.grouplist = "127.0.0.1:8091"
  #degrade, current not support
  enableDegrade = false
  #disable seata
  disableGlobalTransaction = false
}

client {
  rm {
    asyncCommitBufferLimit = 10000
    lock {
      retryInterval = 10
      retryTimes = 30
      retryPolicyBranchRollbackOnConflict = true
    }
    reportRetryCount = 5
    tableMetaCheckEnable = false
    tableMetaCheckerInterval = 60000
    reportSuccessEnable = false
    sagaBranchRegisterEnable = false
    sagaJsonParser = "fastjson"
    sagaRetryPersistModeUpdate = false
    sagaCompensatePersistModeUpdate = false
    tccActionInterceptorOrder = -2147482648 #Ordered.HIGHEST_PRECEDENCE + 1000
    sqlParserType = "druid"
    branchExecutionTimeoutXA = 60000
    connectionTwoPhaseHoldTimeoutXA = 10000
  }
  tm {
    commitRetryCount = 5
    rollbackRetryCount = 5
    defaultGlobalTransactionTimeout = 60000
    degradeCheck = false
    degradeCheckPeriod = 2000
    degradeCheckAllowTimes = 10
    interceptorOrder = -2147482648 #Ordered.HIGHEST_PRECEDENCE + 1000
  }
  undo {
    dataValidation = true
    onlyCareUpdateColumns = true
    logSerialization = "jackson"
    logTable = "undo_log"
    compress {
      enable = true
      # allow zip, gzip, deflater, lz4, bzip2, zstd default is zip
      type = zip
      # if rollback info size > threshold, then will be compress
      # allow k m g t
      threshold = 64k
    }
  }
  loadBalance {
      type = "XID"
      virtualNodes = 10
  }
}
log {
  exceptionRate = 100
}
tcc {
  fence {
    # tcc fence log table name
    logTableName = tcc_fence_log
    # tcc fence log clean period
    cleanPeriod = 1h
  }
}
```

------

## 2 事务模式

### 2.1 AT模式（推荐:自动补偿）

> 二阶提交协议原理

**原理**：基于反向SQL补偿，自动生成回滚日志

![SpringCloud-第 2 页.drawio.png](https://i-blog.csdnimg.cn/img_convert/469d3ddbec2dd2a73cdf5b6d3a1bf2a2.png)

**使用**：

```java
@Service
public class OrderService {
    @GlobalTransactional(name = "createOrder", timeoutMills = 60000)
    public void createOrder(OrderDTO order) {
        // 1. 扣减库存（调用库存服务）
        storageFeignClient.deduct(order.getProductId());
        // 2. 创建订单（本地事务）
        orderMapper.insert(order);
        // 3. 模拟异常触发回滚
        int i = 1 / 0; 
    }
}
```

**关键机制**：

- 一阶段：提交本地事务，生成回滚日志（undo_log）
- 二阶段：成功则异步删除日志，失败则通过日志反向补偿

**特点**：

- 对代码无侵入
- 需创建undo_log表
- 适用于大多数CRUD场景

### 2.2 XA模式（强一致）

**原理**：基于数据库XA协议的两阶段提交
**配置**：

```yaml
seata:
  data-source-proxy-mode: XA  # 默认AT
```

####  特点

- 基于数据库XA协议
- 两阶段提交（2PC）
- 事务持有锁时间较长，适合短事务

**适用场景**：强一致性需求，支持XA协议的数据库（如MySQL 5.7+）

### 2.3 TCC模式（手动补偿）

**原理**：Try-Confirm-Cancel 三阶段控制
**实现**：

```java
// 1. 定义TCC接口
public interface StorageTccService {
    @TwoPhaseBusinessAction(name = "deduct", 
                          commitMethod = "confirm", 
                          rollbackMethod = "cancel")
    boolean deduct(@BusinessActionContextParameter(paramName = "productId") String productId,
                   @BusinessActionContextParameter(paramName = "count") Integer count);
    
    boolean confirm(BusinessActionContext context);
    boolean cancel(BusinessActionContext context);
}

// 2. 实现Try逻辑
@Service
public class StorageTccServiceImpl implements StorageTccService {
    @Override
    public boolean deduct(String productId, Integer count) {
        // Try阶段：资源预留（例如冻结库存）
        return storageMapper.freezeStock(productId, count) > 0;
    }
    
    @Override
    public boolean confirm(BusinessActionContext context) {
        // Confirm阶段：真实扣减（例如删除冻结记录）
        String productId = context.getActionContext("productId");
        Integer count = context.getActionContext("count");
        return storageMapper.reduceStock(productId, count) > 0;
    }
    
    @Override
    public boolean cancel(BusinessActionContext context) {
        // Cancel阶段：释放资源（例如恢复冻结库存）
        String productId = context.getActionContext("productId");
        Integer count = context.getActionContext("count");
        return storageMapper.unfreezeStock(productId, count) > 0;
    }
}
```

####  使用限制

- 需自行实现Try/Confirm/Cancel方法
- Confirm和Cancel需保证幂等性

**特点**：高性能，但需手动编写补偿逻辑

### 2.4 Saga模式（长事务）

#### 实现方式

通过状态机配置补偿策略：

```java
@SagaStart
public void createOrderSaga(Order order) {
    // 1. 创建订单
    orderService.create(order);
    // 2. 调用支付服务（若失败则触发逆向操作）
    paymentService.pay(order.getId());
}
// 定义补偿方法
@Compensate
public void compensateOrder(Order order) {
    orderService.delete(order.getId());
}
```

**原理**：长事务拆分+逆向补偿
**适用场景**：跨系统长时间操作（如订单+物流+支付）

------

## 3. 总结

###  模式选型对照表

| 模式     | 一致性   | 性能 | 侵入性 | 适用场景                       |
| :------- | :------- | :--- | :----- | :----------------------------- |
| **AT**   | 弱一致   | 高   | 低     | 常规业务（库存扣减、订单创建） |
| **TCC**  | 强一致   | 中   | 高     | 资金交易、需精准控制           |
| **XA**   | 强一致   | 低   | 低     | 银行转账、短事务               |
| **Saga** | 最终一致 | 高   | 中     | 跨系统长流程（订单+物流+支付） |

### 核心要点

| 分类       |                           要点说明                           |
| :--------- | :----------------------------------------------------------: |
| **选型**   | **AT模式适用于大多数场景，TCC适合高性能要求，XA适合强一致性** |
| **配置**   |          确保seata-server与微服务的registry配置一致          |
| **事务ID** |          通过`RootContext.getXID()`可获取当前事务ID          |
| **排错**   |        检查undo_log表记录，查看seata-server控制台日志        |

### 最佳实践

1. 生产环境建议使用Nacos作为配置中心
2. AT模式需要开启数据库的本地事务支持（如MySQL的InnoDB引擎）
3. 全局事务超时时间建议设置：`seata.tx.timeout=60000`（单位毫秒）

