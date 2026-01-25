---
created: 2025-12-11T23:01
updated: 2026-01-25T13:42
source: https://www.maeil-mail.kr/question/27
작성 완료:
---
새로운 entity 인지 여부는 JpaEntityInformathin 의 isNew(T entity) 에 의해서 판단됨
다른 설정이 없으면 JpaEntityInformation의 구현체 중 JpaMetamodelEntityInformation 클래스가 동작합니다. `@Version`이 사용된 필드가 없거나 `@Version`이 사용된 필드가 primitive 타입이면 AbstractEntityInformation의 `isNew()`를 호출합니다. `@Version`이 사용된 필드가 wrapper class이면 null여부를 확인합니다.

# 왜 중요한가?
[[jpa save() 메서드]]에서 isNew()를 사용해 persist를 수행할지 merge를 수행할 지 결정함