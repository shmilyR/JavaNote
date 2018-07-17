# 分布式锁
+ 与之对应的有：线程锁、进程锁
    + 线程锁：主要用来给方法、代码块加锁。当某个方法或代码使用锁，在同一时刻仅有一个线程执行该方法或代码块。线程锁只在同一JVM中有效果，因为线程锁的实现基本上是靠线程之间共享内存实现，比如synchronized是共享对象头，显示锁Lock是共享某个对象。
    + 进程锁：为了控制同一操作系统中多个进程访问某个共享资源，因为进程具有独立性，各个进程无法访问其他进程的资源，因此无法通过synchronized等线程锁实现进程锁。
    + 分布式锁：当多个进程不在同一个系统中，用分布式锁控制多个进程对资源的访问
+ 分布式锁的应用场景：
    + 分布式锁用来解决分布情况下的多线程并发问题。
    + 如果在单机情况下，线程之间共享内存，使用线程锁就可解决并发问题。
    + 在分布式情况下(多个JVM)，线程锁可能就起不到作用了，此时，用分布式锁来解决问题。
+ 分布式锁一般有三种实现方式：
    + 数据库乐观锁；
    + 基于Redis的分布式锁；
    + 基于ZooKeeper的分布式锁。
+ 为了确保分布式锁可用，至少要确保锁的实现同时满足以下几种条件：
    + 互斥性。在任意时刻，只有一个客户端能持有锁。
    + 不会发生死锁。即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。
    + 具有容错性。只要大部分的Redis节点正常运行，客户端就可以加锁和解锁。
    + 解铃还须系铃人。加锁和解锁必须是同一个客户端，客户端自己不能把别人加的锁给解了。   
+ java实现  
    + 加锁  
    用setnx()方法设置key,下一个设置相同的key时就会失败，要等到上一个释放锁之后，当前才能获得这个key.setnx()方法就是:<u>set if not exit</u>
    用set()方法设置key,set方法的参数有key、value、nxxx、expx、time
        + key:锁，因为key是唯一的
        + value:解锁时的依据
        + nxxx:如果是SET IF NOT EXIT,即当key不存在时，就进行set操作，当key存在时，就什么也不做
        + expx:给key加一个过期的设置
        + time:代表key的过期时间
    ```java
        public boolean addLock(Jedis jedis, String lockKey, String requestId, int expireTime) {
        String result = jedis.set(lockKey, requestId, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, expireTime);
            if (LOCK_SUCCESS.equals(result)) {
                return true;
            }
            return false;
        }
    ```
    + 解锁  
    ```java
    public static boolean releaseDistributedLock(Jedis jedis, String lockKey, String requestId) {
        
    String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
    Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(requestId));

    if (RELEASE_SUCCESS.equals(result)) {
        return true;
    }
        return false;
    }
    ```
