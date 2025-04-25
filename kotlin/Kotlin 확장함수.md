# Kotlin 확장 함수

> Kotlin 확장 함수는 기존 클래스의 소스를 수정하지 않고도 마치 그 클래스에 메서드를 추가한 것처럼 사용할 수 있는 문법입니다.  
> 자바에 비해 확장성이 뛰어나고, Android 개발에서도 View나 Context를 확장하는 데 자주 활용됩니다.  
> 이 문서에서는 Kotlin 확장 함수의 개념과 활용에 대해 알아보겠습니다  

---
## 1. 확장함수란?

Kotlin의 확장 함수(Extension Function)는 클래스 외부에서 정의되지만, 해당 클래스의 멤버 함수처럼 사용할 수 있는 함수입니다.  
예를 들어 String 클래스에 공백을 제거하는 함수를 추가하고 싶을 때 다음과 같이 작성할 수 있습니다.

```kotlin
fun String.removeSpaces(): String {
    return this.replace(" ", "")
}

val text = "hello world"
println(text.removeSpaces()) // 출력: helloworld
```

---

## 2. Java와의 차이점

| 항목               | Kotlin 확장 함수                      | Java 방식                         |
|--------------------|----------------------------------------|----------------------------------|
| 정의 위치          | 클래스 외부에서 정의 가능              | 반드시 클래스 내부에서 정의해야 함 |
| 외부 라이브러리 확장 | 가능 (`String`, `List` 등)             | 불가능 (상속이나 래퍼 클래스 필요) |
| 가독성              | 높음 (마치 멤버 함수처럼 사용 가능)     | 낮음 (도우미 유틸 클래스 필요)     |
| 재사용성           | 높음                                   | 비교적 낮음                      |

자바에서는 비슷한 기능을 하려면 static 메서드를 유틸 클래스에 정의해야 합니다.

```java
public class StringUtils {
    public static String removeSpaces(String input) {
        return input.replace(" ", "");
    }
}

String result = StringUtils.removeSpaces("hello world");
```

---

## 3. Kotlin 확장 함수를 사용하는 이유

- **가독성 향상**: `객체.함수()` 형태로 표현되어 직관적입니다.
- **기존 클래스 확장**: 상속 없이 새로운 기능을 추가할 수 있습니다.
- **모듈화**: 기능별로 코드를 분리해 유지보수가 용이합니다.
- **DSL 구현에 적합**: Kotlin DSL의 핵심 요소로 활용됩니다.

---

## 4. 확장함수 장점

1. 클래스 수정 없이 기능 확장 가능
2. 외부 라이브러리나 SDK 클래스도 확장 가능
3. 중복 코드 제거
4. 가독성 높은 API 구성 가능

---

## 5. 사용자 정의 클래스에서는 언제 사용할까요?

사용자 정의 클래스에 확장 함수를 사용하는 대표적인 경우는 다음과 같습니다.

- **기능을 분리하여 작성하고 싶을 때**: 핵심 로직과 부가 기능을 나눠 작성할 수 있습니다.
- **해당 클래스의 인터페이스를 바꾸지 않고 기능을 추가하고 싶을 때**
- **코드 재사용성을 높이고 싶을 때**: 여러 곳에서 같은 기능을 재사용할 수 있습니다.

예시:

```kotlin
data class User(val name: String, val age: Int)

fun User.isAdult(): Boolean {
    return this.age >= 18
}

val user = User("홍길동", 21)
println(user.isAdult()) // 출력: true
```

위처럼 `User` 클래스에는 전혀 손대지 않고, 성인 여부를 판단하는 함수를 추가할 수 있습니다.

---

## 6. 실전 예시: List에서 null 제거

```kotlin
fun <T> List<T?>.removeNulls(): List<T> {
    return this.filterNotNull()
}

val original = listOf(1, null, 2, null, 3)
val cleaned = original.removeNulls()

println(cleaned) // 출력: [1, 2, 3]
```

---

## 7. 주의사항 및 한계점

- 실제 멤버 함수를 오버라이드할 수 없습니다. (접근도 불가능)
- 다형성이 적용되지 않기 때문에 정적 디스패치(static dispatch)로 동작합니다.
- 확장 프로퍼티는 backing field를 가질 수 없습니다.

```kotlin
val String.firstChar: Char
    get() = this[0]
```

---

