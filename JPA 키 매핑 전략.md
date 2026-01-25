---
created: 2025-11-12T15:45
updated: 2026-01-25T13:42
source: https://coding-start.tistory.com/71
작성 완료: 1
---
# 키 매핑 전략
1. 직접 할당 : 애플리케이션에서 직접 엔티티 클래스의 @id 필드로 set 해준다
2. 자동 생성 : 대리 키 사용
	- identity : 기본 키 생성을 데이터베이스에 위임
	- sequence 데이터 베이스 시퀀스를 사용해 기본키 할당
	- table : 키 생성 테이블 사용

`@GeneratedValue(strategy= GenerationType. ...)` 으로 사용함

## identity 전략
기본 키 생성을 데이터베이스에 위임
주로 mysql, postgresSQL, SQL, Server, DB2에서 사용됨
```java
@Entity
@Getter
public calss Membeer{
	@Id
	@column(name="ID")
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private Long id;
	
	...
	}
```

### 단점
데이터베이스에 값을 저장하고나서야 기본 키 값을 구할 수 있다
	문제가 되는 이유 : [[JPA 영속성 컨텍스트]]에 1차 캐시를 하기 위해서는 구분자로 기본키 필드를 이용함. 즉 영속성 컨텍스트에 캐싱을 하기 위해 primary key 값을 가져오기 위해 테이블을 추가로 한번 더 조회하게 된다
다른 전략과 다른 행동 중 하나가 prisist() 호출을 하자마자 지연 쓰기를 하는 ㅓㄳ이 아니라, primary key 값을 가져오기 위해 바로 flush() 를 호출하게 됨

## sequence 전략
데이터베이스의 시퀀스 오브젝트를 이용하여 유일한 값을 순서대로 생성함
oracle, PostgresSQL, DB2,H2등 데이터베이스에서 사용할 수 있다
```java
...
@GeneratedValue(strategy=GenerationType.SEQUENCE
            ,generator="BOARD_SEQ_GENERATOR"
    )
    private Long id;
    
    @Column(name = "NAME",nullable=false,length=10)
    private String username;

    private Integer age;
    ...
```
IDENTITY와는** 다른 설정이 있다면 시퀀스생성기 어노테이션이다. 여기서 name속성은 실제 @Id필드에서 참조할 이름
sequenceName은 실제 데이터베이스에 생성되는 시퀀스 오브젝트 이름이다. 그리고 시퀀스 초기값과 allocationSize라는 속성이 있다. 여기서 allocationSize란 실제 데이터베이스에서
가져오는 시퀀스의 한번 호출에 증가하는 값의 크기이다

## table 전략
TABLE 전략은 키 생성 전용 테이블을 하나 만들고 여기에 이름과 값으로 사용할 컬럼을 만들어 데이터베이스 시퀀스를 흉내내는 전략이다.
이것은 벤더에 의존적이지 않은 전략이다.

이 전략을 사용할 때 auto DDL설정을 했다면 상관없지만 나중에 프로덕 환경에서의 데이터베이스 설계에서 꼭 시퀀스 테이블에 생성이 선행되어야한다.
```sql
CREATE TABLE APP_SEQUENCE(

sequence_name varchar2(255) primary key,

next_val number(22,0)

)
```
```java
...
	 @Id
	 @Column(name = "ID")
	 @GeneratedValue(strategy=GenerationType.TABLE, generator = "MEMBER_SEQ_GENERATOR")
	 @TableGenerator(
            name="MEMBER_SEQ_GENERATOR",
            table="MY_SEQUENCE", //시퀀스 생성용 테이블 이름
            pkColumnName="sequence_name", //MY_SEQUENCE 테이블에 생성할 필드이름(시퀀스네임)
            pkColumnValue="MEMBER_SEQ", //SEQ_NAME이라고 지은 칼럼명에 들어가는 값.(키로 사용할 값)
            allocationSize=50
    )

    private Long id;
...
```


