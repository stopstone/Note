# Result로 예외처리하기

## 1. 서론

>  코틀린에서는 try ~ catch보다 **간결하고 선언적인 방식**으로 예외를 다룰 수 있도록 `Result<T>` 클래스를 제공합니다.   
> `runCatching`, `onSuccess`, `onFailure`와 함께 사용하면 **명령형 예외 처리보다 더 깔끔하게 로직을 구성**할 수 있어요.  
> 이 글에서는 `Result<T>`가 무엇인지, 기존 방식과 어떤 차이가 있는지, 내부 동작과 언제 활용하면 좋은지를 정리해볼게요.  

## 2. Result<T>란?

`Result<T>`는 성공과 실패를 명확하게 표현하기 위해 코틀린 표준 라이브러리에서 제공하는 **컨테이너 클래스**입니다.

```kotlin
val result: Result<Int> = runCatching { "123".toInt() }
```

### 특징

* 성공일 경우: 내부에 성공 결과 값을 담고 있음
* 실패일 경우: 내부에 예외(Throwable)를 담고 있음

```kotlin
val parsed = runCatching { "abc".toInt() }  // 실패

parsed.onSuccess {
    println("변환 성공: $it")
}.onFailure {
    println("변환 실패: ${it.message}")
}
```

## 3. try-catch와 뭐가 다를까?

### try-catch 방식

```kotlin
try {
    val result = "abc".toInt()
    println("성공: $result")
} catch (e: Exception) {
    println("실패: ${e.message}")
}
```

### Result<T> 방식

```kotlin
runCatching { "abc".toInt() }
    .onSuccess { println("성공: $it") }
    .onFailure { println("실패: ${it.message}") }
```

### 차이점 비교

| 항목     | try-catch      | Result<T>                     |
| ------ | -------------- | ----------------------------- |
| 문법 구조  | 명령형            | 함수형 체이닝                       |
| 코드 양   | 길어짐            | 간결함                           |
| 테스트    | 힘듦 (예외 비교 어려움) | `isSuccess`, `isFailure` 등 제공 |
| 비동기 구조 | 연계 어렵다         | coroutine과 궁합 좋음              |

## 4. 주요 API 정리

### runCatching

```kotlin
val result = runCatching { someDangerousCall() }
```

→ 내부적으로 try-catch를 감싸고, Result.success() 또는 Result.failure()를 리턴해요.

### onSuccess, onFailure

```kotlin
result.onSuccess { doSomething(it) }
      .onFailure { logError(it) }
```

### getOrNull(), getOrElse(), getOrDefault()

```kotlin
val value = result.getOrNull()           // 실패면 null
val value = result.getOrElse { -1 }      // 실패 시 기본값
val value = result.getOrDefault(0)       // 동일하게 기본값 리턴
```

## 5. 내부 코드 살펴보기

코틀린 표준 라이브러리에는 `Result<T>`가 `inline class`로 정의되어 있고,  
성공/실패 상태를 구분하는 플래그와 데이터를 포함하고 있습니다.

### 요약 구조

```kotlin
@SinceKotlin("1.3")
public inline class Result<out T>(val value: Any?) {
    val isSuccess: Boolean
    val isFailure: Boolean

    fun getOrThrow(): T
    fun exceptionOrNull(): Throwable?
    ...
}
```

### runCatching 내부 구현 (축약)

```kotlin
public inline fun <R> runCatching(block: () -> R): Result<R> =
    try {
        Result.success(block())
    } catch (e: Throwable) {
        Result.failure(e)
    }
```

즉, runCatching은 내부적으로 try-catch를 사용하지만, 결과를 객체로 래핑하여 다룰 수 있게 해줍니다.

## 6. 언제 쓰면 좋을까?

* **try-catch를 함수 밖으로 감추고 싶을 때**
* **체이닝 방식으로 로직을 깔끔하게 표현하고 싶을 때**
* **에러 발생 여부를 값으로 판단하고 싶을 때 (e.g. 테스트 코드)**
* **코루틴이나 Flow 연산 안에서 예외를 함수형으로 핸들링하고 싶을 때**

## 참고 자료

* [공식 문서: kotlin.Result](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-result/)
* [runCatching 공식 문서](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/run-catching.html)
