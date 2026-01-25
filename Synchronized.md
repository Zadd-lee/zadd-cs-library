---
created: 2025-12-10T09:03
updated: 2026-01-25T13:42
작성 완료: 1
---

하나의 스레드가 특정 코드 블록 또는 메서드를 실행할 때 다른 스레드로 진입을 막음

```java
public class DeadlockDemo {
    private final Object A = new Object();
    private final Object B = new Object();

    // 데드락 유발 가능한 메서드
    public void task1() {
        synchronized (A) {
            sleep(50);
            synchronized (B) { // 동시에 다른 스레드가 B->A 잡으면 교착
                System.out.println("task1 done");
            }
        }
    }

    public void task2() {
        synchronized (B) {
            sleep(50);
            synchronized (A) {
                System.out.println("task2 done");
            }
        }
    }

    private void sleep(long ms) {
        try { Thread.sleep(ms); } catch (InterruptedException ignored) {}
    }

    // 회피: 항상 같은 순서로 락 획득
    public void safeTask1() {
        synchronized (A) {
            synchronized (B) {
                System.out.println("safeTask1");
            }
        }
    }

    public void safeTask2() {
        synchronized (A) { // B 먼저가 아닌 A 먼저로 통일
            synchronized (B) {
                System.out.println("safeTask2");
            }
        }
    }
}

```
# 장점 
구현이 간단
jvm 기반 동기화(java 기본 제공, 외부 라이브러리 필요 없음)
동시에 작업이 수행되지 않도록 보장됨
# 단점
멀티 서버 환경에서 적용 불가능
락 경쟁이 많아지면 병목 발생 가능
비지니스 로직이 무거워 장시간 락 보유시 [[데드락]] 위험 존재