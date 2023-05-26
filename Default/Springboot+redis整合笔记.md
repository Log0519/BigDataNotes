# Springboot+redis整合笔记

①. SPRINGBOOT整合REDIS
①. 引入data-redis-starter依赖
     <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-redis</artifactId>
   </dependency>
②. 简单配置redis的host等信息
spring:
  redis:
    host: hadoop102
    port: 6379
    password: fuxiaoluo




适配中文启动redis

redis-cli -a fuxiaoluo -p 6379 --raw

quit 退出

命令

keys * 查看所有key

get key 查看对应value

set key value



使用

第一步：编写service,里面写具体方法

```java
@Resource  
private StringRedisTemplate stringRedisTemplate;

public void addOrder(){  
 int keyId= ThreadLocalRandom.current().nextInt(1000)+1;  
 String serialNo= UUID.randomUUID().toString();  
 String key=ORDER_KEY+keyId;  
 String value="京东订单"+serialNo;  
 **stringRedisTemplate.opsForValue().set(key,value);**
 System.out.println(key+"===="+value);  
 log.info("***key:{}",key);  
 log.info("***value:{}",value);  
}
```



第二步：编写controller：设置接口和调用service中方法

```java
@Resource  
private OrderService orderService;

@ApiOperation("新增一张订单接口")  
@PostMapping("/order/add")  
public void addOrder(){  
 orderService.addOrder();  
}
```

第三步：在页面中使用request.post/get调用controller



使用StringRedisTemplate可以不反序列化，private StringRedisTemplate stringRedisTemplate;




