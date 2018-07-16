# Redis基础
+ 介绍
    + 远程数据服务(Remote Dictionary Server)
    + 内存高速缓存数据库
    + 使用C编写,为key-value的数据模型
    + 支持的value类型：String,Hash,list(链表),set,有序set
    + 为了保证效率，数据都是缓存在内存中，也可以周期性的把更新的数据写入磁盘，或者把修改操作写入追加的记录文件。
    + 出现的原因：mysql是持久化在磁盘里，所以cpu访问会很慢，而redis是在内存中，将redis作为MySQL的本地缓存。效率将会提升，有效缓解数据库压力。
+ 特点：
   + 高速读取数据
   + 减轻数据库负担
   + 有集合计算功能
   + 多种数据结构支持
+ redis的优势：
    + 性能极高 – Redis能读的速度是110000次/s,写的速度是81000次/s 。
    + 丰富的数据类型 – Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
    + 原子 – Redis的所有操作都是原子性的，意思就是要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性，通过MULTI和EXEC指令包起来。
    + 丰富的特性 – Redis还支持 publish/subscribe, 通知, key 过期等等特性
+ 与其他key-value存储结构的不同：
    + Redis有着更为复杂的数据结构并且提供对他们的原子性操作，这是一个不同于其他数据库的进化路径。Redis的数据类型都是基于基本数据结构的同时对程序员透明，无需进行额外的抽象。
    + Redis运行在内存中但是可以持久化到磁盘，所以在对不同数据集进行高速读写时需要权衡内存，因为数据量不能大于硬件内存。在内存数据库方面的另一个优点是，相比在磁盘上相同的复杂的数据结构，在内存中操作起来非常简单，这样Redis可以做很多内部复杂性很强的事情。同时，在磁盘格式方面他们是紧凑的以追加的方式产生的，因为他们并不需要进行随机访问。
+ 数据类型
    + 字符串：  
        Redis最基本的数据类型.<font color="red">可以包含任何数据类型，包含jpg图片和序列化的对象</font>单个value值得最大上限是1G，如果只用String类型，则人为redis加上了持久化特性（重启服务器之后，数据不消失）。
         + set 键名称 值 设置值 (重新设置则直接覆盖)
        + get 键名称 取值 (如果key不存在，则返回nil)
        + incr 键名称 对key的值做++操作，并返回一个新的值，每执行一次，值+1，值类型要是数据类型。
        + incrby 键名称 执行加法的操作，可以指定相加的值。
    + Hash：  
       可以用来存储对应的MySQL中一行的数据，类似于关联数组。
         + hset 哈希的名称(键名称) field(字段名称)   value 将对应的值设置进Hash表中
        ```
        hset user:id:1 name xxx
        :就表示一个普通符号，没有特殊含义
         ```
        + hget 哈希的名称(键名称) field(对应的字段名)
        + hmset 哈希名称(键名称) field1 value1 field2 value2 ...  一次性设置多个filed和value
        + hmget 哈希的名称 value1 value2... 一次性获取多个field的value
        + hgetall 哈希的名称 获取指定哈希中的所有field和value <font color="yellow">指定获得的是指定字段的键和值，而不是此表的所有信息</font>
    + 链表
        + List类型其实就是一个双向链表，通过pop、push操作从链表的头部或尾部添加或删除元素，<font color="red">这使得链表既可以用作栈，也可以用作队列</font>
        + 上进上出：栈 特点：数据 先进后出(从链表的头部添加元素)
        + 上进下出：队列 特点： 数据 先进先出(从链表的尾部添加元素)
        + lpush 链表名称(键的名称) 元素 从链表的头部添加元素
        + lrange 链表的名称 开始下标 结束下标 <font color="red">如果开始下标是0，结束下标是-1，则表示返回链表的所有元素；链表里面的元素是序号的(从0开始)，l类似于索引数组</font>(先进后出)
        + rpush 链表的名称(键的名称) 元素 从链表的尾部插入数据(先进先出) 
        + ltrim 链表的名称(键名称) 开始下标 结束下标 保留指定范围的元素
        + lpop 链表的名称 从链表的头部删除一个元素
    + 集合(无序性、唯一性)
        + 是String类型的无序集合
        + set元素可以包含(2^32-1)个元素
        + set集合可以进行简单的增、删外，还可以取交、并、差集。
        + <font color="red">每个集合中的元素不能重复</font>
        + sadd 集合名称(键名称) 元素名 向集合中添加元素
        + smembers 集合名 获取集合中的元素
        + sdiff 集合1 集合2 获取集合中的差集(在集合1中存在，但在集合2中不存在的元素) 应用：推荐好友
        + sinter 集合1 集合2 求两个集合的交集(在集合1和集合2中都存在) 应用：共同好友
        + sunion 集合1 集合2 求两个集合的并集(两个集合重复后，去掉重复的元素，留一个重复元素)
        + scard 集合名称 获取集合中元素的个数
    + 有序集合
        + 集合的升级版本,在set的基础上，添加了一个顺序属性，这一属性在添加和修改元素的时候可以指定，每次指定后，zset会自动重新按新的值调整顺序。
        + 操作中的key可以理解为zset的名字。
        + 元素由两部分组成：序号 值 
        + 取出有序集合里面的元素时，是根据序号排序后，取出的 
        + zadd 集合名 序号 内容 向有序集合中添加元素，如果元素存在，则更新其顺序
        + zrange 集合名称 开始下标 结束下标 按序号升序获取有序集合中的内容 <font color="red">开始下标、结束下标为排好序的一个索引</font>
        + zrevrange 集合名 开始下标(索引) 结束下标(索引) 按序号降序获取有序集合中的内容
    + HyperLogLog数据结构：用来做基数统计的算法
        + 优点：
            + 在输入元素的数量或者体积非常非常大时，计算基数所需要的空间总是固定的，并且是很小的。
            + 
        + 缺点： HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。
