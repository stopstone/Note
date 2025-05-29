# Kotlin Enum Class

> Kotlin에서 `enum class`는 상수를 그룹화할 때 사용되며, Java의 enum과 달리 더 많은 기능을 제공합니다.  
> 본 글에서는 enum 클래스의 개념, Kotlin과 Java 간 차이점을 정리합니다.  

## 1. Enum 클래스란?

`Enum`(Enumeration)은 **열거형 타입**으로, 제한된 상수 집합을 정의할 때 사용합니다.

Java의 `enum`, Kotlin의 `enum class`는 모두 타입 안전성을 보장하며 상수값을 그룹화하는 데 사용됩니다.

```kotlin
enum class Direction {
    NORTH, SOUTH, EAST, WEST
}
```

## 2. Kotlin에서의 특징

### 2-1. 프로퍼티와 메서드 정의 가능

```kotlin
enum class HttpStatusCode(val code: Int) {
    OK(200),
    NOT_FOUND(404),
    INTERNAL_ERROR(500);

    fun isServerError(): Boolean = code >= 500
}
```

* enum 인스턴스마다 생성자 파라미터를 넘길 수 있음
* 메서드도 정의 가능

### 2-2. when 구문에서 모든 경우 처리 요구

```kotlin
fun handleDirection(direction: Direction) {
    when (direction) {
        Direction.NORTH -> println("북")
        Direction.SOUTH -> println("남")
        Direction.EAST  -> println("동")
        Direction.WEST  -> println("서")
    }
}
```

> enum을 사용할 경우 `when`에서 exhaustive 체크(모든 분기 처리)를 컴파일러가 보장해줍니다.

## 3. Java의 enum과의 차이

| 항목             | Kotlin enum class      | Java enum           |
| -------------- | ---------------------- | ------------------- |
| 선언 방식          | `enum class` 키워드 사용    | `enum` 키워드 사용       |
| 상속             | `Enum<T>`을 자동 상속       | 동일                  |
| sealed-like 동작 | `when`에서 모든 case 처리 강제 | switch-case에서 강제 아님 |
| 기본 생성자         | 생성자 정의 가능              | 생성자 정의 가능           |

---

## 참고 자료

* [Kotlin 공식 문서 - Enum Classes](https://kotlinlang.org/docs/enum-classes.html)
* [Java 공식 문서 - Enum Types](https://docs.oracle.com/javase/tutorial/java/javaOO/enum.html)
