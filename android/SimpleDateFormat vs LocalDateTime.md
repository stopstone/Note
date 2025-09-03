# SimpleDateFormat vs LocalDateTime

> Android에서 날짜와 시간을 다룰 때 가장 많이 사용하는 `SimpleDateFormat`과 `LocalDateTime`의 차이점을 알아보겠습니다.  
> Gemini 코드 리뷰를 통해 받은 피드백을 바탕으로, 왜 `LocalDateTime`을 사용해야 하는지, 그리고 두 방식의 장단점을 공부해보겠습니다.

---

## 1. 개선이 필요한 코드

```kotlin
private fun calculateExpiresIn(expiresAtString: String?): Long? {
    return expiresAtString?.let { expiresAt ->
        val dateFormat = SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.getDefault())
        val expiresAtTime = dateFormat.parse(expiresAt)
        val currentTime = System.currentTimeMillis()
        ((expiresAtTime?.time ?: currentTime) - currentTime) / 1000 // 초 단위로 변환
    }
}
```

이 코드에서 `SimpleDateFormat`을 사용하고 있는데, 코드 리뷰어가 `LocalDateTime` 사용을 권장했습니다.

---

## 2. SimpleDateFormat의 문제점

### 2.1 스레드 안전성 문제 (Thread Safety)

**가장 큰 문제는 `SimpleDateFormat`이 스레드에 안전하지 않다는 점입니다.**

```kotlin
// 위험한 코드 - 멀티스레드 환경에서 문제 발생 가능
class DateUtils {
    private val dateFormat = SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.getDefault())
    
    fun parseDate(dateString: String): Date? {
        return dateFormat.parse(dateString) // 여러 스레드에서 동시 접근 시 오류 발생
    }
}
```

**문제 상황:**
- 여러 스레드에서 동시에 `SimpleDateFormat` 인스턴스에 접근할 때
- 내부 상태가 공유되어 예상치 못한 결과나 `NumberFormatException` 발생
- 특히 Android의 멀티스레드 환경에서는 더욱 위험

### 2.2 해결 방법들

**방법 1: 매번 새 인스턴스 생성**
```kotlin
// 안전하지만 비효율적
fun parseDate(dateString: String): Date? {
    val dateFormat = SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.getDefault())
    return dateFormat.parse(dateString)
}
```

**방법 2: synchronized 사용**
```kotlin
// 안전하지만 성능 저하
class DateUtils {
    private val dateFormat = SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.getDefault())
    
    @Synchronized
    fun parseDate(dateString: String): Date? {
        return dateFormat.parse(dateString)
    }
}
```

**방법 3: ThreadLocal 사용**
```kotlin
// 복잡하지만 효율적
class DateUtils {
    private val dateFormat = ThreadLocal.withInitial {
        SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.getDefault())
    }
    
    fun parseDate(dateString: String): Date? {
        return dateFormat.get().parse(dateString)
    }
}
```

---

## 3. LocalDateTime의 장점

### 3.1 스레드 안전성

**`LocalDateTime`은 불변(immutable) 객체로 스레드에 안전합니다.**

```kotlin
// 안전한 코드 - 불변 객체로 스레드 안전
private fun calculateExpiresIn(expiresAtString: String?): Long? {
    return expiresAtString?.let { expiresAt ->
        try {
            val formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")
            val expiresAtTime = LocalDateTime.parse(expiresAt, formatter)
            val expiresAtMillis = expiresAtTime
                .atZone(ZoneId.systemDefault())
                .toInstant()
                .toEpochMilli()
            (expiresAtMillis - System.currentTimeMillis()) / 1000
        } catch (e: DateTimeParseException) {
            null
        }
    }
}
```

### 3.2 현대적인 API 설계

**`java.time` 패키지는 더 직관적이고 명확한 API를 제공합니다.**

