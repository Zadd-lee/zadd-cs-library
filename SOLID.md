---
created: 2026-01-30T09:14
updated: 2026-01-30T09:35
---

# SOLID
객체 지향 설계의 5원칙
유지보수, 확장, 이해가 쉬운 코드를 만들기 위한 기본 원칙

- SRP : 단일 책임 원칙
- OCP : 개방 폐쇄 원칙
- LSP : 리스코프 치환 원칙
- ISP : 인터페이스 분리 원칙
- DIP : 의존 역전 원칙

## SRP(Single Responsibility Principle)
> 하나의 클래스는 오직 하나의 책임만 가져야 한다
> 책임이란 하나의 역할만 가지고 있어야 하며, 변경의 이유도 한 가지 이유로만 변경해야 한다

**잘못된 설계**
```java
class UserService {
    public void createUser(String name) {
        // 사용자 생성 로직
    }
    public void sendWelcomeEmail(String email) {
        // 이메일 발송 로직
    }
}

```

→
**개선된 설계**
```java
class UserService{
	public void createUser(String name){
	...//사용자 생성 로직
	}}
```

## OCP (Open/Closed Principle)
> 소프트웨어 엔티티(클래스, 모듈, 함수)는 확장에 열려 있어야 하고 수정에는 닫혀 있어야 한다

**안좋은 설계**
```java
class DiscountService {
    public double getDiscount(String type) {
        if (type.equals("VIP")) {
            return 0.2;
        } else if (type.equals("Normal")) {
            return 0.1;
        }
        return 0.0;
    }
}

```
→
**개선된 설계**
```java
interface DiscountPolicy {
    double getDiscount();
}

class VipDiscount implements DiscountPolicy {
    public double getDiscount() {
        return 0.2;
    }
}

class NormalDiscount implements DiscountPolicy {
    public double getDiscount() {
        return 0.1;
    }
}

class DiscountService {
    public double getDiscount(DiscountPolicy policy) {
        return policy.getDiscount();
    }
}

```
## LSP(Liskov Substitution Principle)
> 서브타입(자식 클래스)는 언제나 기반타입(부모 클래스)로 교체할 수 있어야 한다
> 즉, 부모 클래스의 객체가 사용되는 곳에 자식 클래스 객체를 넣어도 프로그램 기능이 깨지지 않아야 한다

**잘못된 예제**
```java
class Bird{
	public void fly(){
		System.out.println("새가 날아간다");
	}
}

class Penguin extends Bird{
	@Override
	public void fly(){
		throw new Exception("펭귄은 날 수 없다");
	}
}
```
Penguin은 Bird를 상송했지만 fly()를 사용할 수 없다
## ISP (Interface Segregation Principle)
> 하나의 일반적인 인터페이스보다는 여러 개의 구체적인 인터페이스가 낫다.
> 클라이언트는 자신이 사용하지 않는 메서드에 의존하지 않아야 한다

**잘못된 설계**
```java
interface Worker {
    void work();
    void eat();
}

class Robot implements Worker {
    public void work() {}
    public void eat() { /* 로봇은 먹지 않음 */ }
}

```
→
**개선된 설계**
```java
interface Workable {
    void work();
}

interface Eatable {
    void eat();
}

class Human implements Workable, Eatable {
    public void work() {}
    public void eat() {}
}

class Robot implements Workable {
    public void work() {}
}

```
## DIP (Dependency Inversion Principle)
> 고수준 모듈은 저 수준 모듈에 의존해서는 안 되며, 둘 다 추상화에 의존해야 한다. 구체적인 구현 클래스가 아니라 인터페이스나 추상 클래스에 의존

**잘못된 설계**
```java
class MySQLDatabase {
    public void save(String data) {}
}

class UserService {
    private MySQLDatabase database = new MySQLDatabase();
    public void saveUser(String data) {
        database.save(data);
    }
}

```
→
**개선된 설계**
```java
interface Database {
    void save(String data);
}

class MySQLDatabase implements Database {
    public void save(String data) {}
}

class MongoDatabase implements Database {
    public void save(String data) {}
}

class UserService {
    private Database database;
    public UserService(Database database) {
        this.database = database;
    }
    public void saveUser(String data) {
        database.save(data);
    }
}

```

# 장점
- 유지 보수성 향상 : 변경이 필요한 부분이 최소화
- 확장성 강화 : 새로운 기능 추가 쉬움
- 코드 재사용성 증가 : 모듈 간 결합도가 낮아짐
- 테스트 용이성 : 단위 테스트와 Mocking이 쉬워진다
# 출처
- [출처1](https://coding-factory.tistory.com/1132)