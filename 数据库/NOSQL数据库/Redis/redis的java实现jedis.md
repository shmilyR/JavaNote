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