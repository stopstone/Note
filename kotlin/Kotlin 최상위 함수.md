# Kotlin 최상위 함수

> Kotlin 최상위 함수(Top-Level Function)는 클래스나 객체 내부가 아닌 파일의 최상위 레벨에서 정의할 수 있는 함수입니다.  
> Java와 달리 모든 함수를 클래스 안에 넣지 않아도 되어 코드 작성이 더욱 자유롭고 간결해집니다.  
> 최상위 함수의 개념과 활용 방법에 대해 알아보겠습니다.

---

## 1. 최상위 함수란?

최상위 함수는 파일의 최상위 레벨에 직접 선언되는 함수로, 어떤 클래스에도 속하지 않습니다.  
Kotlin 파일(.kt)에서 클래스 외부에 직접 함수를 정의하면 최상위 함수가 됩니다.

```kotlin
// Utils.kt
package com.example.utils

fun calculateSum(a: Int, b: Int): Int {
    return a + b
}

fun printMessage(message: String) {
    println("메시지: $message")
}
```

위 예제에서 `calculateSum`과 `printMessage`는 어떤 클래스에도 속하지 않은 최상위 함수입니다.

---

## 2. Java와의 차이점

| 항목               | Kotlin 최상위 함수                     | Java 방식                         |
|--------------------|----------------------------------------|----------------------------------|
| 정의 위치          | 파일 최상위 레벨에서 정의 가능          | 반드시 클래스 내부에 정의해야 함 |
| 클래스 필요 여부   | 불필요                                 | 필수 (모든 메서드는 클래스 소속)  |
| 유틸 함수 작성     | 자연스럽고 간결                         | 유틸 클래스 + static 메서드 필요 |
| 호출 방식          | 직접 호출 또는 import 후 호출           | 클래스명.메서드명() 형태로 호출  |

### Java에서의 유틸 함수

Java에서는 유틸 함수를 만들 때 반드시 클래스 안에 static 메서드로 정의해야 합니다.

```java
// MathUtils.java
public class MathUtils {
    private MathUtils() {} // 인스턴스 생성 방지
    
    public static int calculateSum(int a, int b) {
        return a + b;
    }
}

// 사용
int result = MathUtils.calculateSum(5, 3);
```

### Kotlin에서의 최상위 함수

```kotlin
// MathUtils.kt
package com.example.utils

fun calculateSum(a: Int, b: Int): Int {
    return a + b
}

// 사용
val result = calculateSum(5, 3)
```

훨씬 간결하고 직관적입니다.

---

## 3. Kotlin 최상위 함수를 사용하는 이유

- **코드 간결성**: 불필요한 클래스 래핑이 없어 코드가 간결해집니다.
- **가독성 향상**: 함수의 목적이 명확하게 드러나고, 클래스 이름이 필요 없어 가독성이 좋습니다.
- **함수형 프로그래밍**: 함수를 일급 시민(First-Class Citizen)으로 다루는 함수형 프로그래밍 스타일에 적합합니다.
- **유틸리티 함수 관리**: 공통 유틸리티 함수를 클래스 없이 관리할 수 있습니다.

---

## 4. 컴파일 시 Java 코드로 변환

Kotlin의 최상위 함수는 JVM에서 실행되기 위해 컴파일 시 Java 클래스로 변환됩니다.

### Kotlin 코드

```kotlin
// StringUtils.kt
package com.example.utils

fun toUpperCase(text: String): String {
    return text.uppercase()
}
```

### 컴파일 후 생성되는 Java 코드

```java
// StringUtilsKt.java
package com.example.utils;

public final class StringUtilsKt {
    public static final String toUpperCase(String text) {
        return text.toUpperCase();
    }
}
```

Kotlin은 파일명에 `Kt` 접미사를 붙여 클래스를 자동 생성하고, 최상위 함수는 `public static final` 메서드로 변환됩니다.

---

## 5. Java에서 Kotlin 최상위 함수 호출하기

Java에서 Kotlin의 최상위 함수를 호출할 때는 자동 생성된 클래스 이름을 사용해야 합니다.

```kotlin
// MathUtils.kt
package com.example.utils

fun add(a: Int, b: Int): Int = a + b
```

```java
// Java에서 호출
import com.example.utils.MathUtilsKt;

public class Main {
    public static void main(String[] args) {
        int result = MathUtilsKt.add(5, 3);
        System.out.println(result); // 출력: 8
    }
}
```

### @JvmName으로 Java 클래스명 변경하기

기본적으로 파일명에 `Kt`가 붙는데, `@JvmName` 어노테이션으로 원하는 이름으로 변경할 수 있습니다.

```kotlin
// MathUtils.kt
@file:JvmName("MathHelper")
package com.example.utils

fun add(a: Int, b: Int): Int = a + b
```

```java
// Java에서 호출
import com.example.utils.MathHelper;

public class Main {
    public static void main(String[] args) {
        int result = MathHelper.add(5, 3); // MathUtilsKt 대신 MathHelper 사용
        System.out.println(result);
    }
}
```

---

## 6. 예시: 날짜 포맷 유틸리티

Android 개발에서 자주 사용하는 날짜 포맷 유틸리티를 최상위 함수로 작성해보겠습니다.

```kotlin
// DateUtils.kt
package com.example.utils

import java.text.SimpleDateFormat
import java.util.*

fun formatDate(date: Date, pattern: String = "yyyy-MM-dd"): String {
    val formatter = SimpleDateFormat(pattern, Locale.getDefault())
    return formatter.format(date)
}

fun getCurrentDateTime(): String {
    return formatDate(Date(), "yyyy-MM-dd HH:mm:ss")
}

fun isToday(date: Date): Boolean {
    val today = Calendar.getInstance()
    val targetDate = Calendar.getInstance().apply { time = date }
    return today.get(Calendar.YEAR) == targetDate.get(Calendar.YEAR) &&
           today.get(Calendar.DAY_OF_YEAR) == targetDate.get(Calendar.DAY_OF_YEAR)
}
```

### 사용 예시

```kotlin
// MainActivity.kt
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        val now = Date()
        val formattedDate = formatDate(now)
        val currentDateTime = getCurrentDateTime()
        val isTodayCheck = isToday(now)
        
        Log.d("DateUtils", "포맷된 날짜: $formattedDate")
        Log.d("DateUtils", "현재 시간: $currentDateTime")
        Log.d("DateUtils", "오늘인가요?: $isTodayCheck")
    }
}
```

---

## 7. 최상위 함수 vs 확장 함수

최상위 함수와 확장 함수는 비슷해 보이지만 목적이 다릅니다.

| 구분          | 최상위 함수                            | 확장 함수                          |
|---------------|----------------------------------------|-----------------------------------|
| 정의 위치     | 파일 최상위 레벨                       | 파일 최상위 레벨 (리시버 타입 지정) |
| 호출 방식     | `함수명()`                             | `객체.함수명()`                    |
| 목적          | 범용 유틸리티 함수                     | 특정 타입에 기능 추가              |
| this          | 없음                                   | 리시버 객체를 가리킴               |

### 예시

```kotlin
// 최상위 함수
fun calculateTax(amount: Int): Int {
    return (amount * 0.1).toInt()
}

// 확장 함수
fun Int.calculateTax(): Int {
    return (this * 0.1).toInt()
}

// 사용
val tax1 = calculateTax(1000) // 최상위 함수 호출
val tax2 = 1000.calculateTax() // 확장 함수 호출
```

---
