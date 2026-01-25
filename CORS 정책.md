---
created: 2025-11-12T09:18
updated: 2026-01-25T13:42
작성 완료: 1
---
# CORS
Cross-Origin Resource Sharing'의 약자로,

==**서로 다른 출처(도메인, 프로토콜, 포트 등)**의 웹 페이지가 서버의 자원에 안전하게 접근할 수 있도록 허용하는 **웹 브라우저의 보안 정책**==


## Origin
origin은 출처라는 뜻으로 URL에서 Scheme, Host, Post까지를 의미합니다.

http 요청시 일부 요청에는 자동으로 origin 헤더가 추가되어 전달됩니다.

![Untitled](https://blog.kakaocdn.net/dna/bUuiYB/btsDhUgho4P/AAAAAAAAAAAAAAAAAAAAAOvJkMpVhJQ7CDtbtrNHa8SqfhfDUkOUyX_Cl0UOs99B/img.png?credential=yqXZFxpELC7KVnFOS48ylbz2pIh7yKj8&expires=1764514799&allow_ip=&allow_referer=&signature=SF7yQun%2FQqePH9mrT%2Bk7HVEe35A%3D)

- Protocol(Scheme) : http, https
- Host : 사이트 도메인
- Port : 포트 번호
- Path : 사이트 내부 경로
- Query string : 요청의 key와 value값
- Fragment : 해시 태크

Scheme, Host, Post 3가지만 동일하면 동일한 origin이라 합니다.
## SOP
SOP(Same Origin Policy) 정책은 단어 그대로 **동일한 출처에 대한 정책**을 말한다. 그리고 이 SOP 정책은 '동일한 출처에서만 리소스를 공유할 수 있다.'라는 법률을 가지고 있다. 

즉, 동일 출처(Same-Origin) 서버에 있는 리소스는 자유로이 가져올수 있지만, 다른 출처(Cross-Origin) 서버에 있는 이미지나 유튜브 영상 같은 리소스는 상호작용이 불가능하다는 말이다
