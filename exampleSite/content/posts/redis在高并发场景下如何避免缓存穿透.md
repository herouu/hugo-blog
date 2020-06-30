---
title: redis在高并发场景下如何避免压力作用在数据库
date: 2019-06-09 13:27:34
tags: ["缓存"]

---

缓存，在项目开发中是必不可少的，redis 作为远程缓存来说，在系统优化层面使用的频率还是比较高的。

<!--more-->

一般来说，一个接口的请求到响应的过程一般如下图所示：

![缓存的逻辑](https://bj.bcebos.com/v1/alertcode-blog/redis在高并发场景下如何避免压力作用在数据库/缓存的逻辑_2019-06-09_13-46-01.png)

- 先请求缓存，缓存没有，查询数据库 数据库结果更新到缓存
  在单线程模型中，这样的处理的确不会有问题，但是如果在多线程场景下，下面的这段代码是有问题的
  ![模拟缓存雪崩代码](https://bj.bcebos.com/v1/alertcode-blog/redis在高并发场景下如何避免压力作用在数据库/模拟缓存雪崩代码.png)

### 考虑下面的这个场景

- 设置缓存的时候设置了相同的过期时间，导致缓存在某一时刻同时失效，缓存失效的情况下，多个线程运行上面箭头所指的代码，发现缓存中并没有 key，直接作用到数据库，在并发数较低的情况下，对数据库查询不会产生什么影响，但是并发数上千，上万，请求的压力直接作用到数据库，结果无非就是两种，一种直接挂掉（缓存雪崩），另一种就是响应超时。

### 下面通过 TestNG 模拟高并发场景。

代码如下：

```java
    private static final int invocationCount = 1000;
    private static final int threadPoolSize = 1000;

    // 并发数 threadPoolSize  总请求数invocationCount
    @Test(invocationCount = invocationCount, threadPoolSize = threadPoolSize)
    public void testCacheGetById() {
        commonService.cacheGetById(RepaymentAudit.class, 2);
    }

```

运行结果：
![缓存雪崩模拟](https://bj.bcebos.com/v1/alertcode-blog/redis在高并发场景下如何避免压力作用在数据库/模拟缓存雪崩.gif)
可以看到，线程根本没有作用到缓存，直接请求到数据库，对数据库造成压力
![并发场景下数据库连接超时](https://bj.bcebos.com/v1/alertcode-blog/redis在高并发场景下如何避免压力作用在数据库/并发场景下数据库连接超时.png)

### 如何解决

有以下两种方案：

#### - 设置可重入锁，保证一个时间点，只有一个线程请求数据库，并更新缓存

```java
    ReentrantLock lock = new ReentrantLock();

    private <T> T cacheGetByIdLock(Class<T> clazz, Serializable id) {
        if (tableCacheDao.exists(getHName(clazz), Objects.toString(id))) {
            log.info("从缓存中取数据：线程={}", Thread.currentThread().getId());
            return JsonUtils.readValue(tableCacheDao.get(getHName(clazz), Objects.toString(id)), clazz);
        }
        lock.lock();
        try {
            if (tableCacheDao.exists(getHName(clazz), Objects.toString(id))) {
                log.info("从缓存中取数据：线程={}", Thread.currentThread().getId());
                return JsonUtils.readValue(tableCacheDao.get(getHName(clazz), Objects.toString(id)), clazz);
            }
            T t = (T) super.getById(id);
            log.info("从数据库中取数据：线程={}", Thread.currentThread().getId());
            tableCacheDao.add(getHName(clazz), getId(t), JsonUtils.writeValueAsString(t));
            return t;
        } finally {
            lock.unlock();
        }
    }
```

优点： 适用于并发量级较小，获得所得线程更新缓存，其他线程去缓存中读取
缺点： 锁的类度粗，响应慢，1000 个线程，一个线程获得锁，999 个线程阻塞。

![可重入锁](https://bj.bcebos.com/v1/alertcode-blog/redis在高并发场景下如何避免压力作用在数据库/锁粗粒度.gif)

#### - 缓存熔断或降级

```java
    private ConcurrentHashMap<Serializable, String> mapLock = new ConcurrentHashMap();

    private <T> T cacheGetByIdSegmentLock(Class<T> clazz, Serializable id) {
        if (tableCacheDao.exists(getHName(clazz), Objects.toString(id))) {
            log.info("从缓存中取数据：线程={}", Thread.currentThread().getId());
            return JsonUtils.readValue(tableCacheDao.get(getHName(clazz), Objects.toString(id)), clazz);
        }
        boolean lock = false;
        final String lockKey = getHName(clazz) + id;
        try {
            lock = mapLock.putIfAbsent(lockKey, "true") == null;
            if (lock) {
                T t = (T) super.getById(id);
                log.info("从数据库中取数据：线程={}", Thread.currentThread().getId());
                tableCacheDao.add(getHName(clazz), getId(t), JsonUtils.writeValueAsString(t));
                return t;
            } else {
                log.info("没拿到lock 熔断：{}", Thread.currentThread().getId());
                throw new ProjectException("当前请求忙，请稍后再试");
            }
        } finally {
            if (lock) {
                mapLock.remove(lockKey);
            }
        }
    }
```

优点： 试用于高并发场景，对熔断或降级可以接受的业务，锁的类度细，响应快，
缺点： 1 个成功 999 失败

对于不接受熔断或降级的场景，保证缓存永不失效，则请求压力作用在缓存
