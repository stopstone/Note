# DataStore

> 안드로이드에서는 간단한 설정값을 저장할 때 SharedPreferences를 많이 사용했습니다.  
> 하지만 SharedPreferences는 스레드 안전하지 않고, 비동기 처리가 어려우며 데이터 손실 위험이 존재합니다.  
> 이를 개선하기 위해 Jetpack에서 비동기 기반의 안전한 저장소인 DataStore가 등장했습니다.  

## 1. DataStore란?

Jetpack에서 제공하는 키-값 기반 데이터 저장 솔루션으로, 코루틴과 Flow를 기반으로 비동기적으로 작동합니다.  
내부적으로 트랜잭션 처리와 스레드 안전성이 보장되어, 기존 SharedPreferences의 단점을 해결합니다.

DataStore는 두 가지 종류가 있습니다.

* Preferences DataStore: key-value 기반, SharedPreferences 대체용
* Proto DataStore: Protobuf 기반, 타입 안전한 구조적 데이터 저장

이번 글에서는 **Preferences DataStore**에 대해서만 다룹니다.

## 2. SharedPreferences와 차이점

| 항목         | SharedPreferences    | Preferences DataStore |
| ---------- | -------------------- | --------------------- |
| 비동기 지원     | 부분적 (apply만 비동기)     | 완전 비동기 (suspend 지원)   |
| 스레드 안전성    | 낮음                   | 높음 (내부 동기화 처리)        |
| Flow 지원    | 미지원                  | 지원 (Reactive UI에 적합)  |
| 타입 안전성     | 낮음 (key-value 직접 입력) | 높음 (type-safe key 제공) |
| 데이터 손실 가능성 | 존재                   | 없음 (트랜잭션 처리)          |

SharedPreferences는 UI 스레드에서 I/O 작업을 수행할 경우 ANR 위험이 있으므로, 사용에 주의가 필요합니다.

## 3. 사용 방법

### 3.1 의존성 추가 (build.gradle)

```kotlin
dependencies {
    implementation("androidx.datastore:datastore-preferences:1.0.0")
}
```

### 3.2 DataStore 인스턴스 생성

```kotlin
val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "settings")
```

> Context 확장 프로퍼티로 정의하는 것이 일반적이며, `Application`, `Activity` 등 UI 컴포넌트에서 사용합니다.
> DataStore 접근 로직은 보통 `Repository` 계층으로 분리하여 관리합니다.

### 3.3 값 저장하기

```kotlin
suspend fun saveDarkMode(context: Context, isEnabled: Boolean) {
    val key = booleanPreferencesKey("dark_mode")
    context.dataStore.edit { prefs ->
        prefs[key] = isEnabled
    }
}
```

### 3.4 값 읽기

```kotlin
val darkModeFlow: Flow<Boolean> = context.dataStore.data
    .map { prefs ->
        prefs[booleanPreferencesKey("dark_mode")] ?: false
    }
```

> Flow로 반환되므로 Compose에선 `collectAsState()`를 통해 UI와 바로 연결할 수 있습니다.

## 4. DataStore 장점

* 코루틴 기반의 완전 비동기 처리로 메인 스레드 안전
* Flow를 통한 반응형 데이터 처리 가능
* 트랜잭션 기반의 안정적인 저장
* 스레드 안전성 보장
* 명확한 API 설계로 타입 안전성 향상

## 5. 참고자료

* [Android Developers - DataStore](https://developer.android.com/topic/libraries/architecture/datastore)
