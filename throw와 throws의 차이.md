---
created: 2025-11-12T09:27
updated: 2026-01-25T13:42
작성 완료: 1
---
[[예외처리]]와 관련된 예약어

# throw
예외를 강제적으로 발생시킨 후 상위 블럭이나 catch 문으로 예외를 던진다
```java
...
static public void myException(){
try{
//exception을 강제로 발생
throw new Exception();
...
```

# throws
예외가 발생하면 상위메서드로 예외를 던진다