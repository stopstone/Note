# Deadlock(교착 상태)

> Deadlock(교착 상태)은 여러 프로세스나 스레드가 서로 자원을 점유한 채,  
> 상대방이 가진 자원을 기다리면서 아무도 작업을 못 하고 멈춰버리는 상황입니다.    
> 한 번 이렇게 얽히면 시스템이 멈춘 것처럼 반응이 없어지고, 자원도 회수하지 못해서 문제가 발생하게 됩니다.  

---

## 1. Deadlock이란?

Deadlock은 말 그대로 서로가 서로를 기다리면서 아무 것도 못 하는 상태입니다.  
예를 들어 스레드 A는 락 1을 가지고 락 2를 원하고, 스레드 B는 락 2를 가지고 락 1을 원하게 되면,  
둘 다 서로를 기다리기만 하면서 아무 일도 못 하게 됩니다.  

### 예시

간단한 예를 들어보겠습니다:

* 스레드 A: lock1 획득 → lock2 필요
* 스레드 B: lock2 획득 → lock1 필요

이렇게 되면 서로 필요한 락을 상대가 들고 있기 때문에, 영원히 기다리기만 하게 됩니다. 이게 바로 Deadlock이라고 합니다.

---

## 2. Deadlock이 발생하는 4가지 조건

Deadlock이 발생하려면 이 네 가지 조건이 동시에 만족돼야 합니다:

| 조건                       | 설명                                     |
| ------------------------ | -------------------------------------- |
| 상호 배제 (Mutual Exclusion) | 자원은 한 번에 하나의 프로세스만 사용할 수 있어요.          |
| 점유 대기 (Hold and Wait)    | 이미 자원을 점유한 상태에서, 다른 자원을 더 요구하면서 기다립니다. |
| 비선점 (No Preemption)      | 점유한 자원을 다른 스레드가 강제로 뺏을 수 없습니다.         |
| 순환 대기 (Circular Wait)    | 자원을 서로 물고 물면서 기다리는 순환 형태가 생깁니다.        |

이 네 가지가 모두 만족되면 Deadlock이 발생할 수 있어요.

---

## 3. Deadlock은 언제 발생할까?

### 상황 예시

멀티스레드 환경에서 락을 아래처럼 사용하면:

```kotlin
synchronized(lock1) {
    synchronized(lock2) {
        // 작업
    }
}

// 다른 스레드에서
synchronized(lock2) {
    synchronized(lock1) {
        // 작업
    }
}
```

이처럼 서로 반대 순서로 락을 걸게 되면, 두 스레드가 서로가 가진 락을 기다리면서 아무 일도 못 하게 됩니다.  
이게 바로 Deadlock 상황입니다.  

### Android 실무 예시

* Room에서 트랜잭션 처리 중 LiveData로 UI까지 엮어버리면 MainThread와 IO Thread가 서로 기다리는 상황이 생길 수 있어요.
* `withContext(Dispatchers.IO)` 안에서 UI 자원을 접근하려고 하면, 교착 상태에 빠질 수도 있습니다.
* ViewModel끼리 DataStore 접근할 때, 동시에 락을 걸면 서로 해제하지 않고 기다리는 상황이 발생할 수 있어요.

---

## 4. Deadlock 방지 및 회피 전략

Deadlock을 막는 방법은 다양합니다. 대표적인 방법 몇 가지를 정리해보면 다음과 같습니다:

| 방식                | 설명                                                  |
| ----------------- | --------------------------------------------------- |
| 자원 할당 순서 고정       | 항상 같은 순서로 자원을 요청하면, 순환 대기를 피할 수 있습니다.               |
| 타임아웃 사용           | 일정 시간 동안 락을 획득하지 못하면 포기하도록 설정합니다.                   |
| Lock-free 알고리즘 사용 | 락 대신 CAS 같은 방식으로 동기화 처리하여 락 자체를 사용하지 않습니다.          |
| 자원 사전 점유 방식       | 필요한 자원을 한 번에 모두 점유한 후 작업을 시작해, 중간에 기다리는 일이 없도록 합니다. |

---

## 참고자료

* [Operating System - Deadlock (GeeksForGeeks)](https://www.geeksforgeeks.org/deadlock-in-operating-system/)
* [Android 공식 문서 - Kotlin Concurrency Best Practices](https://developer.android.com/kotlin/coroutines/coroutines-best-practices)
* \[Effective Java - Item 50: Avoid Deadlocks by Lock Ordering]
