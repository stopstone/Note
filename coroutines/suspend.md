# suspend

> `suspend`는 코루틴에서 일시 중단이 가능한 함수를 만들기 위해 사용하는 키워드입니다.  
> 비동기 작업을 스레드를 점유하지 않고 효율적으로 처리하기 위해 필요하며, 이를 통해 코드를 동기식처럼 깔끔하게 구성할 수 있습니다.  

---

## 1. suspend란?

`suspend`는 코틀린에서 **코루틴 함수**임을 나타내는 키워드입니다.  
정확히 말하면, **일시 중단이 가능한 함수**를 정의할 때 사용하는 키워드입니다.  

```kotlin
suspend fun fetchData(): String {
    delay(1000)
    return "결과"
}
```

위 코드는 `fetchData()`가 **코루틴 안에서만 호출될 수 있다는 것**을 의미합니다.  
그리고 `delay()` 같은 suspend 함수들을 사용할 수 있게 됩니다.  

---

## 2. 왜 suspend가 필요할까요?

### 코루틴의 흐름 제어를 위해

코루틴은 `중단(suspend)`과 `재개(resume)`가 가능해야 합니다.
`suspend` 키워드는 이 흐름을 제어하기 위한 **특수한 함수 형태**를 정의해주는 역할을 합니다.

### 스레드 블로킹 없이 일시 정지하기 위해

`suspend` 함수는 일반 함수와 달리 **스레드를 점유하지 않고 일시 정지**할 수 있습니다.
즉, **비동기 작업을 동기처럼 자연스럽게 작성**할 수 있게 해줍니다.

예시:

```kotlin
val result = fetchData()  // 일시정지 가능, 블로킹 아님
println(result)           // 순차적 코드처럼 보임
```

---

## 3. suspend가 붙은 함수는 어떻게 호출할까요?

`suspend` 함수는 반드시 **코루틴 범위(CoroutineScope)** 안에서 호출되어야 합니다.  
즉, 일반 함수에서는 직접 호출할 수 없고, `launch`, `async`, `runBlocking` 같은 **코루틴 빌더** 내부에서만 호출할 수 있습니다.  

```kotlin
fun main() = runBlocking {
    val result = fetchData()
    println(result)
}
```

또한 중요한 규칙이 있습니다:

> **`suspend` 함수는 오직 다른 `suspend` 함수 안에서만 호출할 수 있습니다.**  

즉, 일반 함수에서 직접 `suspend` 함수를 호출하려 하면 컴파일 오류가 발생합니다.  

```kotlin
// 일반 함수에서는 suspend 함수 호출 불가
fun normalFunc() {
    fetchData()  // 오류 발생!
}

// suspend 함수 안에서는 호출 가능
suspend fun suspendedFunc() {
    fetchData()  // OK
}
```

이렇게 제한을 두는 이유는, `suspend` 함수는 **일시 정지 가능성**을 내포하고 있기 때문입니다.  
일반 함수에서 호출하면 일시 중단/재개 시점이 없기 때문에 안전하지 않다고 판단하는 것입니다.  

---

## 4. suspend와 블로킹 함수의 차이

| 구분     | suspend 함수        | 블로킹 함수               |
| ------ | ----------------- | -------------------- |
| 스레드 점유 | 점유하지 않음 (일시정지만 함) | 스레드 점유함              |
| 호출 방법  | 코루틴 내부에서만 호출 가능   | 어디서나 호출 가능           |
| 예시     | `delay(1000)`     | `Thread.sleep(1000)` |
| 사용 목적  | 비동기 흐름 제어         | 동기 코드 일시 정지          |

---

## 참고자료

* [공식 문서 – Kotlin Coroutines](https://kotlinlang.org/docs/coroutines-overview.html)
* [Coroutine Guide – Suspend functions](https://kotlinlang.org/docs/coroutines-basics.html#suspend-functions)
