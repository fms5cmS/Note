Java语言的Redis客户端：Jedis。

Jedis依赖于commons-pool，所以不用单独配置commons-pool。

```xml
<dependency>
  <groupId>redis.clients</groupId>
  <artifactId>jedis</artifactId>
  <version>2.9.0</version>
  <scope>compile</scope>
</dependency>
```

注意：如果要让Windows下的项目连接Linux下的Redis的话，需要在redis.conf中将`bind 127.0.0.1`注释掉，并关闭保护模式，即`protected_mode no`即可



# 常用操作

## 测试连通

```java
public class Demo01 {
  public static void main(String[] args) {
    //连接Linux上的的 Redis 服务
    Jedis jedis = new Jedis("192.168.214.132",6379);
    //查看服务是否运行，打出pong表示OK
    System.out.println("connection is OK==========>: "+jedis.ping());
  }
}
```



## 数据结构

```java
public class Test02 {
  public static void main(String[] args) {

    Jedis jedis = new Jedis("127.0.0.1",6379);
    //key
    Set<String> keys = jedis.keys("*");
    for (Iterator iterator = keys.iterator(); iterator.hasNext();) {
      String key = (String) iterator.next();
      System.out.println(key);
    }
    System.out.println("jedis.exists====>"+jedis.exists("k2"));
    System.out.println(jedis.ttl("k1"));
    //String
    //jedis.append("k1","myreids");
    System.out.println(jedis.get("k1"));
    jedis.set("k4","k4_redis");
    System.out.println("----------------------------------------");
    jedis.mset("str1","v1","str2","v2","str3","v3");
    System.out.println(jedis.mget("str1","str2","str3"));
    //list
    System.out.println("----------------------------------------");
    //jedis.lpush("mylist","v1","v2","v3","v4","v5");
    List<String> list = jedis.lrange("mylist",0,-1);
    for (String element : list) {
      System.out.println(element);
    }
    //set
    jedis.sadd("orders","jd001");
    jedis.sadd("orders","jd002");
    jedis.sadd("orders","jd003");
    Set<String> set1 = jedis.smembers("orders");
    for (Iterator iterator = set1.iterator(); iterator.hasNext();) {
      String string = (String) iterator.next();
      System.out.println(string);
    }
    jedis.srem("orders","jd002");
    System.out.println(jedis.smembers("orders").size());
    //hash
    jedis.hset("hash1","userName","lisi");
    System.out.println(jedis.hget("hash1","userName"));
    Map<String,String> map = new HashMap<String,String>();
    map.put("telphone","13811814763");
    map.put("address","atguigu");
    map.put("email","abc@163.com");
    jedis.hmset("hash2",map);
    List<String> result = jedis.hmget("hash2", "telphone","email");
    for (String element : result) {
      System.out.println(element);
    }
    //zset
    jedis.zadd("zset01",60d,"v1");
    jedis.zadd("zset01",70d,"v2");
    jedis.zadd("zset01",80d,"v3");
    jedis.zadd("zset01",90d,"v4");

    Set<String> s1 = jedis.zrange("zset01",0,-1);
    for (Iterator iterator = s1.iterator(); iterator.hasNext();) {
      String string = (String) iterator.next();
      System.out.println(string);
    }   
  }
}
```



## 事务提交

基本：

```java
public class Test03 {
  public static void main(String[] args) {
    Jedis jedis = new Jedis("127.0.0.1",6379);

    //监控key，如果该动了事务就被放弃
    /*3
     jedis.watch("serialNum");
     jedis.set("serialNum","s#####################");
     jedis.unwatch();*/

    Transaction transaction = jedis.multi();//被当作一个命令进行执行
    Response<String> response = transaction.get("serialNum");
    transaction.set("serialNum","s002");
    response = transaction.get("serialNum");
    transaction.lpush("list3","a");
    transaction.lpush("list3","b");
    transaction.lpush("list3","c");

    transaction.exec();
    //2 transaction.discard();
    System.out.println("serialNum***********"+response.get());       
  }
}
```

加锁：

```java
public class TestTransaction {
  public boolean transMethod() {
    Jedis jedis = new Jedis("127.0.0.1", 6379);
    int balance;// 可用余额
    int debt;// 欠额
    int amtToSubtract = 10;// 实刷额度

    jedis.watch("balance");
    //模拟其他程序已经修改了该条目 jedis.set("balance","5");
    balance = Integer.parseInt(jedis.get("balance"));
    if (balance < amtToSubtract) {
      jedis.unwatch();
      System.out.println("modify");
      return false;
    } else {
      System.out.println("***********transaction");
      Transaction transaction = jedis.multi();
      transaction.decrBy("balance", amtToSubtract);
      transaction.incrBy("debt", amtToSubtract);
      transaction.exec();
      balance = Integer.parseInt(jedis.get("balance"));
      debt = Integer.parseInt(jedis.get("debt"));

      System.out.println("*******" + balance);
      System.out.println("*******" + debt);
      return true;
    }
  }

  /**
   通俗点讲，watch命令就是标记一个键，如果标记了一个键， 在提交事务前如果该键被别人修改过，那事务就会失败，这种情况通常可以在程序中重新再尝试一次。
   首先标记键balance，然后检查余额是否足够，不足就取消标记，并不做扣减； 足够的话，就启动事务进行更新操作，
   如果在此期间键balance被其它人修改， 那在提交事务（执行exec）时就会报错， 程序中通常可以捕获这类错误再重新执行一次，直到成功。
   */
  public static void main(String[] args) {
    TestTransaction test = new TestTransaction();
    boolean retValue = test.transMethod();
    System.out.println("main retValue-------: " + retValue);
  }
}
```