+ 常用命令：
    + 键值相关命令：
        + keys返回当前数据库里的键，可以使用通配符,*表示任意多个,？表示任意一个字符。
        + exists 键名称 判断一个键是否存在 返回1表示此键存在，0表示此键不存在
        + del 键名称 删除指定的键 键存在则返回1，不存在则返回0
        + expire key 有效期(秒数)  设置键的有效期 失效之后，返回的是nil
        + ttl key 返回一个键剩余的过期时间 
        + type key 返回键的数据类型
        + select 数据库的编号 切换数据库 选择数据库 在redis里面 默认有0-15号数据库，默认是0号数据库 可以在redis.conf里面进行设置
        + dbsize 返回当前数据库里面键的个数
        + flushdb 清空当前数据库里面所有的键
        + flushall 清空所有数据库里的所有的键
    + 服务器相关命令
        + redis启动：  
            + 安装目录下: redis-server.exe Redis.windows.conf
            + 新打开一个命令框: redis-cli.exe -h 127.0.0.1 -p 6379
            + 设置服务：redis-server --service-install redis.windows.conf --loglevel verbose
        + 卸载服务：redis-server --service-uninstall
        + 开启服务：redis-server --service-start
        + 停止服务：redis-server --service-stop
        
+ 安全验证
    + 设置客户端连接后进行任何其他操作都需要使用的密码
        + #requirepass 此时设置的密码是明文的，因此需要对配置文件进行严格的权限控制。
        + 设置密码后 需要使用密码来连接redis 登录到服务器之后 语句：auth 密码
# java使用Redis
+ 在这里使用的是maven项目构建。(无需下载jar包)，但是需要配置<denpendy>
```xml
<dependency>
      <groupId>redis.clients</groupId>
      <artifactId>jedis</artifactId>
      <version>2.9.0</version>
</dependency>
```
+ 测试代码
```java
 public static void main(String[] args) {
    Jedis jedis = new Jedis("127.0.0.1",6379);
    System.out.println("连接成功！");
    jedis.auth("123456789");
    jedis.set("name","梁蕊");
    System.out.println(jedis.get("name"));
    jedis.close();
}
```
+ Redis连接池测试代码
```java
    /**
    * Jedis连接池测试
    * 1、构建连接池配置信息
    * 2、设置最大连接数
    * 3、构建连接池
    * 4、从连接池中获取连接
    * 5、密码
    * 6、读取数据
    * 7、将连接还回到连接池
    * 8、释放连接池
    * */
 public static void JedisPoolDemo() {
    JedisPoolConfig config = new JedisPoolConfig();
    config.setMaxTotal(50);
    JedisPool jedisPool = new JedisPool(config,"127.0.0.1",6379);
    Jedis jedis = jedisPool.getResource();
    jedis.auth("123456789");
    System.out.println(jedis.get("name"));
    //包括了归还 因此归还已经弃用
    jedisPool.close();
}
```
+ 分片式集群的连接池
    + 集群：多个Redis服务，将多个Redis服务当做一个整体使用。
    + 多个Redis的原因：内存空间原因，服务器内存大小有上限。
    + 集群中放入数据和读取数据：是根据key的哈希值来选择服务器。（一致性Hash算法）
    + 可以解决单一服务器存储空间大小问题。但是，如果一个服务器出现问题，
    + 存在问题：服务节点没有办法动态的插入和删除。原因：服务器的查找是根据Hash值来实现的。(3.0之后又真正的集群)
    + 分片式集群与集群的区别：
        + 处于分片式集群中的两个Redis服务器根本不知道两者处于同一集群中；真正的集群知道。
    + 需要注意的是：密码的设置。
    ```java
    //分片式集群代码示例
    public static void JedisPoolTestDemo() {
        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(50);

        List<JedisShardInfo> jedisShardInfos = new ArrayList<>();
        JedisShardInfo jedisShardInfo = new JedisShardInfo("127.0.0.1",6379);
        jedisShardInfo.setPassword("123456789");
        jedisShardInfos.add(jedisShardInfo);

        ShardedJedisPool shardedJedisPool = new ShardedJedisPool(config,jedisShardInfos);

        ShardedJedis shardedJedis = shardedJedisPool.getResource();
        String value = shardedJedis.get("name");
        System.out.println(value);
    }
    ```
+ 分布式锁
    + 当多个进程不在同一个系统中，用分布式锁控制多个进程对资源的访问
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
