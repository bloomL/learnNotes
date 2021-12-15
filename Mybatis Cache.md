#### 一级缓存

##### 一级缓存介绍

​	在应用运行中，可能一次数据库会话中，执行多次查询条件相同的SQL，Mybatis提供一级缓存优化这部分，如果是相同的SQL，优先命中一级缓存，避免直接操作数据库，提高性能。

![image-20211206143354677](C:\Users\dell\Desktop\task\md\Mybatis Cache.assets\image-20211206143354677.png)

​	每个Sql Session 中持有Executor，每个Executor中有一个localCache。当用户发起请求时，Mybatis会根据语句生成MappedStatement,在localCache进行查询，如果命中缓存，直接返回结果，如果没有命中，查询数据库，结果写入localCache，最后返回结果

![image-20211208151218163](C:\Users\dell\Desktop\task\md\Mybatis Cache.assets\image-20211208151218163.png)

##### 一级缓存配置

一级缓存有两个选项**SESSION** **STATEMENT**

SESSION:一个会话中执行的所有语句，都共享这个缓存

STATEMENT:只对当前执行的这一个Statement生效

 ![image-20211208152015131](C:\Users\dell\Desktop\task\md\Mybatis Cache.assets\image-20211208152015131.png)

##### 一级缓存工作流程&源码

一级缓存时序图

![image-20211208175519257](C:\Users\dell\Desktop\task\md\Mybatis Cache.assets\image-20211208175519257-16394466189231.png)

源码

**SqlSession**：对外提供用户和数据库交互的方法，隐藏底层实现细节。默认实现类`DefaultSqlSession`

**Executor**：SqlSession向用户提供操作数据库的方法，但和数据库操作有关的职责都会委托给Executor

**BaseExecutor**：实现了Executor接口的抽象类，定义若干抽象类，执行的时候，把具体操作委托给子类

**Cache**：Mybatis的Cache接口，提供缓存相关的基本操作

`BaseExecutor`成员变量之一的`PerpetualCach`，是对Cache最基本的实现。其内部持有HashMap，对一级缓存的操作实则是对HashMap的操作

![image-20211214104533560](C:\Users\dell\Desktop\task\md\Mybatis Cache.assets\image-20211214104533560.png)

执行顺序

1.为执行和数据库的交互，首先需初始化`SqlSession`,通过`DefaultSqlSessionFactory`开启`SqlSession`

```java
private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
  Transaction tx = null;
  try {
    final Environment environment = configuration.getEnvironment();
    final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
    tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
    // 通过configuration创建新的Executor，作为DefaultSqlSession构造函数的参数
    final Executor executor = configuration.newExecutor(tx, execType);
    return new DefaultSqlSession(configuration, executor, autoCommit);
  } catch (Exception e) {
    closeTransaction(tx); // may have fetched a connection so lets call close()
    throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
  } finally {
    ErrorContext.instance().reset();
  }
}
```

2.`SqlSession`创建完成后，根据Statment的不同参数，进入SqlSession不同方法，如果是`Select`语句，最后会执行`SqlSession`的`selectList`

```java
@Override
public <E> List<E> selectList(String statement, Object parameter, RowBounds rowBounds) {
  return selectList(statement, parameter, rowBounds, Executor.NO_RESULT_HANDLER);
}
```

3.`SqlSession`把具体的查询职责委托给Executor。如果只开启一级缓存，首先会进入`BaseExecutor`的`query`

```java
@Override
public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {
  BoundSql boundSql = ms.getBoundSql(parameter);
  // 根据传入参数生成CacheKey
  CacheKey key = createCacheKey(ms, parameter, rowBounds, boundSql);
  return query(ms, parameter, rowBounds, resultHandler, key, boundSql);
}
```

除去hashcode、checksum和count的比较外，只要updatelist中的元素一一对应相等，那么就可以认为是CacheKey相等。只要两条SQL的下列五个值相同，即可以认为是相同的SQL

> Statement Id + Offset + Limmit + Sql + Params

4.`query`方法继续执行

```java
@Override
public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
  ......
  try {
    queryStack++;
    list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
    if (list != null) {
      handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
    } else {
      // 查不到就从数据库查，并写入localCache
      list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
    }
  } finally {
    queryStack--;
  }
  ......
    //会判断一级缓存级别是否是STATEMENT级别，如果是的话，就清空缓存，这也就是STATEMENT级别的一级缓存无法共享localCache的原因
    if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
      // issue #482
      clearLocalCache();
    }
  }
  return list;
}
```

如果是`insert/update/delete`，缓存都会刷新

`SqlSession`的`insert`和`delete`方法，都会统一走`update`的流程

```java
@Override
public int insert(String statement, Object parameter) {
  return update(statement, parameter);
}

@Override
  public int delete(String statement, Object parameter) {
    return update(statement, parameter);
  }
```

`update`方法也委托给Executor执行，`BaseExecutor`中执行方法

