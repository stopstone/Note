# launch와 async의 차이

> 코루틴을 사용할 때 가장 기본적인 빌더 중 두 가지인 `launch`와 `async`는 기능이 비슷해 보이지만, 분명한 차이를 가지고 있습니다.
> 이 글에서는 각각의 특징과 사용 목적, 차이점에 대해 정리해보겠습니다.

---

## 1. launch란?

`launch`는 결과값을 반환하지 않는 코루틴을 시작할 때 사용하는 빌더입니다.  
일반적으로 로그 출력, UI 상태 변경, 데이터 삭제 등 **결과가 필요 없는 작업**에 사용합니다.  

### 특징

* 반환값: `Job`
* 완료 대기: `job.join()`
* 예외 처리: `CoroutineExceptionHandler` 사용 권장
* 대표 용도: 로그, UI 변경, 알림 등

```kotlin
val job = launch {
    delay(1000)
    println("launch: 작업 완료")
}
job.join()
```

---

## 2. async란?

`async`는 **값을 반환하는 비동기 작업**을 시작할 때 사용합니다.  
결과값을 담고 있는 `Deferred<T>` 객체를 반환하며, 이후 `await()`를 통해 값을 얻을 수 있습니다.  

### 특징

* 반환값: `Deferred<T>`
* 완료 대기 및 결과 획득: `deferred.await()`
* 예외 처리: `try-catch`로 await 시 처리
* 대표 용도: 계산, API 요청, 파일 읽기 등

```kotlin
val deferred = async {
    delay(1000)
    "결과 값"
}
val result = deferred.await()
println("async 결과: $result")
```

---

## 3. launch vs async 요약 비교

| 항목       | `launch`                       | `async`                  |
| -------- | ------------------------------ | ------------------------ |
| 반환 타입    | `Job`                          | `Deferred<T>`            |
| 결과 반환 여부 | 없음                             | 있음 (`await()`로 결과 획득 가능) |
| 완료 대기 방식 | `join()`                       | `await()`                |
| 예외 처리 방법 | `CoroutineExceptionHandler` 사용 | `try-catch`로 await 시 처리  |
| 사용 목적    | 단순 실행, UI 처리, 로그 등             | 계산, 네트워크 요청 등 결과 필요 작업   |

---

## 4. 어떤 상황에서 어떤 걸 써야 할까?

* UI 변경, 로그 출력, 알림 등 결과를 필요로 하지 않는 작업: `launch`  
* API 요청, 계산, DB 쿼리 등 결과값이 필요한 작업: `async`  
  
---

## 참고자료

* [Kotlin 공식 문서 - Coroutine builders](https://kotlinlang.org/docs/coroutines-basics.html#coroutine-builders)

---
