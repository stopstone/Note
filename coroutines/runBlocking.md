# runBlocking

## 1. runBlocking이란?

`runBlocking`은 코루틴을 **일반 함수(main, 테스트 등)에서 동기적으로 호출하기 위해 사용하는 코루틴 빌더**입니다.  
이름 그대로 코루틴을 시작하면서 현재 스레드를 **블로킹(blocking)** 합니다. 일반적인 코루틴 빌더인 `launch`, `async`와 다르게,  
`runBlocking`은 호출한 스레드가 코루틴이 종료될 때까지 기다립니다.

```kotlin
fun main() {
    println("시작")
    runBlocking {
        delay(1000L)
        println("코루틴 내부")
    }
    println("끝")
}
```

이 예제는 "시작" → 1초 후 "코루틴 내부" → "끝" 순으로 출력됩니다. `runBlocking`이 코루틴이 끝날 때까지 블로킹하기 때문입니다.

---

## 2. 언제 사용하는가?

* **main 함수에서 코루틴 호출이 필요할 때**: Kotlin 애플리케이션의 진입점에서 코루틴을 사용할 때 주로 사용합니다.  
* **JUnit 테스트에서 코루틴 테스트할 때**: 실제 서비스 코드에서는 `viewModelScope` 등을 사용하더라도, 테스트에서는 `runBlocking`을 통해 코루틴을 명시적으로 실행하고 결과를 확인할 수 있습니다.

```kotlin
@Test
fun someTest() = runBlocking {
    val result = suspendFunction()
    assertEquals(expected, result)
}
```

---

## 3. runBlocking의 특징과 주의사항

| 특징                    | 설명                                             |
| --------------------- | ---------------------------------------------- |
| 스레드 블로킹               | 내부 코루틴이 끝날 때까지 호출 스레드를 블로킹함                    |
| 디스패처 기본값              | `Dispatchers.Default`가 아닌 현재 스레드(보통 main)를 사용함 |
| 주로 테스트나 main 함수에서만 사용 | 실제 앱 코드에서는 `runBlocking`을 피해야 함                |

---
