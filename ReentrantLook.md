---
created: 2025-12-10T09:03
updated: 2026-01-25T13:42
작성 완료: 1
---

java.util.concurrent.locks 패키지에 포함된 명시적 락 객체

```java
import java.util.concurrent.locks.ReentrantLock;

public class Counter {
    private int count = 0;
    private final ReentrantLock lock = new ReentrantLock();

    public void increment() {
        lock.lock();             // 락 획득
        try {
            count++;             // 임계 구역
        } finally {
            lock.unlock();       // 반드시 해제
        }
    }

    public int get() {
        lock.lock();
        try {
            return count;
        } finally {
            lock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Counter counter = new Counter();

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) counter.increment();
        });
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 10000; i++) counter.increment();
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println(counter.get()); // 항상 20000
    }
}

```
# 장점
유연한 락 제어 가능
정교한 제어 가능
# 단점
락 해제를 명시적으로 해야하므로 실수하면 데드락 위험 존재
코드가 ssynchronize에 비해 복잡
jvm 내부에서만 유효 (멀티 서버 x)
