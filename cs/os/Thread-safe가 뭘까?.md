# Thread-safe라는 말이 뭘까?

> 멀티스레드 환경에서 하나의 데이터에 여러 스레드가 동시에 접근해도  
> **문제 없이 안전하게 동작하는 상태**를 Thread-safe하다고 말합니다.   
> 이 글에서는 Thread-safe의 정의와 필요성, 그리고 안전하게 구현하는 방법까지 단계적으로 설명합니다.
> 
## 1. 왜 Thread-safe가 필요할까?

멀티스레드 환경에서는 여러 작업이 동시에 실행되며, 이들이 **공유 자원**에 동시에 접근할 경우 충돌이 발생할 수 있습니다.  
이런 충돌로 인해 **예상하지 못한 동작(Race Condition)** 이나 **데이터 손상**이 발생할 수 있어요.  

예를 들어, 다음과 같은 코드가 있을 때:

```kotlin
var count = 0

fun increase() {
    count++
}
```

`count++`는 단순한 연산처럼 보이지만, 실제로는 아래처럼 3단계로 분리됩니다.  

1. `count` 값을 읽고  
2. 1을 더한 뒤  
3. 다시 `count`에 저장  

여러 스레드가 동시에 이 작업을 하면 **값이 덮어써지거나 누락**되어 결국 원하는 값이 나오지 않게 됩니다.  

---

## 2. Thread-safe한 방식으로 코드를 작성하는 방법

### 1) `@Synchronized` 사용

```kotlin
@Synchronized
fun increase() {
    count++
}
```

`synchronized` 키워드는 한 번에 하나의 스레드만 해당 메서드에 진입할 수 있도록 합니다.  
간단한 방법이지만, **락 경합(lock contention)** 이 발생하면 성능 저하가 있을 수 있습니다.

---

### 2) 코루틴 환경에서 `Mutex` 사용

```kotlin
val mutex = Mutex()
var count = 0

suspend fun increase() {
    mutex.withLock {
        count++
    }
}
```

코루틴에서는 `@Synchronized` 대신 `Mutex`를 사용하여 동시성을 제어합니다.   
`withLock` 블록 안의 코드는 한 번에 하나의 코루틴만 실행할 수 있습니다.  

---

## 3. 어떤 상황에서 Thread-safe를 고려해야 할까?

* 싱글톤 객체, 전역 변수 등 **공유 자원**을 여러 스레드가 접근할 경우
* **UI 상태 관리**를 ViewModel 등에서 병렬로 수행하는 경우
* **백그라운드 작업과 메인 스레드가 데이터를 주고받는 경우**

안드로이드에서는 LiveData, Flow, SharedPreferences 등에서 이런 문제가 자주 발생하므로,  
내부적으로도 Thread-safe 처리가 되어 있거나 개발자가 직접 관리해줘야 합니다.

---

## 참고자료

* [Kotlin Concurrency](https://kotlinlang.org/docs/concurrency-overview.html)
* Java Concurrency in Practice
* Effective Kotlin
