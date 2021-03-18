# Springboot lettuce connection pool?



- [lettuce guide](https://lettuce.io/core/release/reference/)
- [lettuce guide - connection pooling](https://lettuce.io/core/release/reference/#_connection_pooling) 

레터스는 대부분의 경우 thread-safe 하게 설계 되었음.   

모든 Redis user operations는 single-thread로 동작함.   

multiple connections의 사용은 어플리케이션 성능에 영향을 주지 않음.  

Blocking operations의 사용은 일반적으로 전용 connection을 얻는 worker thread와 함께 사용됨.   

Redis transactions의 사용은 동적 Connection pooling의 전형적인 use case임. (그 만큼의 thread가 전용 connection을 요구함).  

즉, 동적 Connection pooling 대한 요구사항은 제한됨.  

Connection pooling은 항상 복잡성과 유지 관리 비용이 따름.   

---



- [jedis보다 lettuce를 쓰자](https://jojoldu.tistory.com/418)

  - 2019년 5월 작성. 성능을 비교했음.

- [Azure-Redis-Java-Bast-Practice](https://gist.github.com/warrenzhu25/1beb02a09b6afd41dff2c27c53918ce7#why-lettuce)

  - lettuce

    - transactions이나 blocking operation 같이 각각의 worker thread가 dedicated connections을 얻어야 하는 경우, 따라서 dedicated connection이 많이 필요한 경우 Connection pool을 사용하면 된다.
    - 레디스는 single threaded 이므로 많은 connection이 성능과 동시성을 향상시키는 건 아니다. 
    - 사용된 Connection은 다시 pool로 돌아가야 하므로 try-with-resource와 함꼐 써야함. 
    - 프로젝트가 react pattern이라면 의심의 여지없이 레터스의 react api를 사용할 것
    - netty 기반으로 구축된 multi threaded, event-driven I/O framework이므로 asynce api 나 sync api나 async하게 동작함. ㅋ

  - jedis

    - jedis 인스턴스는 not thread-safe

      

- [redis transaction with spring](https://ka0oll.tistory.com/25)

  ```java
  // RedisTemplate 이용한 트랜잭션
  List<Object> txResults = redisTemplate.execute(new SessionCallback<List<Object>>() {
    public List<Object> execute(RedisOperations operations) throws DataAccessException {
      operations.multi();
      operations.opsForSet().add("key", "value1");
  
      // This will contain the results of all operations in the transaction
      return operations.exec();
    }
  });
  
  
  // @Trascational 어노테이션 이용
  @Bean
  public StringRedisTemplate redisTemplate() {
    StringRedisTemplate template = new StringRedisTemplate(redisConnectionFactory());
    // explicitly enable transaction support
    template.setEnableTransactionSupport(true);              
    return template;
  }
  ```

---
## [timeout?](https://www.baeldung.com/spring-data-redis-properties)
-  lettuce는 redisConnectionFactory에 설정할 필요가 없다. 프로퍼티만 넘겨주면 springboot가 알아서 한다. 
`spring.redis.timeout=60000`a
  

  

