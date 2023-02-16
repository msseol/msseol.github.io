---
layout: post
title:  "자원의 동시접근제어 관련 메모"
date:   2023-01-25 00:00:01
categories: DATABASE
tags: spring redis redission lock
cover: redis.png
---

대부분 데이터와 자원 설계시 자원 접근에 대해 서로 영향을 최소화 하게 하여 문제를 회피할 수 있지만 불가피하게 동시접근이 필요한 경우가 있다.
이럴 때 별다른 처리를 해주지 않으면 데이터의 무결성을 보장할 수 없게 되므로 별도의 처리가 필요해진다.   

이럴때 사용하는 자원 접근은 다양한 방식이 있는데 잘 정리된 포스트가 있어 정리하여 남긴다. [원본보기][from]

---

#### Dummy Table

pk 하나만 존재하는 더미 테이블을 생성하며 auto_increment를 이용하여 값을 획득 한다. 이후 LAST_INSERT_ID() 를 통해 커넥션별로 고유한 값을 얻어 올 수 있다.

mybatis를 사용한다면 아래와 같이 사용한다.

```xml
<insert id="insertDummyTable">
    INSERT INTO dummy_table values (default)
    <selectKey keyProperty="id" resultType="int" order="AFTER">
        SELECT LAST_INSERT_ID()
    </selectKey>
</insert>
```

#### UUID

application에서 고유값을 얻는 방법이다. UUID는 Universally Unique IDentifier의 약자이며 네트워크 상에서 고유 id를 보장하기 위한 규약이다. 128비트의 숫자이며 8-4-4-4-12 글자마다 하이픈을 집어넣어 5개의 그룹으로 표현한다. 생성 규칙에 따라 4가지 버전이 존재하며 MAC주소를 이용하기에 해쉬 충돌이 발생한 가능성은 겨우 10억분의 1(버전4 기준) 이다.

```java
var uuid = UUID.randomUUID().toString(); // 4717170d-8a5e-4310-bfcc-609649fdc666
BigInteger bigInteger = 
  new BigInteger(uuid.replace("-", ""), 16); //4717170d8a5e4310bfcc609649fdc666
System.out.println(bigInteger); // 94495078096685211775394821825447708262
spring을 사용한다면 혹시 모를 상황(10억분의 1…)을 대비하여 spring-retry를 적용해보는 것도 좋다.

public interface MyService{
  @Retryable(value = DuplicatekeyException.class)
  void addUniqueValue(Model model)
}
```

기존에 생성된 데이터가 해당 숫자와 충돌할 가능성을 생각하여 반려하였습니다만, 처음부터 설계 했더라면 UUID 도입을 적극 고려했을 것 같다.

#### Redis

In-Memory DB인 redis를 이용하는 방식이다. 2가지 정도의 방식이 있다.

**INCR Key**

redis는 싱글쓰레드 기반이며 모든 명령어는 queue에 담겨 순차적으로 진행된다. INCR 명령어 사용시 atomic value 여부를 고민할 필요가 없다. 자세한 사용법은 레디스 공식 문서를 참고한다.

```bash
127.0.0.1:6379> set unique_key 0
OK
127.0.0.1:6379> get unique_key
"0"
127.0.0.1:6379> incr unique_key
(integer) 1
127.0.0.1:6379> incr unique_key
(integer) 2
```

spring에서 redisTemplate를 사용한다면 아래와 같이 작성한다.

```java
long uniqueKey = redisTemplate.opsForValue().increment("unique_key");
```

**Redisson**

간단하게 key를 얻는것이 아니라 lock 자체를 획득해야 한다면 redssion을 고려해보는 것도 좋은 방법이다. redisson은 java 기반의 구현체이며 transaction, lock 등을 제공한다. 추가적으로 pub/sub 방식으로 lock을 획득하기에 성능상의 이점 또한 가져올 수 있다.

```java
RedissonClient redissonClient = new RedissonClient();
String key = "unique_key";
RLock rLock = redissonClient.getLock(key);
try {
  if (rLock.tryLock(1000, 2000, TimeUnit.SECONDS)) { // ... (1)
      RTransaction rTransaction =
          redissonClient.createTransaction(TransactionOptions.defaults().timeout(1000, SECONDS));
      // doSomething()       // ... (2)
      rTransaction.commit(); // ... (3)
  }
} finally {
    rLock.unlock(); // ... (4)
}
```