## 主从复制

启动6379、6380，各自独立。

```java
public static void main(String[] args) throws InterruptedException {
  Jedis jedis_M = new Jedis("127.0.0.1",6379);
  Jedis jedis_S = new Jedis("127.0.0.1",6380);

  jedis_S.slaveof("127.0.0.1",6379);

  jedis_M.set("k6","v6");
  Thread.sleep(500);
  System.out.println(jedis_S.get("k6"));
}
```



# 配置

JedisPool的配置参数大部分是由JedisPoolConfig的对应项来赋值的。

- **maxActive**：控制一个pool可分配多少个jedis实例，通过pool.getResource()来获取；如果赋值为-1，则表示不限制；如果pool已经分配了maxActive个jedis实例，则此时pool的状态为exhausted。

- **maxIdle**：控制一个pool最多有多少个状态为idle(空闲)的jedis实例；

- whenExhaustedAction：表示当pool中的jedis实例都被allocated完时，pool要采取的操作；默认有三种。

  - WHEN_EXHAUSTED_FAIL --> 表示无jedis实例时，直接抛出NoSuchElementException；
  - WHEN_EXHAUSTED_BLOCK --> 则表示阻塞住，或者达到maxWait时抛出JedisConnectionException；
  - WHEN_EXHAUSTED_GROW --> 则表示新建一个jedis实例，也就说设置的maxActive无用；

- **maxWait**：表示当borrow一个jedis实例时，最大的等待时间，如果超过等待时间，则直接抛JedisConnectionException；

- **testOnBorrow**：获得一个jedis实例的时候是否检查连接可用性（ping()）；如果为true，则得到的jedis实例均是可用的；

- testOnReturn：return 一个jedis实例给pool时，是否检查连接可用性（ping()）；

- testWhileIdle：如果为true，表示有一个idle object evitor线程对idle object进行扫描，如果validate失败，此object会被从pool中drop掉；这一项只有在timeBetweenEvictionRunsMillis大于0时才有意义；

- timeBetweenEvictionRunsMillis：表示idle object evitor两次扫描之间要sleep的毫秒数；

- numTestsPerEvictionRun：表示idle object evitor每次扫描的最多的对象数；

- minEvictableIdleTimeMillis：表示一个对象至少停留在idle状态的最短时间，然后才能被idle object evitor扫描并驱逐；这一项只有在timeBetweenEvictionRunsMillis大于0时才有意义；

- softMinEvictableIdleTimeMillis：在minEvictableIdleTimeMillis基础上，加入了至少minIdle个对象已经在pool里面了。如果为-1，evicted不会根据idle time驱逐任何对象。如果minEvictableIdleTimeMillis>0，则此项设置无意义，且只有在timeBetweenEvictionRunsMillis大于0时才有意义；

- lifo：borrowObject返回对象时，是采用DEFAULT_LIFO（last in first out，即类似cache的最频繁使用队列），如果为False，则表示FIFO队列；

- 其中JedisPoolConfig对一些参数的默认设置如下：

  ```
  testWhileIdle=true
  minEvictableIdleTimeMills=60000
  timeBetweenEvictionRunsMillis=30000
  numTestsPerEvictionRun=-1
  ```



# JedisPool

|            | 优点                                                         | 缺点                                                         |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Jedis直连  | 简单方便；适用于少量长期连接的场景                           | 存在每次新建/关闭TCP的开销；<br/>资源无法控制，存在连接泄露的可能<br/>Jedis对象线程不安全 |
| Jedis Pool | Jedis预先生成，降低开销使用<br/>连接池的形式保护和控制资源的使用 | 使用相对麻烦，尤其在资源的管理上需要很多参数来保证，<br/>一旦规划不合理也会出现问题 |



