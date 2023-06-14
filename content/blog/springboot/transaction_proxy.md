---
title: '@Transaction 의 구조, 프록시'
date: 2023-06-11 18:37:00
category: 'springboot'
draft: false
---

# @Transaction 적용시 프록시 객체 주입

## 프록시 도입 전
- 트랜잭션을 처리하기 위한 프록시를 도입하기 전에는 서비스 로직에서 트랜잭션을 직접 시작했다.
- `1. 클라이언트` -> `2. 서비스(트랜잭션 처리 로직)` -> `3. 리포지토리(데이터 접근 로직)`
  - 2의 서비스에는 그냥 비즈니스 로직만 있는 것이 아니라 트랜잭션 시작, 끝의 로직에 다 들어있다.
  ```
  트랜잭션 시작

  비즈니스 로직

  트랜잭션 종료
  ```
  - 트랜잭션 사용 코드 예시
  ```java
  // 트랜잭션 시작
  TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());
  try {
    //비즈니스 로직
    bizLogic(fromId, toId, money); 
    transactionManager.commit(status); //성공시 커밋
  } catch (Exception e) { 
    transactionManager.rollback(status); //실패시 롤백
    throw new IllegalStateException(e);
  }
  ```

## 프록시 도입 후
- 트랜잭션을 처리하기 위한 프록시를 적용하면 트랜잭션을 처리하는 객체와 비즈니스 로직을 처리하는 서비스 객체를 명확하게 분리할 수 있다.
- `1. 클라이언트` --(프록시 호출)--> `2. 트랜잭션 프록시(트랜잭션 처리 로직)` ----> `3. 서비스(비즈니스 로직)` -----> `4. 리포지토리(데이터 접근 로직)`
- 처리하기 위한 프록시를 적용하면 트랜잭션을 처리하는 객체와 비즈니스 로직을 처리하는 서비스 객체를 명확하게 분리할 수 있다.
- 2단계의 트랜잭션 프록시 코드 예시 
```java
public class TransactionProxy {
    private MemberService target;

    public void logic() { //트랜잭션 시작
      TransactionStatus status = transactionManager.getTransaction(..);

      try {
        //실제 대상 호출 
        target.logic();
        transactionManager.commit(status); //성공시 커밋 
      } catch (Exception e) {
        transactionManager.rollback(status); //실패시 롤백
        throw new IllegalStateException(e);
      }
    }
}
```
- 트랜잭션 적용 후 3단계의 서비스 코드 예시
```java
public class Service {

  public void logic() {
    // 트랜잭션 관련 코드 제거, 순수 비즈니스 로직만 남음
    bizLogic(fromId, toId, money);
  }

}

```