```java
@Override
public int update(MappedStatement ms, Object parameter) throws SQLException {
  ErrorContext.instance().resource(ms.getResource()).activity("executing an update").object(ms.getId());
  if (closed) {
    throw new ExecutorException("Executor was closed.");
  }
  // 每次执行update前都会清空localCache
  clearLocalCache();
  return doUpdate(ms, parameter);
}
```

##### 总结

1. MyBatis一级缓存的生命周期和SqlSession一致。
2. MyBatis一级缓存内部设计简单，只是一个没有容量限定的HashMap，在缓存的功能性上有所欠缺。
3. MyBatis的一级缓存最大范围是SqlSession内部，有**多个SqlSession**或者**分布式**的环境下，数据库写操作会引起**脏数据**，建议设定缓存级别为**Statement**。

#### 二级缓存

##### 二级缓存介绍

​	如果多个SqlSession之间需要共享缓存，则需要使用到二级缓存。开启二级缓存后，会使用CachingExecutor装饰Executor，进入一级缓存的查询流程前，先在CachingExecutor进行二级缓存的查询

![image-20211214160403858](C:\Users\dell\Desktop\task\md\Mybatis Cache.assets\image-20211214160403858.png)

二级缓存开启后，同一个**namespace**下的所有操作语句，都影响着同一个Cache，即二级缓存被多个SqlSession共享，是一个全局的变量。

当开启缓存后，数据的查询执行的流程就是 二级缓存 -> 一级缓存 -> 数据库。

##### 二级缓存配置

1. 在MyBatis的配置文件中开启二级缓存
2. 在MyBatis的映射XML中配置cache或者 cache-ref 

##### 二级缓存源码

​	工作流程与一级缓存类似，在一级缓存处理之前，用`CachingExecutor`装饰`BaseExecutor`子类，在委托具体职责给`delegate`之前，实现二级缓存的查询和写入

源码分析

1.`CacheingExecutor`的`query`方法，首先从`MappedStatement`中获取在配置时的初始化Cache

```java
public class CachingExecutor implements Executor {
    private final Executor delegate;
	private final TransactionalCacheManager tcm = new TransactionalCacheManager();
    // TransactionalCacheManager有个map
    //private final Map<Cache, TransactionalCache> transactionalCaches = new HashMap<>();
    //TransactionalCache实现了Cache接口，CachingExecutor会默认使用他包装初始生成的Cache，作用是如果事务提交，对缓存的操作才会生效，如果事务回滚或者不提交事务，则不对缓存产生影响

    @Override
    public <E> List<E> query(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)
        throws SQLException {
      Cache cache = ms.getCache();
        if (cache != null) {
           // 是否需要刷新缓存 select不会刷新，insert/update/delte会刷新缓存
          flushCacheIfRequired(ms);
          if (ms.isUseCache() && resultHandler == null) {
            // 处理存储过程
            ensureNoOutParams(ms, boundSql);
            @SuppressWarnings("unchecked")
            // 获取缓存列表
            // 在getObject方法中，会把获取值的职责一路传递，最终到PerpetualCache。如果没有查到，会把key加入Miss集合，这个主要是为了统计命中率
            List<E> list = (List<E>) tcm.getObject(cache, key);
            if (list == null) {
              list = delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
              // 缓存放入值
              tcm.putObject(cache, key, list); // issue #578 and #116
            }
            return list;
          }
        }
        return delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
    }

    private void flushCacheIfRequired(MappedStatement ms) {
        Cache cache = ms.getCache();
        if (cache != null && ms.isFlushCacheRequired()) {
          tcm.clear(cache);
        }
      }
}
```

本质上是装饰器的使用，装饰器链

> SynchronizedCache -> LoggingCache -> SerializedCache -> LruCache -> PerpetualCache

![image-20211214173901496](C:\Users\dell\Desktop\task\md\Mybatis Cache.assets\image-20211214173901496.png)

- `SynchronizedCache`：同步Cache，实现比较简单，直接使用synchronized修饰方法。
- `LoggingCache`：日志功能，装饰类，用于记录缓存的命中率，如果开启了DEBUG模式，则会输出命中率日志。
- `SerializedCache`：序列化功能，将值序列化后存到缓存中。该功能用于缓存返回一份实例的Copy，用于保存线程安全。
- `LruCache`：采用了Lru算法的Cache实现，移除最近最少使用的Key/Value。
- `PerpetualCache`： 作为为最基础的缓存类，底层实现比较简单，直接使用了HashMap。

##### 总结

1. MyBatis的二级缓存相对于一级缓存来说，实现了`SqlSession`之间缓存数据的共享，同时粒度更加的细，能够到`namespace`级别，通过Cache接口实现类不同的组合，对Cache的可控性也更强。
2. MyBatis在多表查询时，极大可能会出现脏数据，有设计上的缺陷，安全使用二级缓存的条件比较苛刻。
3. 在分布式环境下，由于默认的MyBatis Cache实现都是基于本地的，分布式环境下必然会出现读取到脏数据，需要使用集中式缓存将MyBatis的Cache接口实现，有一定的开发成本，直接使用Redis、Memcached等分布式缓存可能成本更低，安全性也更高。
