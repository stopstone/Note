# Kotlin의 Nullable

> Kotlin에서는 null 안정성을 언어 차원에서 지원합니다.  
> Java에서 흔하게 발생하는 NullPointerException(NPE)을 방지하기 위한 설계로,  
> Kotlin의 nullable 타입 시스템은 코드 안정성과 명확성을 높이는 데 중요한 역할을 합니다.  
> 이번 글에서는 nullable 타입이 필요한 이유, 사용하는 문법, 그리고 실전에서의 활용 방식까지 정리합니다.  

## 1. 왜 Nullable인가?

Java에서는 어떤 변수든 null이 될 수 있기 때문에 런타임 오류인 NullPointerException이 빈번하게 발생합니다.  
Kotlin은 이를 방지하기 위해 **nullable과 non-nullable 타입을 명확하게 구분**합니다.

```kotlin
val name: String = "신짱구"       // null이 될 수 없음
val nickname: String? = null     // null을 허용함
```

null을 허용하려면 타입 뒤에 `?`를 명시해야 하며, 그렇지 않으면 null 대입 시 컴파일 오류가 발생합니다.

## 2. Nullable 타입 활용 문법

### 2-1. Safe Call (`?.`)

객체가 null이 아닐 때만 접근하며, null이면 전체 식의 결과가 null이 됩니다.

```kotlin
val length: Int? = nickname?.length
```

### 2-2. Elvis Operator (`?:`)

null일 경우 대체 값을 지정할 수 있습니다.

```kotlin
val displayName = nickname ?: "Guest"
```

### 2-3. Not-null Assertion (`!!`)

개발자가 "절대 null이 아님"을 단언할 때 사용하지만, null이면 런타임에서 예외가 발생합니다. 이 연산자는 실무에서 가장 많은 NPE의 원인이 되므로 **가급적 사용을 지양**하는 것이 좋습니다.

```kotlin
val length = nickname!!.length // nickname이 null이면 예외 발생
```

```kotlin
val safeLength = nickname?.length // null이면 null 반환 (예외 없음)
```

## 참고 자료

* [Kotlin 공식 문서](https://kotlinlang.org/docs/null-safety.html)
* Kotlin 인 액션, 6장: 널 가능성과 플랫폼 타입
