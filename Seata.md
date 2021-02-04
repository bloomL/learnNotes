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

##### API支持

**GlobalTransaction**

全局事务：包括开启事务、提交、回滚、获取当前状态等方法

```java
public interface GlobalTransaction {

    /**
     * 开启一个全局事务（使用默认的事务名和超时时间）
     */
    void begin() throws TransactionException;

    /**
     * 开启一个全局事务，并指定超时时间（使用默认的事务名）
     */
    void begin(int timeout) throws TransactionException;

    /**
     * 开启一个全局事务，并指定事务名和超时时间
     */
    void begin(int timeout, String name) throws TransactionException;

    /**
     * 全局提交
     */
    void commit() throws TransactionException;

    /**
     * 全局回滚
     */
    void rollback() throws TransactionException;

    /**
     * 获取事务的当前状态
     */
    GlobalStatus getStatus() throws TransactionException;

    /**
     * 获取事务的 XID
     */
    String getXid();
}
```

**GlobalTransactionContext**

GlobalTransaction 实例的获取需要通过 GlobalTransactionContext

```java
	/**
    * 获取当前的全局事务实例，如果没有则创建一个新的实例。
    */
    public static GlobalTransaction getCurrentOrCreate() {
        GlobalTransaction tx = getCurrent();
        if (tx == null) {
            return createNew();
        }
        return tx;
    }

    /**
     * 重新载入给定 XID 的全局事务实例，这个实例不允许执行开启事务的操作。
     * 这个 API 通常用于失败的事务的后续集中处理。
     * 比如：全局提交超时，后续集中处理通过重新载入该实例，通过实例方法获取事务当前状态，并根据状态判断是否需要重试全局提交操作。
     */
    public static GlobalTransaction reload(String xid) throws TransactionException {
        GlobalTransaction tx = new DefaultGlobalTransaction(xid, GlobalStatus.UnKnown, GlobalTransactionRole.Launcher) {
            @Override
            public void begin(int timeout, String name) throws TransactionException {
                throw new IllegalStateException("Never BEGIN on a RELOADED GlobalTransaction. ");
            }
        };
        return tx;
    }
```

**TransactionalTemplate**
事务化模板：通过上述 GlobalTransaction 和 GlobalTransactionContext API 把一个**业务服务的调用**包装成带有**分布式事务支持的服务**。

```java
public class TransactionalTemplate {

    public Object execute(TransactionalExecutor business) throws TransactionalExecutor.ExecutionException {

        // 1. 获取当前全局事务实例或创建新的实例
        GlobalTransaction tx = GlobalTransactionContext.getCurrentOrCreate();
        // 2. 开启全局事务
        try {
            tx.begin(business.timeout(), business.name());
        } catch (TransactionException txe) {
            // 2.1 开启失败
            throw new TransactionalExecutor.ExecutionException(tx, txe,
                TransactionalExecutor.Code.BeginFailure);
        }
        Object rs = null;
        try {
            // 3. 调用业务服务
            rs = business.execute();
        } catch (Throwable ex) {
            // 业务调用本身的异常
            try {
                // 全局回滚
                tx.rollback();
                // 3.1 全局回滚成功：抛出原始业务异常
                throw new TransactionalExecutor.ExecutionException(tx, TransactionalExecutor.Code.RollbackDone, ex);
            } catch (TransactionException txe) {
                // 3.2 全局回滚失败：
                throw new TransactionalExecutor.ExecutionException(tx, txe,
                    TransactionalExecutor.Code.RollbackFailure, ex);
            }
        }

        // 4. 全局提交
        try {
            tx.commit();
        } catch (TransactionException txe) {
            // 4.1 全局提交失败：
            throw new TransactionalExecutor.ExecutionException(tx, txe,
                TransactionalExecutor.Code.CommitFailure);
        }
        return rs;
    }
}
```

模板方法执行的异常：ExecutionException

```java
    class ExecutionException extends Exception {

        // 发生异常的事务实例
        private GlobalTransaction transaction;

        // 异常编码：
        // BeginFailure（开启事务失败）
        // CommitFailure（全局提交失败）
        // RollbackFailure（全局回滚失败）
        // RollbackDone（全局回滚成功）
        private Code code;

        // 触发回滚的业务原始异常
        private Throwable originalException;
```

外层调用逻辑 try-catch 这个异常，根据异常编码进行处理：

- BeginFailure （开启事务失败）：getCause() 得到开启事务失败的框架异常，getOriginalException() 为空。
- CommitFailure （全局提交失败）：getCause() 得到全局提交失败的框架异常，getOriginalException() 为空。
- RollbackFailure （全局回滚失败）：getCause() 得到全局回滚失败的框架异常，getOriginalException() 业务应用的原始异常。
- RollbackDone （全局回滚成功）：getCause() 为空，getOriginalException() 业务应用的原始异常。