(1): timeout을 지정하지 않으면 무한루프 발생할 수 있음   
(2): 비즈니스 로직을 작성   
(3): commit을 해야 해당 내용을 반영할 수 있음   
(4): lock을 반납하지 않으면 dead-lock 발생 할 수 있음   

lock 획득시 주의사항을 반드시 지켜야 하며 혹시라도 redis서버가 다운됐을시의 전략 또한 중요한다.

#### MySQL

redis를 사용한다면 위와 같은 방식으로 간편하게 처리 할 수 있지만, 비용상의 문제등으로 새롭게 서버를 추가할 수 없다면 MySQL의 user level lock을 활용하는 것도 하나의 방법이다. user_level_lock을 사용하면 keyword을 사용하여 락의 획득이 가능한다.

```java
Class.forName("com.mysql.cj.jdbc.Driver");
Connection connection = DriverManager.getConnection(url, id, passwd);
Statement statement = connection.createStatement();

ResultSet resultSet = statement.executeQuery("SELECT GET_LOCK('unique', 3)");
//doSomenthing()
statement.execute("SELECT RELEASE_LOCK('unique')");
```

주의 할 점은 반드시 RELEASE_LOCK 을 호출해야 한다는 점 이다. 만약 호출하지 않고 connection을 끊거나 thread 종료시 kill 명령어등을 사용하여 해당 락을 직접 제거 해야 한다. 따라서 spring이나 dbcp 등 에서 사용할 경우 exception 발생시에 해당 connection에서 RELEASE_LOCK 을 호출하도록 작성한다.

#### JdbcLockRegistry

spring을 사용한다면 JdbcLockRegistry사용도 고려해봄직 한다. spring-integration의 하위 모듈로 간편하게 lock을 획득할 수 있으며 기존의 선언적 트랜잭션(@Transactional) 또한 지원 한다.

사용하는 datasource에 맞춰 table을 생성한다.

```yml
dependencies {
  ...
  implementation 'org.springframework.boot:spring-boot-starter-integration'
  implementation 'org.springframework.integration:spring-integration-jdbc'
  implementation 'org.springframework.integration:spring-integration-core'
  ...
}
```

관련 라이브러를 추가한다.

```java
@Bean
public LockRepository lockRepository(DataSource datasource) {
    return new DefaultLockRepository(datasource);
}

@Bean
public JdbcLockRegistry jdbcLockRegistry(LockRepository repository) {
    return new JdbcLockRegistry(repository);
}
```

bean을 등록한다.

```java
@Autowired
private JdbcLockRegistry lockRegistry;

@Transactional
public void uniqueJob() throws InterruptedException {
    String key = "unique";
    Lock lock = lockRegistry.obtain(key);
    try {
        if (lock.tryLock(5, TimeUnit.SECONDS)) { // ... (1)
          // doSomething() ... (2)
        }
    } finally {
        lock.unlock(); // ... (3)
    }
}
```

(1): 해당 시간동안 대기 후 lock 미 획득시 InterruptedException 발생   
(2): 비즈니스 로직 수행   
(3): lock 해제   

---

[출처][from]   
[관련참조1][ref1]   
[관련참조2][ref2]   
[관련참조3][ref3]  
[관련참조4][ref4]  
[관련참조5][ref5]   

각각 상황에 알맞는 방식을 적용하면 좋을 것 같다.

---

#### 기타 메모

**Redis event notification**

redis key 관련 이벤트를 수신할 수 있다. [링크][link]

[from]: https://hahava.github.io/dev_log/devlog-distributed-key

[ref1]: https://github.com/spring-projects/spring-integration/tree/e901c89fef3eea00ddf6d503ae9926667a1d6972/spring-integration-jdbc/src/main/resources/org/springframework/integration/jdbc

[ref2]: https://velog.io/@hgs-study/redisson-distributed-lock

[ref3]: https://tanzu.vmware.com/developer/guides/spring-integration-lock/

[ref4]: https://hyperconnect.github.io/2019/11/15/redis-distributed-lock-1.html

[ref5]: https://techblog.woowahan.com/2631/

[link]: https://moonsiri.tistory.com/87