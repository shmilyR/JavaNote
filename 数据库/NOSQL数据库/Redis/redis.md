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
        + setnx 键 值 分布式锁
        + expire 键 时间 设置过期时间
        + del 键 删除 
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
        + hscan 键 游标 
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
        + 优点：在输入元素的数量或者体积非常非常大时，计算基数所需要的空间总是固定的，并且是很小的。
        + 缺点： HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。
        + 	PFADD key element [element ...]  添加指定元素到HyperLogLog中
        + PFCOUNT key [key...] 返回给定HyperLogLog的技术估计值
        + PFMERGE destkey sourcekey [sourcekey...] 将多个HyperLogLog合并为一个HyperLogLog
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
# 发布订阅
+ 一种消息通信模式。发送者发送信息，接收者订阅信息。
+ Redis客户端可以订阅任意数量的频道。
+ 相关命令
    + 在一个客户端 负责订阅信息 SUBSCRIBE 频道名
    + 重开一个客户端 负责发布信息 PUBLISH 频道名 消息
    + PUBSUB subcommand 查看订阅与发布系统状态
    + PUNSUBSCRIBE 频道名 退订所有给定模式的频道
    + PSUBSCRIBE pattern 订阅一个或多个符合给定模式的频道
    + UNSUBSCRIBE 频道名 退订给定频道
+ java代码实现发布订阅
    + 订阅端
    ```java
    public class Subscribe_Client {
        public static void main(String[] args) {
            Jedis jedis = new Jedis("127.0.0.1",6379);
            jedis.auth("123456789");
            Subscribe subscribe = new Subscribe();
            jedis.subscribe(subscribe,"liang");
        }
    }
    //消息展示 主要是通过重写JedisPubSub的onMessage()方法来实现
    public class Subscribe extends JedisPubSub {
        @Override
        public void onMessage(String s,String s1) {
            System.out.println(s+"==========="+s1);
        }
    }
    ```
    + 发布端
    ```java
    public class Subscribe_Server {
        public static void main(String[] args) {
            Jedis jedis = new Jedis("127.0.0.1",6379);
            jedis.auth("123456789");
            jedis.publish("liang","欢迎您！");
        }
    }
    ```
    客户端发布一条消息时，订阅它的服务端就会同时收到所发布的信息.
    + 发布订阅思想的重要性：  
        应用场景
        + 注册发短信
        + 抢单：用户发起打车请求，将打车信息发给每个订阅的司机
# Redis事务  
<font color="red">事务里边的数据类型必须是String类型的，如果出现其他类型，将会出现错误，但不会发生回滚</font>
+ Redis事务可以一次执行多个命令，并且带有以下保证：
    + 批量操作在发送EXEC命令前被放入队列缓存；
    + 收到EXEC命令后进入事务执行，事务中任意命令执行失败，其余的命令依然被执行；
    + 在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。
+ 一个事务从开始到执行会经历以下阶段：
    + 开始事务；
    + 命令入队；
    + 执行事务。
+ 单个Redis命令的执行是原子性的，但Redis没有在事务上增加任何维持原子性的机制，所有，Redis事务的执行并不是原子性的。
+ 事务可以理解为一个打包的批量执行脚本，但批量指令并非原子化的操作，中间某条指令的失败不会导致前面已做指令的回滚，也不会造成后续的指令不做。
+ 事务命令：
    + MULTI 标志一个事务块的开始
    + EXEC 执行所有事务块内的命令
    + DISCARD 取消事务，放弃执行事务块内的所有命令
    + WATCH 键名 监视一个或多个key,如果在事务执行之前这个或这些key被其他命令所改动，那么事务将被打断<font color="yellow"> 只能在WATCH域内修改数据，其他事务不能修改被WATCH的数据(相当于加了锁)，除非将键unwatch.</font>
    + UNWATCH 取消WATCH命令对所有key的监视
+ Redis事务执行流程  
<img src="image/redis事务执行流程.PNG"></img>
+ 注意：但并不是所有的命令都会被放进事务队列，其中的例外就是EXEC、DISCARD、MULTI、WATCH，当这四个命令从客户端发送到服务器时，它们会像客户端处于非事务状态一样，直接被服务器执行。
<img src="image/redis事务.PNG"></img>
如果客户端正处于事务状态，那么当EXEC命令执行时，服务器根据客户端所保存的事务队列，以先进先出的方式执行事务队列中的命令，最先入队的命令最先执行，而最后入队的命令最后执行。  当事务队列里的所有命令被执行完之后，EXEC命令会将回复队列作为自己的执行结果返回给客户端，客户端从事务状态返回到非事务状态，至此，事务执行完毕。
+ Redis不支持回滚的原因：
    + 只有当被调用的Redis命令有语法错误时，这条命令才会执行失败(在将这个命令放入事务队列期间，Redis才会发现此类问题),或者对某个键执行不符合其数据类型的操作；实际上，这就意味着只有程序错误才会导致Redis命令执行失败，这种错误很有可能在程序开发期间发现，一般很少在生产环境发现。Redis已经在系统内部进行功能简化，这样可以确保更快的运行速度，因为Redis不需要事务回滚功能。
# Redis脚本
+ Redis脚本使用Lua解释器来执行脚本、
# Redis持久化