```kotlin
// SimpleDateFormat 방식
val dateFormat = SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.getDefault())
val date = dateFormat.parse("2024-01-15 14:30:00")

// LocalDateTime 방식
val formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")
val dateTime = LocalDateTime.parse("2024-01-15 14:30:00", formatter)
```

### 3.3 타입 안전성

**`LocalDateTime`은 컴파일 타임에 타입 안전성을 보장합니다.**

```kotlin
// 명확한 타입 구분
val localDate = LocalDate.now()        // 날짜만
val localTime = LocalTime.now()        // 시간만  
val localDateTime = LocalDateTime.now() // 날짜 + 시간
val zonedDateTime = ZonedDateTime.now() // 날짜 + 시간 + 타임존
```

---

## 4. 성능 비교

### 4.1 메모리 사용량

| 방식 | 메모리 사용량 | 설명 |
|------|---------------|------|
| SimpleDateFormat | 높음 | 매번 새 인스턴스 생성 시 GC 부담 |
| LocalDateTime | 낮음 | 불변 객체로 효율적인 메모리 관리 |

### 4.2 성능 테스트

```kotlin
// 성능 비교 예시 (실제 측정값은 환경에 따라 다름)
fun performanceTest() {
    val iterations = 10000
    
    // SimpleDateFormat (매번 새 인스턴스)
    val simpleDateFormatTime = measureTimeMillis {
        repeat(iterations) {
            val formatter = SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.getDefault())
            formatter.parse("2024-01-15 14:30:00")
        }
    }
    
    // LocalDateTime
    val localDateTimeTime = measureTimeMillis {
        val formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")
        repeat(iterations) {
            LocalDateTime.parse("2024-01-15 14:30:00", formatter)
        }
    }
    
    println("SimpleDateFormat: ${simpleDateFormatTime}ms")
    println("LocalDateTime: ${localDateTimeTime}ms")
}
```

---

## 5. 마이그레이션 가이드

### 5.1 기본 변환

```kotlin
// Before: SimpleDateFormat
val dateFormat = SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.getDefault())
val date = dateFormat.parse(dateString)

// After: LocalDateTime
val formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")
val dateTime = LocalDateTime.parse(dateString, formatter)
```

### 5.2 날짜 포맷 변환

```kotlin
// Before: SimpleDateFormat
val inputFormat = SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.getDefault())
val outputFormat = SimpleDateFormat("MM/dd/yyyy", Locale.getDefault())
val date = inputFormat.parse(inputDate)
val formattedDate = outputFormat.format(date)

// After: LocalDateTime
val inputFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")
val outputFormatter = DateTimeFormatter.ofPattern("MM/dd/yyyy")
val dateTime = LocalDateTime.parse(inputDate, inputFormatter)
val formattedDate = dateTime.format(outputFormatter)
```

### 5.3 밀리초 변환

```kotlin
// Before: SimpleDateFormat
val dateFormat = SimpleDateFormat("yyyy-MM-dd HH:mm:ss", Locale.getDefault())
val date = dateFormat.parse(dateString)
val millis = date?.time ?: 0L

// After: LocalDateTime
val formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")
val dateTime = LocalDateTime.parse(dateString, formatter)
val millis = dateTime.atZone(ZoneId.systemDefault()).toInstant().toEpochMilli()
```

---

## 6. 주의사항

### 6.1 API 레벨 확인

**`java.time` 패키지는 API 레벨 26(Android 8.0)부터 사용 가능합니다.**

```kotlin
// minSdk 26 이상에서만 사용 가능
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
    // LocalDateTime 사용
} else {
    // SimpleDateFormat 사용 (또는 ThreeTenABP 라이브러리)
}
```

### 6.2 ThreeTenABP 라이브러리

**API 레벨 26 미만에서도 `java.time`을 사용하려면 ThreeTenABP 라이브러리를 사용하세요.**

```kotlin
// build.gradle (Module: app)
implementation 'com.jakewharton.threetenabp:threetenabp:1.4.6'

// Application 클래스에서 초기화
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        AndroidThreeTen.init(this)
    }
}
```
