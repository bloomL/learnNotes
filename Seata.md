#### 概述

##### Seata 是什么

一款**开源**的**分布式事务解决**方案，致力于提供**高性能**和**简单易用**的分布式事务服务。Seata 将为用户提供了 AT、TCC、SAGA 和 XA 事务模式

##### 术语

###### TC (Transaction Coordinator) - 事务协调者

维护全局和分支事务的状态，**驱动全局事务**提交或回滚。

###### TM (Transaction Manager) - 事务管理器

定义全局事务的范围：开始全局事务、提交或回滚全局事务。

###### RM (Resource Manager) - 资源管理器

管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

#### 用户文档

##### example

用户购买商品的业务逻辑。整个业务逻辑由3个微服务提供支持：

- 仓储服务：对给定的商品扣除仓储数量。
- 订单服务：根据采购需求创建订单。
- 帐户服务：从用户帐户中扣除余额。

**架构图**

![image-20210204160256889](Seata.assets/image-20210204160256889.png)

**仓库服务**

```java
public interface StorageService {

    /**
     * 扣除存储数量
     */
    void deduct(String commodityCode, int count);
}
```

**订单服务**

```java
public interface OrderService {

    /**
     * 创建订单
     */
    Order create(String userId, String commodityCode, int orderCount);
}
```

**帐户服务**

```java
public interface AccountService {

    /**
     * 从用户账户中借出
     */
    void debit(String userId, int money);
}
```

**主业务逻辑**

```java
public class BusinessServiceImpl implements BusinessService {

    private StorageService storageService;

    private OrderService orderService;

    /**
     * 采购
     */
    @GlobalTransactional
    public void purchase(String userId, String commodityCode, int orderCount) {
        // 减库存
        storageService.deduct(commodityCode, orderCount);
        // 创订单
        orderService.create(userId, commodityCode, orderCount);
    }
}

public class OrderServiceImpl implements OrderService {

    private OrderDAO orderDAO;

    private AccountService accountService;

    public Order create(String userId, String commodityCode, int orderCount) {

        int orderMoney = calculate(commodityCode, orderCount);
        // 扣费
        accountService.debit(userId, orderMoney);

        Order order = new Order();
        order.userId = userId;
        order.commodityCode = commodityCode;
        order.count = orderCount;
        order.money = orderMoney;

        // INSERT INTO orders ...
        return orderDAO.insert(order);
    }
}
```

SEATA 的分布式交易解决方案

![image-20210204160851830](Seata.assets/image-20210204160851830.png)

只需要使用一个 `@GlobalTransactional` 注解在业务方法上:

```java
    @GlobalTransactional
    public void purchase(String userId, String commodityCode, int orderCount) {
        ......
    }
```

UNDO_LOG表

```mysql
-- 注意此处0.3.0+ 增加唯一索引 ux_undo_log
CREATE TABLE `undo_log` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `branch_id` bigint(20) NOT NULL,
  `xid` varchar(100) NOT NULL,
  `context` varchar(128) NOT NULL,
  `rollback_info` longblob NOT NULL,
  `log_status` int(11) NOT NULL,
  `log_created` datetime NOT NULL,
  `log_modified` datetime NOT NULL,
  `ext` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

API支持