## 트랜잭션 적용(프록시) 확인하기
- `AopUtils.isAopProxy()` 로 프록시 객체 확인가능
  - [스프링 공식 문서](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/aop/support/AopUtils.html#isAopProxy(java.lang.Object))
- `TransactionSynchronizationManager.isActualTransactionActive()` 로 트랜잭션 확인
  - [스프링 공식 문서](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/support/TransactionSynchronizationManager.html#isActualTransactionActive())

- 코드로 확인하기
  - spring boot 2.6,x, java11, gradle
  - lombok, spring data jpa, h2 database
```java
@Slf4j
@SpringBootTest
public class TxBasicTest {

  @Autowired
  BasicService basicService;

  @Test
  void proxyCheck() {
     
     //BasicService$$EnhancerBySpringCGLIB...
     log.info("aop class={}", basicService.getClass());
     assertThat(AopUtils.isAopProxy(basicService)).isTrue();
  }

  @Test
  void txTest() {
    basicService.tx();
    basicService.nonTx();
  }

  @TestConfiguration
  static class TxApplyBasicConfig {
    @Bean
    BasicService basicService() {
      return new BasicService();
    }
  }

  @Slf4j
  static class BasicService {
    
    @Transactional
    public void tx() {
      log.info("call tx");
      boolean txActive = TransactionSynchronizationManager.isActualTransactionActive();
      log.info("tx active={}, txActive";
    }

    public void nonTx() {
      log.info("call nonTx");
      boolean txActive = TransactionSynchronizationManager.isActualTransactionActive();
      log.info("tx active={}, txActive";
    }
  }


}

```
- proxyCheck() 메서드 실행 
- AopUtils.isAopProxy() : 선언적 트랜잭션 방식에서 스프링 트랜잭션은 AOP를 기반으로 동작한다.
- @Transactional 을 메서드나 클래스에 붙이면 해당 객체는 트랜잭션 AOP 적용의 대상이 되고, 결과적으로 실제 객체 대신에 트랜잭션을 처리해주는 프록시 객체가 스프링 빈에 등록된다. 그리고 주입을 받을 때도 실제 객체 대신에 프록시 객체가 주입된다.
클래스 이름을 출력해보면 basicService$$EnhancerBySpringCGLIB... 라고 프록시 클래스의 이름이 출력되는 것을 확인할 수 있다.
- proxyCheck() - 실행 결과
```
TxBasicTest : aop class=class ..$BasicService$$EnhancerBySpringCGLIB$$xxxxxx
```
- `스프링 컨테이너에 트랜잭션 프록시 등록 (매우 중요) - 위의 구조 설명`
  - @Transactional 애노테이션이 특정 클래스나 메서드에 하나라도 있으면 있으면 트랜잭션 AOP는 프록시를 만들어서 스프링 컨테이너에 등록한다. 그리고 실제 basicService 객체 대신에 프록시인 basicService$$CGLIB 를 스프링 빈에 등록한다. 그리고 프록시는 내부에 실제 basicService 를 참조하게 된다. 여기서 핵심은 실제 객체 대신에 프록시가 스프링 컨테이너에 등록되었다는 점이다.
  - 클라이언트인 txBasicTest 는 스프링 컨테이너에 @Autowired BasicService basicService 로 의존관계 주입을 요청한다. 스프링 컨테이너에는 실제 객체 대신에 프록시가 스프링 빈으로 등록되어 있기 때문에 프록시를 주입한다.
  - 프록시는 BasicService 를 상속해서 만들어지기 때문에 다형성을 활용할 수 있다. 따라서 BasicService 대신에 프록시인 BasicService$$CGLIB 를 주입할 수 있다.

- txTest() 실행
  - 실행하기 전에 먼저 다음 로그를 추가하자.
  - 로그 추가
      - application.properties
      ```
      logging.level.org.springframework.transaction.interceptor=TRACE
      ```
      - 이 로그를 추가하면 트랜잭션 프록시가 호출하는 트랜잭션의 시작과 종료를 명확하게 로그로 확인할 수 있다.

- basicService.tx() 호출
  - 클라이언트가 basicService.tx() 를 호출하면, 프록시의 tx() 가 호출된다. 여기서 프록시는 tx() 메서드가트랜잭션을사용할수있는지확인해본다. tx()메서드에는 @Transactional이 붙어있으므로 트랜잭션 적용 대상이다.
  - 따라서 트랜잭션을 시작한 다음에 실제 basicService.tx() 를 호출한다.
  - 그리고 실제 basicService.tx() 의 호출이 끝나서 프록시로 제어가(리턴) 돌아오면 프록시는 트랜잭션 로직을 커밋하거나 롤백해서 트랜잭션을 종료한다.

- TransactionSynchronizationManager.isActualTransactionActive()
  - 현재 쓰레드에 트랜잭션이 적용되어 있는지 확인할 수 있는 기능이다. 결과가 true 면 트랜잭션이 적용되어 있는 것이다. 트랜잭션의 적용 여부를 가장 확실하게 확인할 수 있다.

- 실행 결과
```
#tx() 호출
TransactionInterceptor          : Getting transaction for [..BasicService.tx]
y.TxBasicTest$BasicService      : call tx
y.TxBasicTest$BasicService      : tx active=true
TransactionInterceptor          : Completing transaction for [..BasicService.tx]

#nonTx() 호출
y.TxBasicTest$BasicService      : call nonTx
y.TxBasicTest$BasicService      : tx active=false
```
- 로그를 통해 tx() 호출시에는 tx active=true 를 통해 트랜잭션이 적용된 것을 확인할 수 있다.
- TransactionInterceptor 로그를 통해 트랜잭션 프록시가 트랜잭션을 시작하고 완료한 내용을 확인할
수 있다.
- nonTx() 호출시에는 tx active=false 를 통해 트랜잭션이 없는 것을 확인할 수 있다.





> 출처 : 인프런의 김영한 강의, 스프링 DB 2편 - 데이터 접근 활용 기술(스프링 트랜잭션 이해)