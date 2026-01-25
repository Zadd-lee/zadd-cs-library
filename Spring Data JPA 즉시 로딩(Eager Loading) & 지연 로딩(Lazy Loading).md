---
title: Spring Data JPA 즉시 로딩(Eager Loading) & 지연 로딩(Lazy Loading)
source: https://zzang9ha.tistory.com/347
updated: 2026-01-25T13:42
created: 2025-03-20T15:51
작성 완료: 1
---
## **📎 JPA - @ManyToOne 즉시 로딩과 지연 로딩(Eager Loading / Lazy Loading)**

Spring Data JPA에서 **@ManyToOne(N:1)**으로 연관관계가 설정되어 있는 2개의 Entity가 존재할 때, 

데이터베이스의 입장에서 보면 join이 필요합니다.

실제 **@ManyToOne**의 경우 **FK쪽의 엔티티를 가져올 때 PK쪽의 엔티티도 같이 가져오게 되는데요**, 이러한 과정이 꼭 필요한건지, 필요하지 않다면 어떻게 해결할 수 있는지 **즉시 로딩과 지연 로딩**에 대해 예제를 통해 살펴보겠습니다.

#### **⌨️ Board, Member 엔티티**

```java
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.ToString;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.ManyToOne;

@Entity
@Getter
@ToString(exclude = "member")
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Board extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long bno;

    @Column
    private String title;

    @Column
    private String content;

    @ManyToOne
    private Member member;
}

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.ToString;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.Id;

@Entity
@Getter
@ToString
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Member extends BaseEntity {

    @Id
    private String email;

    @Column
    private String password;

    @Column
    private String name;
}
```

위 코드에서 Board(게시글)와 Member(회원)은 @ManyToOne(N:1)의 관계를 맺고 있습니다.

**즉 한 명의 회원(1)은 여러 게시글(N)을 작성할 수 있습니다.**

따라서 게시글의 기준으로 봤을때 회원과의 관계는 N:1이 됩니다.

만약 Board의 엔티티를 조회한다면 Member의 엔티티도 함께 조회가 되는데요,

