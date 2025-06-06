# 블로킹(Blocking)과 논블로킹(Non-Blocking)

> 동시성 처리를 다룰 때 꼭 알아야 할 기본 개념인 블로킹과 논블로킹은, 특히 안드로이드 개발에서 코루틴과 연결되어 매우 중요합니다.  
> 이 글에서는 블로킹과 논블로킹의 개념, 차이점, 코루틴에서의 동작 방식까지 정리합니다.  

---

## 1. 블로킹(Blocking)과 논블로킹(Non-Blocking)의 차이

| 항목     | 블로킹(Blocking)                                 | 논블로킹(Non-Blocking)             |
| ------ | --------------------------------------------- | ------------------------------ |
| 정의     | 작업이 완료될 때까지 현재 스레드를 멈춤                        | 작업을 요청하고 바로 다음 작업 수행           |
| 동작 방식  | 결과가 나올 때까지 대기                                 | 즉시 반환, 결과는 나중에 처리              |
| 스레드 사용 | 하나의 스레드가 작업을 기다림                              | 하나의 스레드가 여러 작업 처리 가능           |
| 예시     | `Thread.sleep()`, `BufferedReader.readLine()` | `Coroutine`, `Callback`, `NIO` |
| 성능     | 스레드 많이 필요, 전환 비용 큼                            | 스레드 적게 사용, 효율적                 |
| 오류 처리  | try-catch 내부 처리                               | 콜백, Result, Promise 등으로 처리     |

---

## 2. 예시 코드 비교

### 블로킹 예시

```kotlin
fun readFileBlocking() {
    val file = File("test.txt")
    val content = file.readText() // 완료까지 스레드 멈춤
    println(content)
}
```

### 논블로킹 예시 (코루틴)

```kotlin
suspend fun readFileNonBlocking() = withContext(Dispatchers.IO) {
    val file = File("test.txt")
    val content = file.readText()
    println(content)
}
```

---

## 3. `suspend fun`이 논블로킹인 이유

* `suspend fun`은 컴파일 시 내부적으로 Continuation 객체를 파라미터로 받는 함수로 변환됨
* 함수 실행을 일시 중단(suspend)하고 재개(resume)할 수 있음
* 중단된 시점에서 스레드는 반환되고, 다른 작업이 가능함

### 예시

```kotlin
suspend fun networkCall() {
    delay(1000) // 스레드 점유 안 함
    println("응답 받음")
}
```

* `delay()`는 `Thread.sleep()`과 달리 스레드를 점유하지 않음
* 중단 → 다른 일 처리 → 재개 로 동작

---

## 4. `runBlocking`이 블로킹인 이유와 용도

* `runBlocking`은 호출한 스레드를 점유함 → 블로킹 동작
* 내부에서 코루틴을 실행하고, 결과가 나올 때까지 기다림

### 테스트에서의 활용

```kotlin
@Test
fun `test fetch data`() = runBlocking {
    viewModel.fetchData()
    assertEquals("완료", viewModel.state.value)
}
```

* 테스트에서는 모든 작업을 동기적으로 처리해야 결과를 정확히 검증 가능
* 따라서 `runBlocking`으로 전체 흐름을 감싸는 것이 일반적

---

## 5. 정리

| 구분            | 설명                           |
| ------------- | ---------------------------- |
| `suspend fun` | 논블로킹. 중단 가능한 함수. 상태 머신처럼 동작  |
| `delay()`     | 논블로킹. 스레드 점유하지 않고 중단만 함      |
| `runBlocking` | 블로킹. 호출한 스레드를 점유함            |
| 코루틴의 핵심       | 비동기 코드를 동기처럼 작성할 수 있도록 돕는 도구 |

---

## 참고 자료

* [Kotlin Coroutine 공식 문서](https://kotlinlang.org/docs/coroutines-overview.html)
* [Android 공식 Coroutine 가이드](https://developer.android.com/kotlin/coroutines)
