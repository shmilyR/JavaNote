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
## 数据库实现
+ 基于数据库的实现方式的核心思想是：在数据库中创建一个表，表中包含方法名等字段，并在方法名字段上创建唯一索引，想要执行某个方法，就使用这个方法名向表中插入数据，成功插入则获取锁，执行完成后删除对应的行数据释放锁.
+ 使用基于数据库的这种实现方式很简单，但是对于分布式锁应该具备的条件来说，它有一些问题需要解决及优化：
    + 因为是基于数据库实现的，数据库的可用性和性能将直接影响分布式锁的可用性及性能，所以，数据库需要双机部署、数据同步、主备切换；
    + 不具备可重入的特性，因为同一个线程在释放锁之前，行数据一直存在，无法再次成功插入数据，所以，需要在表中新增一列，用于记录当前获取到锁的机器和线程信息，在再次获取锁的时候，先查询表中机器和线程信息是否和当前机器和线程相同，若相同则直接获取锁；
    + 没有锁失效机制，因为有可能出现成功插入数据后，服务器宕机了，对应的数据没有被删除，当服务恢复后一直获取不到锁，所以，需要在表中新增一列，用于记录失效时间，并且需要有定时任务清除这些失效的数据；
    + 不具备阻塞锁特性，获取不到锁直接返回失败，所以需要优化获取逻辑，循环多次去获取。
    + 在实施的过程中会遇到各种不同的问题，为了解决这些问题，实现方式将会越来越复杂；依赖数据库需要一定的资源开销，性能问题需要考虑。
## redis实现分布式锁  
 <img src="image/redis分布式锁优化版流程图.PNG"><img> 
在此流程下，不会产生死锁。
+ 加锁  
用setnx()方法设置key,下一个设置相同的key时就会失败，要等到上一个释放锁之后，当前才能获得这个key.setnx()方法就是:<u>set if not exit</u>
用set()方法设置key,set方法的参数有key、value、nxxx、expx、time
    + key:锁，因为key是唯一的
    + value:解锁时的依据
    + nxxx:如果是SET IF NOT EXIT,即当key不存在时，就进行set操作，当key存在时，就什么也不做
    + expx:给key加一个过期的设置
    + time:代表key的过期时间
```java
//1、首先要创建一个Redis连接池 
public class RedisPool {
    private static JedisPool pool;//jedis连接池
    private static int maxTotal = 20;//最大连接数
    private static int maxIdle = 10;//最大空闲连接数
    private static int minIdle = 5;//最小空闲连接数
    private static boolean testOnBorrow = true;//在取连接时测试连接的可用性
    private static boolean testOnReturn = false;//再还连接时不测试连接的可用性

    static {
        initPool();//初始化连接池
    }

    public static Jedis getJedis(){
        return pool.getResource();
    }

    public static void close(Jedis jedis){
        jedis.close();
    }

    private static void initPool(){
        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(maxTotal);
        config.setMaxIdle(maxIdle);
        config.setMinIdle(minIdle);
        config.setTestOnBorrow(testOnBorrow);
        config.setTestOnReturn(testOnReturn);
        config.setBlockWhenExhausted(true);
        pool = new JedisPool(config, "127.0.0.1", 6379, 5000, "liqiyao");
    }
//2、对Jedis的api进行封装，封装一些实现分布式锁要用到的操作。  
public class RedisPoolUtil {
    private RedisPoolUtil(){}
    private static RedisPool redisPool;
    public static String get(String key){
        Jedis jedis = null;
        String result = null;
        try {
            jedis = RedisPool.getJedis();
            result = jedis.get(key);
        } catch (Exception e){
            e.printStackTrace();
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        return result;
    }
}

public static Long setnx(String key, String value){
    Jedis jedis = null;
    Long result = null;
    String password = "123456789";
    try {
        jedis = RedisPool.getJedis();
        jedis.auth(password);
        result = jedis.setnx(key, value);
    } catch (Exception e){
        e.printStackTrace();
    } finally {
        if (jedis != null) {
            jedis.close();
        }
        return result;
    }
}

public static String getSet(String key, String value){
    Jedis jedis = null;
    String result = null;
    try {
        jedis = RedisPool.getJedis();
        jedis.auth(password);
        result = jedis.getSet(key, value);
    } catch (Exception e){
        e.printStackTrace();
    } finally {
        if (jedis != null) {
            jedis.close();
        }
        return result;
    }
}

public static Long expire(String key, int seconds){
    Jedis jedis = null;
    Long result = null;
    try {
        jedis = RedisPool.getJedis();
        jedis.auth(password);
        result = jedis.expire(key, seconds);
    } catch (Exception e){
        e.printStackTrace();
    } finally {
        if (jedis != null) {
            jedis.close();
        }
        return result;
    }
}

public static Long del(String key){
    Jedis jedis = null;
    Long result = null;
    try {
        jedis = RedisPool.getJedis();
        jedis.auth(password);
        result = jedis.del(key);
    } catch (Exception e){
        e.printStackTrace();
    } finally {
        if (jedis != null) {
            jedis.close();
        }
        return result;
    }
}
//3、分布式锁的工具类
public class DistributedLockUtil {
    private DistributedLockUtil(){
    }

    public static boolean lock(String lockName){//lockName可以为共享变量名，也可以为方法名，主要是用于模拟锁信息
        System.out.println(Thread.currentThread() + "开始尝试加锁！");
        Long result = RedisPoolUtil.setnx(lockName, String.valueOf(System.currentTimeMillis() + 5000));
        if (result != null && result.intValue() == 1){
            System.out.println(Thread.currentThread() + "加锁成功！");
            RedisPoolUtil.expire(lockName, 5);
            System.out.println(Thread.currentThread() + "执行业务逻辑！");
            RedisPoolUtil.del(lockName);
            return true;
        } else {
            String lockValueA = RedisPoolUtil.get(lockName);
            if (lockValueA != null && Long.parseLong(lockValueA) >= System.currentTimeMillis()){
                    String lockValueB = RedisPoolUtil.getSet(lockName, String.valueOf(System.currentTimeMillis() + 5000));
                if (lockValueB == null || lockValueB.equals(lockValueA)){
                System.out.println(Thread.currentThread() + "加锁成功！");
                RedisPoolUtil.expire(lockName, 5);
                System.out.println(Thread.currentThread() + "执行业务逻辑！");
                RedisPoolUtil.del(lockName);
                return true;
                } else {
                return false;
                }
            } else {
            return false;
            }
        }
    }
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
## 基于ZooKeeper的分布式锁
+ ZooKeeper介绍：为分布式应用提供一致性服务的开源组件，内部是一个分层的文件系统目录树结构，规定同一个目录下只能有一个唯一文件名.
+ 基本思想：每个客户端对某个方法加锁时，在zookeeper上的与该方法对应的指定节点的目录下，生成一个唯一的瞬时有序节点。
判断是否获取锁的方式很简单，只需要判断有序节点中序号最小的一个。
当释放锁的时候，只需将这个瞬时节点删除即可
+ 步骤：
    + 创建一个目录mylock； 
    + 线程A想获取锁就在mylock目录下创建临时顺序节点； 
    + 获取mylock目录下所有的子节点，然后获取比自己小的兄弟节点，如果不存在，则说明当前线程顺序号最小，获得锁； 
    + 线程B获取所有节点，判断自己不是最小节点，设置监听比自己次小的节点； 
    + 线程A处理完，删除自己的节点，线程B监听到变更事件，判断自己是不是最小的节点，如果是则获得锁。

