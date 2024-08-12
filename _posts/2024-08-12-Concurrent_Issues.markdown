---
layout: post
title:  "동시성 문제(Concurrent Issues)"
date:   2024-08-12 21:20:00 +0900
categories: architecture
---

동시성 문제는 멀티스레드 또는 멀티프로세스 환경에서 여러 스레드 또는 프로세스가 동시에 실행될 때 발생할 수 있는 문제들을 말합니다. 이러한 문제들은 시스템의 예측 가능성과 안정성을 저하시킬 수 있습니다. 주요 동시성 문제로는 데드락(교착 상태), 레이스 컨디션(경쟁 상태), 라이브락(활성 상태 교착), 기아 상태 등이 있습니다.

### 1. 데드락 (Deadlock)
#### 정의
* 데드락은 두 개 이상의 스레드가 서로가 필요로 하는 자원을 가지고 있어, 서로가 자원을 해제하기를 기다리면서 무한히 대기하는 상태입니다.

#### 조건
데드락이 발생하기 위한 네 가지 조건(필수 조건)은 다음과 같습니다:

* 상호 배제(Mutual Exclusion): 자원은 한 번에 하나의 스레드만 사용할 수 있습니다.

* 점유와 대기(Hold and Wait): 자원을 점유한 스레드가 다른 자원을 기다리며 점유한 자원을 놓지 않습니다.

* 비선점(No Preemption): 자원을 강제로 빼앗을 수 없습니다.

* 순환 대기(Circular Wait): 두 개 이상의 스레드가 자원을 대기하는 동안 서로 순환적으로 대기합니다.

#### 예제
```java
public class DeadlockExample {
    public static void main(String[] args) {
        final Object resource1 = "resource1";
        final Object resource2 = "resource2";

        Thread t1 = new Thread(() -> {
            synchronized (resource1) {
                System.out.println("Thread 1: locked resource 1");

                try {
                    Thread.sleep(50);
                } catch (InterruptedException e) {
                }

                synchronized (resource2) {
                    System.out.println("Thread 1: locked resource 2");
                }
            }
        });

        Thread t2 = new Thread(() -> {
            synchronized (resource2) {
                System.out.println("Thread 2: locked resource 2");

                try {
                    Thread.sleep(50);
                } catch (InterruptedException e) {
                }

                synchronized (resource1) {
                    System.out.println("Thread 2: locked resource 1");
                }
            }
        });

        t1.start();
        t2.start();
    }
}
```

### 2. 레이스 컨디션 (Race Condition)
#### 정의
* 레이스 컨디션은 두 개 이상의 스레드가 공유 자원에 동시에 접근할 때 발생하며, 실행 순서에 따라 결과가 달라지는 상태를 말합니다. 이는 데이터 무결성 문제를 일으킬 수 있습니다.

#### 예제
```java
public class RaceConditionExample {
    private int counter = 0;

    public static void main(String[] args) {
        RaceConditionExample example = new RaceConditionExample();
        example.demo();
    }

    public void demo() {
        Thread t1 = new Thread(this::increment);
        Thread t2 = new Thread(this::increment);

        t1.start();
        t2.start();

        try {
            t1.join();
            t2.join();
        } catch (InterruptedException e) {
        }

        System.out.println("Counter: " + counter);
    }

    public void increment() {
        for (int i = 0; i < 1000; i++) {
            counter++;
        }
    }
}
```

### 3. 라이브락 (Livelock)
#### 정의
* 라이브락은 두 개 이상의 스레드가 서로의 진행을 방해하지 않으면서 계속해서 상태를 변경하지만, 실제로 작업을 진행하지 못하는 상태입니다. 데드락과 달리, 스레드들이 활동적이지만 작업이 완료되지 않습니다.

#### 예제
* 라이브락은 데드락과 유사한 방식으로 발생하지만, 자원을 해제하고 다시 요청하는 로직에서 발생할 수 있습니다.

### 4. 기아 상태 (Starvation)
#### 정의
* 기아 상태는 하나의 스레드가 자원을 얻지 못해 무한정 대기하는 상태를 말합니다. 이는 우선순위가 높은 스레드들이 계속해서 자원을 차지할 때 발생합니다.

#### 예제
* 기아 상태는 자원을 요청하는 우선순위가 낮은 스레드가 자원을 할당받지 못할 때 발생할 수 있습니다. 이는 주로 우선순위 기반 스케줄링에서 나타납니다.

## 동시성 문제 해결 방법
1. #### 락(Lock) 사용
* 상호 배제 락(Mutex), 읽기-쓰기 락(Read-Write Lock) 등을 사용하여 동시 접근을 제어합니다.

1. #### 데드락 예방
* 자원 할당 순서를 정하고, 순환 대기를 방지합니다.
* 타임아웃을 설정하여 일정 시간 대기 후 자원을 포기하도록 합니다.

1. #### 원자적 연산(Atomic Operation)
* 원자적 연산을 제공하는 라이브러리(예: Java의 AtomicInteger)를 사용하여 동시성 문제를 피합니다.

1. #### 락 프리(lock-free) 및 대기 프리(wait-free) 알고리즘
* 고성능 및 실시간 시스템에서 사용되며, 락 없이 동시성을 제어합니다.

1. #### 스레드 안전한 데이터 구조 사용
* Java의 Concurrent 패키지 등 스레드 안전한 데이터 구조를 사용합니다.

이러한 동시성 문제들은 복잡하고 디버깅이 어려울 수 있으므로, 올바른 동시성 제어 기법을 사용하여 안전하고 효율적인 멀티스레드 애플리케이션을 개발하는 것이 중요합니다.