```java
    @Test
    @DisplayName("즉시 로딩은 연관관계를 맺고있는 엔티티는 모두 조회가 된다")
    void testRead1() {
        /* given */
        Optional<Board> result = boardRepository.findById(100L);

        /* when */
        Board board = result.get();

        /* then */
        System.out.println(board);
    }
```
![](https://blog.kakaocdn.net/dn/cDcmM6/btq7xSHRcHt/ummULQo9jDOSOE4WKbqdo0/img.png)

실행된 SQL 쿼리문을 확인해보면 board 테이블 외에도 member 테이블도 함께 조회를 하고 join으로 처리되는 것을 볼 수 있습니다.

만약 Board를 조회할 때 Member을 함께 조회하지 않으려면 어떻게 해야할까요?

****⌨️** 즉시 로딩(Eager Loading)과 지연 로딩(Lazy Loading)**

![](https://blog.kakaocdn.net/dn/xdbjc/btq7tscL5mr/L3lGNXqK1PLm0Hs4Pn1EJ0/img.png)

위 쿼리 결과와 같이 특정 엔티티를 조회할 때 연관된 모든 엔티티를 같이 로딩하는 것을 **즉시 로딩(Eager Loading)**이라고 합니다.

이와 같은 즉시 로딩은 연관된 엔티티를 모두 가져온다는 장점이 있지만,

실무에서 엔티티간의 관계가 복잡해질수록 **조인으로 인한 성능 저하**를 피할 수 없게 됩니다.

JPA에서 연관관계의 데이터를 어떻게 가져올 것인가를 **fetch(패치)**라고 하는데,

연관관계의 어노테이션 속성으로 'fetch'모드를 지정합니다.

'**즉시 로딩**'은 불필요한 조인까지 포함해서 처리하는 경우가 많기 때문에 '**지연 로딩**'의 사용을 권장하고 있습니다.

(보편적으로 '지연 로딩'을 기본으로 사용하고, 상황에 맞게 필요한 방법을 찾는것이 좋은 것 같습니다 😃)

**지연 로딩(Lazy Loading)**이란, 가능한 객체의 초기화를 지연시키는데 사용하는 패턴입니다.

**각 연관관계의 default 속성은 다음과 같습니다.**

**@OneToMany: LAZY**

**@ManyToOne: EAGER**

**@ManyToMany: LAZY**

**@OneToOne: EAGER**

[https://stackoverflow.com/questions/26601032/default-fetch-type-for-one-to-one-many-to-one-and-one-to-many-in-hibernate](https://stackoverflow.com/questions/26601032/default-fetch-type-for-one-to-one-many-to-one-and-one-to-many-in-hibernate)

지연 로딩은 Board 엔티티의 @ManyToOne 어노테이션에 적용을 할 수 있습니다.

![](https://blog.kakaocdn.net/dn/buLEke/btq8wKOYcJg/bjLWL5zMLRczyK6vtC4PL0/img.png)

```java
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.ToString;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.ManyToOne;

@Entity
@Getter
@ToString(exclude = "member")
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class Board extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long bno;

    @Column
    private String title;

    @Column
    private String content;

    // Lazy loading
    @ManyToOne(fetch = FetchType.LAZY)
    private Member member;
}

...
```
![](https://blog.kakaocdn.net/dn/J6jI1/btq7yisJjT3/bHaEx7kDlV5aa9tA2Iouc1/img.png)

Board 엔티티에 지연 로딩을 적용한 후 SQL 쿼리문을 확인해보면, 이전과는 다르게 Board 테이블만 조회가 됩니다.

하지만 지연 로딩을 적용한 후에는 **Board의 Member을 접근**하려고 하면 오류가 발생하는데요,

```java
    @Transactional
    @Test
    void testRead1() {
        /* given */
        Optional<Board> result = boardRepository.findById(100L);

        /* when */
        Board board = result.get();

        /* then */
        System.out.println(board);
        System.out.println(board.getMember()); // Error
    }
```
![](https://blog.kakaocdn.net/dn/JSGcf/btq7xBM9omn/qlKEgS1kEzryNATdurKJlK/img.png)

위 오류 내용인 **proxy ~ 'no Session'**은 데이터베이스와 추가적인 연결이 필요하다는 메시지 입니다.

지연 로딩 방식으로 로딩하기 때문에 board 테이블만 가져오는 것은 문제가 없지만, member 테이블을 가져올 때 문제가 발생합니다.

이러한 문제를 해결하기 위해서는 데이터베이스와의 재연결이 필요한데, **@Transactional** 어노테이션을 통해 해결할 수 있습니다.

**@Transactional** 어노테이션은 해당 메서드를 하나의 '트랜잭션' 으로 처리하라는 의미입니다.

트랜잭션으로 처리하면 속성에 따라 다르게 동작하지만, 기본적으로는 필요할 때 다시 데이터베이스와의 연결이 생성되기 때문에 위 테스트는 정상적으로 실행이 됩니다.

![](https://blog.kakaocdn.net/dn/eaM6aL/btq7yaavFoN/Zc3IgJbccXY7jgfnhoRM9k/img.png)

#### ****⌨️** 즉시 로딩(Eager Loading)과 지연 로딩(Lazy Loading)의 장단점**

**즉시 로딩(Earge Loading) 장점**

- 지연된 초기화와 관련해서 성능적인 영향이 없음

**즉시 로딩(Earge Loading) 단점**

- 지연 로딩보다 긴 초기의 로딩 시간이 필요함
- 불필요한 데이터를 많이 로딩하면 성능에 영향을 줄 수 있음

**지연 로딩(Lazy Loading) 장점**

- 다른 접근 방식보다 훨씬 적은 초기의 로딩 시간
- 다른 접근 방식에 비해 메모리 소비량 감소

**지연 로딩(Lazy Loading) 단점**

- 초기화가 지연되면 원하지 않는 순간 성능에 영향을 줄 수 있음

#### ****⌨️** 즉시 로딩(Eager Loading)과 지연 로딩(Lazy Loading)의 주의할 점**

- 가급적이면 **지연 로딩(Lazy Loading)**만 사용(특히 실무에서)
- 즉시 로딩(Eager Loading)을 적용하면 예상하지 못한 SQL이 발생할 수 있음
- 즉시 로딩(Earge Loading)은 JPQL에서 [**N+1 문제**](https://jojoldu.tistory.com/165)를 일으킴

#### **References**

- [http://www.yes24.com/Product/Goods/96051853](http://www.yes24.com/Product/Goods/96051853)
- [https://www.baeldung.com/hibernate-lazy-eager-loading](https://www.baeldung.com/hibernate-lazy-eager-loading)
- [https://stackoverflow.com/questions/26601032/default-fetch-type-for-one-to-one-many-to-one-and-one-to-many-in-hibernate](https://stackoverflow.com/questions/26601032/default-fetch-type-for-one-to-one-many-to-one-and-one-to-many-in-hibernate)
- [https://www.inflearn.com/course/ORM-JPA-Basic](https://www.inflearn.com/course/ORM-JPA-Basic)