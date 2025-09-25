# Exception과 Error, Throwable의 차이

> 안드로이드에서 예외 처리를 할 때 `Exception`, `Error`, `Throwable`이라는 용어들을 볼 수 있습니다.  
> 같은 개념이라고 생각할 수 있지만 차이가 조금 있습습니다.
> 이번 글에서는 각각의 차이 정리해보겠습니다.

---

## 1. 기본 개념부터 이해하기

### Throwable이란?
**`Throwable`은 모든 예외와 오류의 최상위 클래스**입니다.  
Kotlin/Java에서 `throw` 키워드로 던질 수 있는 모든 객체는 `Throwable`을 상속받아야 합니다.

```kotlin
// Throwable을 상속받은 커스텀 예외
class CustomException(message: String) : Throwable(message)

fun throwSomething() {
    throw CustomException("커스텀 예외 발생!")
}
```

### Exception이란?
**`Exception`은 프로그램에서 복구 가능한 문제를 나타내는 클래스**입니다.  
개발자가 예상할 수 있고 처리할 수 있는 상황들입니다.

```kotlin
// 예외 상황 예시
fun divideNumbers(a: Int, b: Int): Int {
    if (b == 0) {
        throw ArithmeticException("0으로 나눌 수 없습니다") // Exception
    }
    return a / b
}
```

### Error가 무엇인가요?
**`Error`는 시스템 레벨의 심각한 문제를 나타내는 클래스**입니다.  
일반적으로 개발자가 복구할 수 없는 상황들입니다.

```kotlin
// Error 예시 (실제로는 이런 코드를 작성하지 않음)
fun causeOutOfMemoryError() {
    // OutOfMemoryError는 Error의 하위 클래스
    val hugeArray = Array(Int.MAX_VALUE) { it }
}
```

---

## 2. 계층 구조로 이해하기

```
Throwable
├── Error (시스템 레벨 오류)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── NoClassDefFoundError
└── Exception (복구 가능한 예외)
    ├── RuntimeException (Unchecked Exception)
    │   ├── NullPointerException
    │   ├── IllegalArgumentException
    │   └── IndexOutOfBoundsException
    └── Checked Exception
        ├── IOException
        ├── SQLException
        └── ParseException
```

### 핵심 차이점 정리

| 구분 | Exception | Error |
|:---|:---|:---|
| **복구 가능성** | 복구 가능 | 복구 불가능 |
| **발생 원인** | 프로그래밍 로직 오류 | 시스템 리소스 부족 |
| **처리 방법** | try-catch로 처리 | 일반적으로 처리하지 않음 |
| **예시** | NullPointerException, IOException | OutOfMemoryError, StackOverflowError |

---

## 3. Checked vs Unchecked Exception

### Checked Exception (검사 예외)
컴파일 타임에 반드시 처리해야 하는 예외입니다.

```kotlin
// Kotlin에서는 Checked Exception이 없지만, Java와의 호환성을 위해 설명
// Java 예시
public void readFile() throws IOException {
    // IOException은 Checked Exception
    // 반드시 try-catch 또는 throws로 처리해야 함
}
```

### Unchecked Exception (비검사 예외)
컴파일 타임에 처리하지 않아도 되는 예외입니다. 주로 `RuntimeException`의 하위 클래스들입니다.

```kotlin
fun demonstrateUncheckedException() {
    // NullPointerException - Unchecked Exception
    val list: List<String>? = null
    val size = list!!.size // 런타임에 NullPointerException 발생
    
    // IllegalArgumentException - Unchecked Exception
    fun validateAge(age: Int) {
        if (age < 0) {
            throw IllegalArgumentException("나이는 음수일 수 없습니다")
        }
    }
}
```

---
