# SharedPreferences

> 안드로이드에서 간단한 데이터를 저장할 때 흔히 사용되는 `SharedPreferences` 를 사용합니다.   
> 이 글에서는 `SharedPreferences`의 개념과 사용법, 주의사항까지 실전 중심으로 정리합니다.    

## 1. SharedPreferences란?

* `Key-Value` 쌍의 데이터를 영구 저장할 수 있는 Android API
* 내부 저장소에 `XML` 형태로 저장되며 앱 외부에서는 접근 불가
* 설정값, 로그인 여부 등 간단한 상태 저장에 적합

## 2. 주요 메서드와 사용 예시

```kotlin
val prefs = context.getSharedPreferences("setting", Context.MODE_PRIVATE)

prefs.edit().putString("user_name", "지석").apply()

val name = prefs.getString("user_name", "기본값")

prefs.edit().remove("user_name").apply()

prefs.edit().clear().apply()
```

### 저장 방식 차이

| 메서드        | 특징                            |
| ---------- | ----------------------------- |
| `apply()`  | 비동기 저장, 메인 스레드 차단 없음 (권장)     |
| `commit()` | 동기 저장, 저장 완료 여부 반환, ANR 위험 있음 |

## 3. Context.MODE 옵션

| 모드                   | 설명                         |
| -------------------- | -------------------------- |
| `MODE_PRIVATE`       | 기본. 앱 내부에서만 접근 가능          |
| `MODE_MULTI_PROCESS` | 여러 프로세스 간 공유 (deprecated)  |
| `MODE_APPEND`        | 기존 파일에 데이터 추가 (거의 사용되지 않음) |

## 4. 실무 예제: 자동 로그인 여부 저장

```kotlin
fun saveLoginState(context: Context, isLoggedIn: Boolean) {
    val prefs = context.getSharedPreferences("auth", Context.MODE_PRIVATE)
    prefs.edit().putBoolean("is_logged_in", isLoggedIn).apply()
}

fun isUserLoggedIn(context: Context): Boolean {
    val prefs = context.getSharedPreferences("auth", Context.MODE_PRIVATE)
    return prefs.getBoolean("is_logged_in", false)
}
```

## 참고 자료

* Android 공식 문서: [SharedPreferences](https://developer.android.com/reference/android/content/SharedPreferences)
