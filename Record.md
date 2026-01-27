---
created: 2026-01-27T10:56
updated: 2026-01-27T10:59
---

# Record
Record는 불변(Immutable) 데이터를 간결하고 읽기 쉽게 담는 데 초점을 맞추고 있으며, 기존 클래스(DTO 등)에서 필요했던 반복적인 코드를 많이 줄여줍니다.

```java
public record UserRecord(String name,int age,String email){
}
```
이 한줄로
생성자, 각 필드에 대한 getter 메서드, equals() 메서드, hashCode(),toString() 을 자동으로 생성해줌

## 왜 사용할까?
불필요한 코드 감소 : 반복적으로 작성되는 getter(),equals() 같은 코드를 사용할 필요 없어짐
불변성 보장 : 객체가 생성된 후 데이터를 변경할 수 없게 만들어줌
명확한 의도 : Record를 사용하면 해당 객체가 추가적인 동작이나 로직 없이 데이터를 전달하기 위한 것임을 명시할 수 있음
# 출처
- [출처1](https://yozm.wishket.com/magazine/detail/2814/)