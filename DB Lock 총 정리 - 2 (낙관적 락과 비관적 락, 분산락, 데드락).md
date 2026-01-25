---
title: "[DB] Lock 총 정리 - 2 (낙관적 락과 비관적 락, 분산락, 데드락)"
source: https://velog.io/@soyeon207/DB-Lock-%EC%B4%9D-%EC%A0%95%EB%A6%AC-2-%EB%82%99%EA%B4%80%EC%A0%81-%EB%9D%BD%EA%B3%BC-%EB%B9%84%EA%B4%80%EC%A0%81-%EB%9D%BD-%EB%B6%84%EC%82%B0%EB%9D%BD-%EB%8D%B0%EB%93%9C%EB%9D%BD
작성 완료: 1
created: 2025-03-13T21:03
updated: 2026-01-25T13:42
---
## 낙관적 락과 비관적 락

## 낙관적 락 (optimistic Lock)

- 충돌이 발생하지 않는다고 낙관적으로 가정한다.
- DB 가 제공하는 락 기능이 아니라 어플리케이션에서 제공하는 버전 관리 기능을 사용한다.
- version 등의 구분 컬럼으로 충돌을 예방한다.
- 트랜잭션을 커밋하는 시점에 충돌을 알 수 있다.
- 최종 업데이트 과정에서만 락을 점유하기 때문에 락 점유 시간을 최소화하여 동시성을 높일 수 있다.

설명만 들었을 때에는 이해가 어려우니 예시를 들어보자.

두 트랜잭션(A, B) 가 거의 동시에 어떤 row 를 변경하려고 하는 상황으로 가정한다.

① A가 살짝 먼저 접근하고 바로 뒤이어 B가 접근한다.  
② A가 해당 ROW 와 version 를 UPDATE 한다. (선수치기)  
③ B가 커밋 시점에 해당 ROW를 업데이트 하려고 version 을 체크해 보니..  
처음과 다른 경우 어플리케이션은 예외를 발생시키고 첫번째 A의 커밋만 적용하여 정합성을 유지한다.  
④ 실패된 커밋은 어플리케이션 에서 후처리 한다.

### 루비 & JPA 에서 낙관적 락 사용하기

**👉 루비에서 낙관적 락 사용하기**  
테이블에 아래 쿼리를 사용해서 lock\_version 필드를 추가한다.

```ruby
t.integer :lock_version, default: 0 ## 기본값은 0 으로 설정해줘야 함
```

낙관적 락이 발생하는 경우에는 ActiveRecord::StaleObjectError 예외가 발생하고, 해당 에러를 catch 해서 어플리케이션에서 후처리해주면 된다.

**👉 JPA에서 낙관적 락 사용하기**  
@Version 을 통해 낙관적 락을 사용할 수 있다.  
낙관적 락이 발생하는 경우 ObjectOptimisticLockingFailureException 예외가 발생하고, 해당 에러를 catch 해서 어플리케이션에서 후처리해주면 된다.

- 충돌이 발생한다고 비관적으로 가정하는 방식
- Repeatable Read, Serializableable 정도의 격리성에서 가능하다.
- 트랜잭션이 시작될 때 S Lock 또는 X Lock을 걸고 시작한다.
- DB 가 제공하는 락 사용한다.
- 데이터 수정 즉시 트랜잭션 충돌을 알 수 있다.
- 교착 상태 문제가 자주 발생할 수 있다.

## 낙관적 락 vs 비관적 락

**트랜잭션**  
낙관적 락 (optimistic lock) 은 트랜잭션을 필요로 하지 않는다.

① 클라이언트가 서버에 정보를 요청  
② 서버에서는 정보를 반환  
③ 클라이언트에서 이 정보를 이용하여 수정 요청  
④ 서버에서는 수정 적용 (충돌 감지 가능)

의 경우에도 충돌 감지를 할 수 있다.

하지만, 비관적 락 (perssimistic lock) 같은 경우에는  
② 에서 해당 row 에 S-Lock 이 걸리고, ③ 에서는 해당 row 에 X-Lock 이 날라가기 때문에 트랜잭션을 유지 할 수 없다.

**성능의 측면**  
낙관적 락이 성능적으로 좋다.  
왜냐하면, 충돌이 일어나지 않을 거라고 가정하지만, 비관적 락은 충돌이 발생할거라고 생각하고 바로 Lock 을 걸어버리기 때문이다.  
그렇기 때문에 낙관적 락은 충돌이 많이 일어나지 않을 것이라고 예상되는 곳에 사용하면 좋은 성능을 기대할 수 있다.

**롤백**  
비관적 락의 경우 트랜잭션만 롤백해주면 되지만,  
낙관적 락의 경우에는 충돌이 발생하면 개발자가 수동으로 롤백처리를 하나하나 해줘야 한다는 단점이 있다.

---

## 분산락 (Distributed lock)