```java
/**Redis连接池。项目中所有的配置建议从配置文件中获取*/
public class RedisPool {
  /**Jedis连接池。为了保证Jedis连接池在Tomcat启动时加载出来，所以声明为static*/
  private static JedisPool pool;

  /**最大连接数*/
  private static Integer maxTotal = 20;

  /**在Jedis Pool中最大的idle(空闲)状态得Jedis实例的个数*/
  private static Integer maxIdle = 10;

  /**在Jedis Pool中最小的idle(空闲)状态得Jedis实例的个数*/
  private static Integer minIdle = 2;

  /**在borrow一个Jedis实例的时候，是否进行验证操作，如果赋值为true，则得到的Jedis实例肯定是可用的*/
  private static Boolean testOnBorrow = true;

  /**在return一个Jedis实例的时候，是否进行验证操作，如果赋值为true，则放回Jedis Pool的Jedis实例肯定是可用的*/
  private static Boolean testOnReturn = true;

  /**这里没有设置默认值，是希望当连接错误时可以将错误报告出来*/
  private static String redisIp = "127.0.0.1";

  private static Integer redisPort = 6379;

  /**
     * 初始化Jedis连接池
     * 为防止外部调用，设为private
     */
  private static void initPool(){
    JedisPoolConfig config = new JedisPoolConfig();
    config.setMaxTotal(maxTotal);
    config.setMaxIdle(maxIdle);
    config.setMinIdle(minIdle);

    config.setTestOnBorrow(testOnBorrow);
    config.setTestOnReturn(testOnReturn);
    //连接耗尽时是否阻塞，false会抛出异常，true会阻塞直到超时。默认为true
    config.setBlockWhenExhausted(true);
    //超时时间设置为2秒
    pool = new JedisPool(config, redisIp, redisPort, 1000 * 2);
  }

  static {
    //使用静态块调用该方法，确保该方法仅被调用一次
    initPool();
  }

  /**从连接池中拿到一个Jedis实例*/
  public static Jedis getJedis(){
    return pool.getResource();
  }

  /**
     * 将Jedis实例返回给连接池
     * @param jedis 不再使用的Jedis实例
     */
  public static void returnResource(Jedis jedis){
    //源码中已经对参数进行了空判断，所以这里不再进行
    jedis.close();
  }
}
```



```java
public class Test {
  @Test
  public void testConnection(){
    Jedis jedis = null;
    try{
      jedis = RedisPool.getJedis();//获取Jedis实例需要从JedisPool中获取
      jedis.set("zzk", "zzzzz");      
    } catch (Exception e) {
      e.printStackTrace();
    }finally{
      //用完Jedis实例需要返还给JedisPool；如果Jedis在使用过程中出错，则也需要还给JedisPool
      RedisPool.returnResource(jedis);
    }
  }
}
```



# 分布式JedisPool

```java
public class RedisShardedPool {
  /** Sharded Jedis连接池。为保证Jedis连接池在Tomcat启动时加载出来，所以声明为static*/
  private static ShardedJedisPool pool;

  /**最大连接数*/
  private static Integer maxTotal = 20;

  /**在Jedis Pool中最大的idle(空闲)状态得Jedis实例的个数*/
  private static Integer maxIdle = 10;

  /**在Jedis Pool中最小的idle(空闲)状态得Jedis实例的个数*/
  private static Integer minIdle = 2;

  /**在borrow一个Jedis实例的时候，是否进行验证操作，如果赋值为true，则得到的Jedis实例肯定是可用的*/
  private static Boolean testOnBorrow = true;

  /**在return一个Jedis实例的时候，是否进行验证操作，如果赋值为true，则放回Jedis Pool的Jedis实例肯定是可用的*/
  private static Boolean testOnReturn = true;

  private static String redis1Ip = "127.0.0.1";
  private static Integer redis1Port = 6379;

  private static String redis2Ip = "127.0.0.1";
  private static Integer redis2Port = 6380;

  /**
     * 初始化Jedis连接池
     * 为防止外部调用，设为private
     */
  private static void initPool(){
    JedisPoolConfig config = new JedisPoolConfig();
    config.setMaxTotal(maxTotal);
    config.setMaxIdle(maxIdle);
    config.setMinIdle(minIdle);

    config.setTestOnBorrow(testOnBorrow);
    config.setTestOnReturn(testOnReturn);
    //连接耗尽时是否阻塞，false会抛出异常，true会阻塞直到超时。默认为true
    config.setBlockWhenExhausted(true);
    //超时时间设置为2秒
    JedisShardInfo info1 = new JedisShardInfo(redis1Ip, redis1Port, 1000 * 2);
    //如果Redis有密码的话，可以调用info1.setPassword("")来设置密码
    JedisShardInfo info2 = new JedisShardInfo(redis2Ip, redis2Port, 1000 * 2);
    //这里只放两个，所以指定了大小
    List<JedisShardInfo> jedisShardInfoList = new ArrayList<>(2);
    jedisShardInfoList.add(info1);
    jedisShardInfoList.add(info2);

    //MURMUR_HASH即为一致性哈希算法
    pool = new ShardedJedisPool(config, jedisShardInfoList, Hashing.MURMUR_HASH, Sharded.DEFAULT_KEY_TAG_PATTERN);
  }

  static {
    //使用静态块调用该方法，确保该方法仅被调用一次
    initPool();
  }

  /**
     * 从连接池中拿到一个Jedis实例
     * @return Jedis实例
     */
  public static ShardedJedis getJedis(){
    return pool.getResource();
  }

  /**
     * 将Jedis实例返回给连接池
     * @param jedis 不再使用的Jedis实例
     */
  public static void returnResource(ShardedJedis jedis){
    //源码中已经对参数进行了空判断，所以这里不再进行
    jedis.close();
  }
}
```

