# Kotlin vararg

> Kotlin의 `vararg` 키워드는 **가변 인자(Variable Arguments)**를 받는 함수를 정의할 때 사용하는 키워드입니다.  
> Java의 `...` (Varargs)와 유사하지만, 더 강력하고 안전한 기능을 제공합니다.

---

## 1. Java와 Kotlin의 차이점

### Java의 가변 인자
```java
// Java - ... 사용
public void printMessages(String... messages) {
    for (String message : messages) {
        System.out.println(message);
    }
}

// 호출
printMessages("Hello", "World", "Java");
```

### Kotlin의 가변 인자
```kotlin
// Kotlin - vararg 사용
fun printMessages(vararg messages: String) {
    for (message in messages) {
        println(message)
    }
}

// 호출
printMessages("Hello", "World", "Kotlin")
```

### 배열 전달 시 차이점
```java
// Java - 배열을 그대로 전달 가능
String[] array = {"A", "B", "C"};
printMessages(array);
```

```kotlin
// Kotlin - 스프레드 연산자(*) 필요
val array = arrayOf("A", "B", "C")
printMessages(*array)  // * 연산자로 배열 전개
```

---

## 2. vararg 키워드란?

### 정의와 특징
```kotlin
// 기본 사용법
fun sum(vararg numbers: Int): Int {
    var total = 0
    for (number in numbers) {
        total += number
    }
    return total
}

// 다양한 호출 방법
val result1 = sum(1, 2, 3, 4, 5)        // 15
val result2 = sum()                      // 0 (인자 없음)
val result3 = sum(10)                    // 10 (단일 인자)
```

### vararg의 타입
```kotlin
fun printTypes(vararg items: Any) {
    // vararg 매개변수는 Array<T> 타입
    println("타입: ${items::class.simpleName}")  // Array
    println("크기: ${items.size}")
    
    items.forEachIndexed { index, item ->
        println("[$index] $item (${item::class.simpleName})")
    }
}

printTypes("문자열", 123, true, 45.6)
```

---

## 3. 스프레드 연산자 (Spread Operator)

### 배열을 vararg로 전달
```kotlin
fun concatenate(vararg strings: String): String {
    return strings.joinToString(" ")
}

// 배열 전개
val words = arrayOf("Kotlin", "is", "awesome")
val result = concatenate(*words)  // "Kotlin is awesome"

// 일부는 직접, 일부는 배열로
val moreSentence = concatenate("I think", *words, "!")
// "I think Kotlin is awesome !"
```

### 리스트를 vararg로 전달
```kotlin
val list = listOf("Apple", "Banana", "Cherry")

// 리스트 -> 배열 -> vararg
printMessages(*list.toTypedArray())

// 또는 직접 전개
fun printFruits(vararg fruits: String) {
    fruits.forEach { println("과일: $it") }
}

printFruits(*list.toTypedArray())
```

---

## 4. vararg 활용해보기

### 1. Android에서 문자열 포맷팅
```kotlin
object StringFormatter {
    fun format(template: String, vararg args: Any): String {
        return String.format(template, *args)
    }
    
    fun join(separator: String = ", ", vararg items: String): String {
        return items.joinToString(separator)
    }
}

// 사용 예시
val message = StringFormatter.format(
    "사용자 %s님, %d개의 메시지가 있습니다.", 
    "김철수", 5
)

val tags = StringFormatter.join(" | ", "Android", "Kotlin", "Mobile")
```

### 2. 데이터베이스 쿼리 헬퍼
```kotlin
class DatabaseHelper {
    fun select(vararg columns: String): String {
        val columnList = if (columns.isEmpty()) "*" else columns.joinToString(", ")
        return "SELECT $columnList FROM"
    }
    
    fun insert(table: String, vararg pairs: Pair<String, Any>): String {
        val columns = pairs.map { it.first }.joinToString(", ")
        val values = pairs.map { "?" }.joinToString(", ")
        return "INSERT INTO $table ($columns) VALUES ($values)"
    }
}

// 사용 예시
val db = DatabaseHelper()
val selectQuery = db.select("name", "age", "email") + " users"
val insertQuery = db.insert("users", 
    "name" to "김철수", 
    "age" to 30, 
    "email" to "kim@example.com"
)
```

---

## 5. vararg 사용 시 주의사항

### 1. 함수당 하나의 vararg만 가능
```kotlin
// 컴파일 오류 - vararg는 하나만 가능
fun badFunction(vararg strings: String, vararg numbers: Int) { }

// 올바른 방법 - 다른 매개변수와 조합
fun goodFunction(prefix: String, vararg items: String, suffix: String) {
    // vararg 뒤에 다른 매개변수가 있으면 named argument 사용 필요
}

// 호출 시
goodFunction("시작", "A", "B", "C", suffix = "끝")
```

### 2. 성능 고려사항
```kotlin
// 매번 배열 생성으로 인한 성능 저하
fun inefficientLogging(vararg messages: String) {
    if (isDebugEnabled) {  // 디버그 모드가 아니어도 배열 생성됨
        println(messages.joinToString(" "))
    }
}

// 조건부 실행으로 최적화
fun efficientLogging(messageProvider: () -> Array<String>) {
    if (isDebugEnabled) {
        val messages = messageProvider()
        println(messages.joinToString(" "))
    }
}

// 또는 inline 함수 사용
inline fun debugLog(messageProvider: () -> String) {
    if (isDebugEnabled) {
        println(messageProvider())
    }
}
```

### 3. 타입 안전성
```kotlin
// 제네릭과 함께 사용할 때 주의
fun <T> processItems(vararg items: T): List<T> {
    return items.toList()  // Array<T> -> List<T>
}

// 사용 시 타입 추론
val strings = processItems("A", "B", "C")        // List<String>
val numbers = processItems(1, 2, 3)             // List<Int>
val mixed = processItems("A", 1, true)          // List<Any>
```

---

## 6. 언제 사용하면 좋을까?

### vararg 사용이 좋은 경우

**1. 유사한 타입의 여러 인자를 받는 함수**
```kotlin
fun createList(vararg items: String): List<String> = items.toList()
fun findMaximum(vararg numbers: Int): Int = numbers.maxOrNull() ?: 0
```

**2. 빌더 패턴이나 DSL 구현**
```kotlin
fun html(vararg elements: String): String {
    return "<html>${elements.joinToString("")}</html>"
}
```

**3. 로깅이나 디버깅 함수**
```kotlin
fun log(level: String, vararg messages: Any) {
    println("[$level] ${messages.joinToString(" ")}")
}
```

### vararg 사용을 피해야 하는 경우

**1. 배열이나 컬렉션을 주로 받는 경우**
```kotlin
// vararg 보다는 컬렉션 매개변수가 더 적합
fun processUserIds(vararg userIds: Long) { }

// 더 나은 방법
fun processUserIds(userIds: List<Long>) { }
fun processUserIds(userIds: Collection<Long>) { }
```

**2. 성능이 중요한 상황**
```kotlin
// 빈번한 호출에서 배열 생성 오버헤드
fun hotPathFunction(vararg data: Int) { }

// 배열을 직접 받아서 처리
fun hotPathFunction(data: IntArray) { }
```

## 참고 자료

- [Kotlin 공식 문서 - Functions](https://kotlinlang.org/docs/functions.html#variable-number-of-arguments-varargs)
- [Kotlin 공식 문서 - Spread Operator](https://kotlinlang.org/docs/functions.html#spread-operator)
- [Oracle Java Documentation - Varargs](https://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html#varargs)