- 서버가 여러 대인 상황에서 동일한 데이터에 대한 동기화를 보장하기 위해 사용한다.
- 서버들 간 동기화된 처리가 필요하고, 여러 서버에 공통된 락을 적용해야 하기 때문에 redis 를 이용하여 분산락을 이용한다.
- 분산락 같은 경우 공통된 데이터 저장소(DB)를 이용해 자원이 사용중인지 확인하기 때문에 전체 서버에 동기화된 처리가 가능하다.

## Redis 기반 분산락 사용 하기

### ① Lettuce

스핀락 (만약 다른 스레드가 Lock 을 소유하고 있다면 그 lock이 반환될 때까지 계속 확인 하며 기다리는 것) 을 사용하여 구현 가능 하다.

Lock 을 점유하는 시간이 짧을 경우 유용하지만 스레드가 Lock 을 오래 유지하는 경우 CPU 에 부담을 줄 수 있다.

샘플 코드는 아래와 같이 사용할 수 있다.

```java
void doProcess() {
    String lockKey = "lock";

    try {
        while (!tryLock(lockKey)) { // ②
            try {
                Thread.sleep(50);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
        }
        
        // (3) do process
    } finally {
        unlock(lockKey); // (4)
    }
}

boolean tryLock(String key) {
    return command.setnx(key, "1"); // ①
}

void unlock(String key) {
    command.del(key);
}
```

