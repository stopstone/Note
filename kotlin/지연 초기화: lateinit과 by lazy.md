# Kotlin에서의 지연 초기화: `lateinit`과 `by lazy`

## 1. 지연 초기화란?

**지연 초기화(lazy initialization)** 란, 변수를 선언할 때 바로 초기화하지 않고 실제로 사용되는 시점까지 초기화를 미루는 것을 의미합니다.  
이는 주로 리소스 낭비를 줄이고 초기화 비용이 큰 객체의 생성 시점을 제어하기 위해 사용됩니다.  
예를 들어, 다음과 같이 로그 출력을 위한 문자열을 실제 로그 호출 시점에만 초기화하는 것도 지연 초기화의 한 예시입니다.

```kotlin
val logMessage: String by lazy { "로그를 출력합니다." }
```

Kotlin에서는 이 기능을 위해 `lateinit`과 `by lazy`라는 두 가지 방법을 제공합니다.

---

## 2. `lateinit`과 `by lazy`의 차이점

| 항목               | `lateinit`                              | `by lazy`                                     |
|------------------|-----------------------------------------|-----------------------------------------------|
| 선언 키워드        | `var` (가변 변수)                         | `val` (불변 변수)                               |
| 초기화 시점        | 명시적으로 코드 내에서 초기화할 때          | 최초 접근 시 자동으로 초기화                    |
| 재할당 가능 여부   | 가능                                     | 불가능                                        |
| 사용 가능한 타입   | 참조형 타입만 사용 가능                    | 모든 타입 사용 가능                             |
| 스레드 안전성      | 보장되지 않음                              | 기본적으로 스레드 안전 (SYNCHRONIZED)           |
| 초기화 여부 확인   | `this::변수명.isInitialized` 사용           | `Lazy` 객체의 `isInitialized()` 사용 가능       |
| 널 허용 여부       | 불가능 (`null`로 초기화할 수 없음)          | 가능 (`nullable` 타입 선언 가능)                |
| 예외 발생 시점     | 초기화 전에 접근하면 런타임 예외 발생        | 초기화 블록 내부에서 예외가 발생할 수 있음      |

---

## 3. 사용 예제

### `lateinit` 예제

```kotlin
class UserProfile {
    lateinit var nickname: String

    fun initializeNickname() {
        nickname = "신짱구"
    }

    fun printNickname() {
        if (this::nickname.isInitialized) {
            println("닉네임: $nickname")
        } else {
            println("닉네임이 아직 초기화되지 않았습니다.")
        }
    }
}
```

해당 예제에서는 `nickname` 변수를 `lateinit`으로 선언하여 나중에 초기화합니다.  
초기화 여부를 확인하기 위해 `this::nickname.isInitialized`를 사용하며, 초기화되지 않은 상태에서 접근할 경우 예외가 발생할 수 있으므로 반드시 확인 후 사용하는 것이 안전합니다.

### `by lazy` 예제

```kotlin
class Config {
    val apiKey: String by lazy {
        println("API 키를 초기화합니다.")
        "SECRET_API_KEY"
    }
}
```

`apiKey`는 실제로 사용되기 전까지 초기화되지 않으며, 최초 접근 시 초기화 블록이 실행됩니다.  
단, 초기화 블록 안에서 예외가 발생할 수 있기 때문에 주의하여 작성해야 합니다.

---

## 4. 언제 어떤 것을 사용해야 될까?

### `lateinit`을 사용할 때

- 초기화 시점을 명확하게 제어해야 할 때
- 의존성 주입(Dependency Injection)을 사용할 때
- 안드로이드에서 View나 ViewBinding 객체를 늦게 초기화할 때

### `by lazy`를 사용할 때

- 초기화 비용이 큰 객체의 생성을 지연하고 싶을 때
- 싱글톤 객체를 생성할 때
- 스레드 안전한 초기화가 필요할 때

---

## 참고자료

[Kotlin 공식 문서](https://kotlinlang.org/docs/properties.html#late-initialized-properties-and-variables)

