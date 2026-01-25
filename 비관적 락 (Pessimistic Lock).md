---
created: 2025-12-10T09:03
updated: 2026-01-25T13:42
작성 완료: 1
---

데이터 충돌이 자주 발생 예상해 데이터에 접근할 때 동시 수정을 막기 위해 미리 락

**데이터 조회 시 db 수준**의 락을 먼저 걸어둠
동시에 두 트랜젝션이 데이터를 수정할 수 없음
락을 잡고 있는 트랜잭션이 끝나야 다음 트랜잭션 진행 가능

```java
// Repository
public interface ProductRepository extends JpaRepository<Product, Long> {

    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("select p from Product p where p.id = :id")
    Product findByIdForUpdate(@Param("id") Long id);
}

```
```java
// Service
@Transactional
public void decreaseStock(Long productId, int quantity) {
    Product product = productRepository.findByIdForUpdate(productId);
    product.decrease(quantity);     // 수정 작업
}

```

# 장점
race condition 완벽 방지
데이터 무결정 보장
코드로 직접 동기화 구현 필요 없음
# 단점
 일반 조회도 락이 걸려 동시 처리 성능 저하(락 대기 시간 발생)
 데드락 위험 존재