출처 : [레디스와 분산 락(1/2) - 레디스를 활용한 분산 락과 안전하고 빠른 락의 구현](https://hyperconnect.github.io/2019/11/15/redis-distributed-lock-1.html)

① redis 의 setnx 는 동일한 key 가 없을 경우에만 저장한다.  
② 락을 획득 즉, redis 에 key 를 저장할 때까지 while 반복문으로 계속 락 획득을 시도하고, 락을 획득한 경우에만 로직을 수행한다.  
③ 로직을 모두 수행 한 후에 redis 에서 해당 키 삭제하는 걸로 락을 해제한다.

하지만, 해당 방식에도 2가지의 문제점이 존재한다.

첫번째, Lock 타임아웃 설정이 되지 않는다.  
락을 획득하지 못한 경우엔 무한루프를 돌게 된다. 그렇기 때문에 일정 시간이 지나면 락이 만료되도록 해줘야 하는데 setnx 명령어는 expire time 을 설정할 수 없다.

두번째, Redis 에 많은 부하가 발생한다.  
스핀락은 지속적으로 락의 획득을 시도하기 때문에 Redis 에 많은 부담을 줄 수 밖에 없다.

### ② Redisson

Redisson 은 Lettuce 방식과는 다르게 스핀락 방식이 아니라 pub/sub 방식을 사용하기 때문에 Redis 에 가는 부하를 최대한으로 줄일 수 있다.

> **pub/sub 방식이란 ?**  
> 채널을 구독한 subscribe 에게 락이 해제될 때마다 메시지를 전송하는 방식 이다.  
> 구독자들에게 바로 메세지를 전달하기 때문에 메시지를 보관하지는 않는다.  
> 어느 채널을 통해서 메시지를 전달해준다고 생각하면 된다.  
> `redis-cli -> subscribe test -> publish test "테스트입니다"` 로 redis 에서 pub/sub 방식을 테스트해볼 수 있다.

Lock 획득 프로세스는 다음과 같다. 👇

1. 대기없는 tryLock 을 통해 락 획득에 성공하면 true 를 반환한다.  
이는 경합이 없을 때 아무런 오버헤드 없이 락을 획득할 수 있도록 해준다.
2. pub/sub을 이용하여 메세지가 올 때까지 대기하다가 락이 해제되었다는 메세지가 오면 대기를 풀고 다시 락 획득을 시도한다.  
락 획득에 실패하면 다시 락 해제 메세지를 기다린다.  
이 프로세스를 타임아웃시까지 반복한다.
3. 타임아웃이 지나면 최종적으로 false를 반환하고 락 획득에 실패했음을 알려준다.  
대기가 풀릴 때 타임아웃 여부를 체크하므로 타임아웃이 발생하는 순간은 파라미터로 넘긴 타임아웃시간과 약간의 차이가 있을 수 있을 수 있다.

**특징**

① Lock에 타임아웃을 설정할 수 있다.

```java
public boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException
```

waitTime : 락을 얻기 위한 대기 시간  
leaseTime : 락이 만료되어 사라지는 시간 지정

② Lua 스크립트를 사용한다.  
락 획득가능 여부, 획득, pubsub 알림은 atomic 해야한다.  
Lua 스크립트 자체가 하나의 큰 명령으로 해석되기 때문에 atomic 하게 처리 된다.

## MySQL 기반 분산락 적용

Redis 뿐만 아니라 MySQL 을 사용해서도 분산락을 적용할 수 있다.  
아래 예시는 Spring 에서 MySQL 을 적용하는 방법에 대한 예시이다.

① ExclusiveLock 테이블을 생성한다.  
② repository에 분산락 관련 코드를 설정해준다.

```java
@Lock(LockModeType.PESSIMISTIC_WRITE)
@QueryHints({@QueryHint(name = "javax.persistence.lock.timeout", value = "300000")})
@Override
Optional<ExclusiveLock> findById(Long id);
   
@Lock(LockModeType.PESSIMISTIC_WRITE)
@QueryHints({@QueryHint(name = "javax.persistence.lock.timeout", value = "300000")})
Optional<ExclusiveLock> findByCode(String code);
```

③ ExclusiveLock.find 후 update 해준다.

➡️ find 할 때 비관적 락이 걸리 기 때문에 순서가 보장 된다.

---

## 교착상태 (데드락)

두 트랜잭션이 각각 Lock을 설정하고 다음 서로의 Lock에 접근하여 값을 얻어오려고 할 때 이미 각각의 트랜잭션에 의해 Lock이 설정되어 있기 때문에 양쪽 트랜잭션 모두 영원히 처리가 되지않게 되는 상태이다.

예를 들어서  
트랜잭션 A 가 Resource1 데이터를 수정(X-Lock 발생) 하고 Resource2 데이터를 수정한다.  
트랜잭션 B 는 Resource2 데이터를 수정(X-Lock 발생)하고 Resource1 데이터를 수정한다.

A 트랜잭션이 Resoucre1 데이터를 수정하는 작업과 B 트랜잭션이 Resource2 데이터를 수정하는 작업이 동시에 일어난 경우 데드락이 발생할 수 있는데,

A는 Resource2 에 베타 Lock 을 B 는 Rescoure1 에 베타 Lock 을 걸려고 하는데 이미 베타 Lock 이 걸려 있으니 영원히 풀리지 않게 된다.

해당 과정을 그림으로 나타내면 아래와 같다.

![https://t1.daumcdn.net/cfile/tistory/243E89355714C26E28](https://t1.daumcdn.net/cfile/tistory/243E89355714C26E28)

그렇기 때문에 교착상태 (DeadLock) 이 발생하는 경우 보통 DBMS 가 한 트랜잭션에 에러를 일으키면서 문제를 해결한다.

## DeadLock 발생 조건

한 시스템 내 네가지 조건이 동시에 성립할 때 데드락이 발생한다.  
결국, 하나라도 성립하지 않도록 한다면 데드락은 해결 가능하다.

① 상호 배제  
한번에 하나의 프로세스만 해당 자원을 사용할 수 있다. 즉, 자원을 동시에 사용할 수 없는 경우  
👉 해결방법 : 여러 개의 프로세스가 공유 자원을 사용하도록 함

② 점유 대기  
자원을 붙잡은 상태로 다른 자원을 기다리고 있다. 자원을 최소한 하나 보유하고, 다른 프로세스에 할당된 자원을 점유하기 위해 대기하는 프로세스가 존재해야 한다  
👉 해결방법 : 프로세스가 실행되기 전 필요한 모든 자원을 할당

③ 비선점  
이미 할당된 자원을 강제로 빼앗을 수 없다  
👉 해결방법 : 자원을 점유하고 있는 프로세스가 다른 자원을 요구할 때 점유하고 있는 자원을 반납하고, 요구한 자원을 사용하기 위해 기다리게 한다.

④ 순환 대기  
대기 프로세스의 집합이 순환 형태로 자원을 대기해야 한다  
👉 해결방법 : 자원에 고유한 번호를 할당하고, 번호 순서대로 자원을 요구하도록 한다

## 데드락(Deadlock)의 해결법

데드락의 해결법은 크게 3가지로 나눌 수 있다.

1. 데드락이 발생하지 않도록 **예방**하기  
데드락의 발생조건 4가지 중 하나라도 발생하지 않게 하는 것  
위의 해결 방법 중 하나를 따르면 된다.  
하지만 효율성을 떨어뜨릴 수 있다.
2. 데드락 발생 가능성을 인정하면서도 적절하게 **회피**하기  
교착 상태가 발생하기 전에 교착 상태를 예상하여 안전한 상태에서만 자원 요청을 허용하는 방법
3. 데드락 발생을 허용하지만 데드락을 **탐지**하여, 데드락에서 **회복**하기

## 레퍼런스 🙇‍♀️

[\[database\] 낙관적 락(Optimistic Lock)과 비관적 락(Pessimistic Lock)](https://sabarada.tistory.com/175)  
[레디스와 분산락](https://happyer16.tistory.com/entry/%EB%A0%88%EB%94%94%EC%8A%A4%EC%99%80-%EB%B6%84%EC%82%B0%EB%9D%BD)  
[\[운영체제\] 스핀락(Spin Lock)이란?](http://itnovice1.blogspot.com/2019/09/spin-lock.html)  
[레디스와 분산 락(1/2) - 레디스를 활용한 분산 락과 안전하고 빠른 락의 구현](https://hyperconnect.github.io/2019/11/15/redis-distributed-lock-1.html)  
[Atomic 처리와 cache stampede 대책을 위해 Redis Lua script를 활용한 이야기](https://engineering.linecorp.com/ko/blog/atomic-cache-stampede-redis-lua-script/)  
[\[운영체제\] 데드락(Deadlock, 교착 상태)이란?](https://chanhuiseok.github.io/posts/cs-2/)