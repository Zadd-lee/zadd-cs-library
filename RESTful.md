---
created: 2026-01-28T06:59
updated: 2026-01-28T07:07
---

# RESTful
HTTP 기반의 REST 아키텍쳐 원리를 준수하여 설계된 인터페이스
자원을 명시적으로 URL로 식별하고
HTTP Method(GET,POST,PUT,DELETE)로 해당 자원의 상태를 관리하는 방식


## 특징
- 자원 중심 설계: URL은 행위가 아닌 명사형 리소스(Resource)를 사용해 자원을 식별
	`….com/create/user` (x) `…com/user url` (0)
- HTTP 메소드 활용
	GET : 데이터 조회
	POST : 데이터 저장
	PUT/PATCH : 데이터 수정
	DELETE : 데이터 삭제
- HTTP 상태 코드 사용 : 200(성공), 404(찾을 수 없음), 500(서버 에러)등 표준 코드를 통해 통신 상태를 명확히 전달
- 상태 비저장(stateless) : 각 요청은 독립적이며 서버는 클라이언트의 이전 상태를 저장하지 않음

## 장점
- 명확한 구조
- 높은 확장성 및 유연성

# 출처
- [출처1](https://hahahoho5915.tistory.com/54)