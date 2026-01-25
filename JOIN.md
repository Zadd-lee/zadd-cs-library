---
created: 2025-11-08T09:04
updated: 2026-01-25T13:42
작성 완료: 1
---
# JOIN
두 개 이상의 테이블을 관련된 필드들의 기준으로 결합하는 데 사용됨

# JOIN 종류
## inner join
두 테이블 간 공통된 값을 가진 행들만 반환
```sql
select * from tableA a
inner join tableB b
on a.key = b.key
```


## left join
왼쪽 테이블의 모든 행과 오른쪽 테이블에서 왼쪽 테이블과 공통된 값을 가진 행들을 반환
만약 오른쪽 테이블에서 공통된 값을 가진 행이 없다면 null 반환
```Mysql
select * from tableA a
left join tableB b
on a.key = b.key
```

## right join
left join과 반대로 오른쪽 테이블의 모든 행과 왼쪽 테이블의 공통된 값을 가진 행들을 반환
데이터가 없는 경우 null 반환
```Mysql
select * from tableA a
right join tableB b
on a.key= b.key
```
## outer join
조인하는 여러 테이블 중 한 쪽에는 데이터가 있고 한쪽에는 데이터가 없는 경우 데이터가 있는 쪽 테이블의 내용을 모두 출력

조건에 맞지 않아도 해당 행을 출력하고 싶을 때 사용할 수 있다

# 성능
성능에는 영향을 주지 않는다고 한다(?!)

확인해볼 자료
- [자료](https://blog.naver.com/shino1025/223124936920)
- [자료](https://schatz37.tistory.com/2)