**RootContext**

事务的根上下文：负责在应用的运行时，维护 XID 。

```java
    /**
     * 得到当前应用运行时的全局事务 XID
     */
    public static String getXID() {
        return CONTEXT_HOLDER.get(KEY_XID);
    }

    /**
     * 将全局事务 XID 绑定到当前应用的运行时中
     */
    public static void bind(String xid) {
        if (LOGGER.isDebugEnabled()) {
            LOGGER.debug("bind " + xid);
        }
        CONTEXT_HOLDER.put(KEY_XID, xid);
    }

    /**
     * 将全局事务 XID 从当前应用的运行时中解除绑定，同时将 XID 返回
     */
    public static String unbind() {
        String xid = CONTEXT_HOLDER.remove(KEY_XID);
        if (LOGGER.isDebugEnabled()) {
            LOGGER.debug("unbind " + xid);
        }
        return xid;
    }

    /**
     * 判断当前应用的运行时是否处于全局事务的上下文中
     */
    public static boolean inGlobalTransaction() {
        return CONTEXT_HOLDER.get(KEY_XID) != null;
    }
```

应用的当前运行的操作是否在一个全局事务的上下文中，就是看 RootContext 中是否有 XID。

RootContext 的默认实现是基于 ThreadLocal 的，即 XID 保存在当前线程上下文中。

**RootContext典型场景**

1. 远程调用事务上下文的传播

   远程调用前获取当前 XID：

   ```java
   String xid = RootContext.getXID();
   ```

   远程调用过程把 XID 也传递到服务提供方，在执行服务提供方的业务逻辑前，把 XID 绑定到当前应用的运行时：

   ```java
   RootContext.bind(rpcXid);
   ```

2. 事务的暂停和恢复

   在一个全局事务中，如果需要某些业务逻辑不在全局事务的管辖范围内，则在调用前，把 XID 解绑：

   ```java
   String unbindXid = RootContext.unbind();
   ```

   待相关业务逻辑执行完成，再把 XID 绑定回去，即可实现全局事务的恢复：

   ```java
   RootContext.bind(unbindXid);
   ```

##### 微服务框架支持

 **事务上下文**

Seata 的事务上下文由 RootContext 来管理。

应用开启一个全局事务后，RootContext 会自动绑定该事务的 XID，事务结束（提交或回滚完成），RootContext 会自动解绑 XID。

```java
// 绑定 XID
RootContext.bind(xid);

// 解绑 XID
String xid = RootContext.unbind();
```

应用可以通过 RootContext 的 API 接口来获取当前运行时的全局事务 XID。

```java
// 获取 XID
String xid = RootContext.getXID();
```

应用是否运行在一个全局事务的上下文中，就是通过 RootContext 是否绑定 XID 来判定的。

```java
    public static boolean inGlobalTransaction() {
        return CONTEXT_HOLDER.get(KEY_XID) != null;
    }
```

**事务传播**

Seata 全局事务的传播机制就是指事务上下文的传播，根本上，就是 XID 的应用运行时的传播方式。

*1. 服务内部的事务传播*

默认的，RootContext 的实现是基于 ***ThreadLocal*** 的，即 XID 绑定在当前**线程上下文**中。

```java
public class ThreadLocalContextCore implements ContextCore {

    private ThreadLocal<Map<String, String>> threadLocal = new ThreadLocal<Map<String, String>>() {
        @Override
        protected Map<String, String> initialValue() {
            return new HashMap<String, String>();
        }

    };

    @Override
    public String put(String key, String value) {
        return threadLocal.get().put(key, value);
    }

    @Override
    public String get(String key) {
        return threadLocal.get().get(key);
    }

    @Override
    public String remove(String key) {
        return threadLocal.get().remove(key);
    }
}
```

所以服务内部的 XID 传播通常是天然的通过同一个线程的调用链路串连起来的。默认不做任何处理，事务的上下文就是传播下去的。

如果希望挂起事务上下文，则需要通过 RootContext 提供的 API 来实现：

```java
// 挂起（暂停）
String xid = RootContext.unbind();

// TODO: 运行在全局事务外的业务逻辑

// 恢复全局事务上下文
RootContext.bind(xid);
```

*2. 跨服务调用的事务传播*

通过上述基本原理，我们可以很容易理解：

> 跨服务调用场景下的事务传播，本质上就是要把 XID 通过服务调用传递到服务提供方，并绑定到 RootContext 中去。

只要能做到这点，理论上 Seata 可以支持任意的微服务框架。

