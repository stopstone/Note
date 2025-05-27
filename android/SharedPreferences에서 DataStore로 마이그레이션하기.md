# SharedPreferences에서 DataStore로 마이그레이션하기

> 기존 앱에서 SharedPreferences를 사용하고 있지만, 내부 구현을 DataStore로 교체하여 더 안전하고 현대적인 방식으로 전환하고자 합니다.  
> 이 글에서는 `AuthPreferences` 클래스의 메서드 구조나 이름은 그대로 유지하면서, 내부 동작만 SharedPreferences에서 DataStore로 바꿔주는 것을 목표로 합니다.  
> 이렇게 하면 Repository나 ViewModel 같은 다른 계층의 코드를 수정하지 않고도 안전하게 마이그레이션할 수 있습니다.   

---

## 0. 의존성 추가

```kotlin
// build.gradle (module)
dependencies {
    implementation("androidx.datastore:datastore-preferences:1.0.0")
    // 코루틴 사용 시
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.3")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3")
}
```

---

## 1. 기존 SharedPreferences 구조 예시

```kotlin
class AuthPreferences(context: Context) {
    private val prefs = context.getSharedPreferences("auth", Context.MODE_PRIVATE)

    fun putAccessToken(token: String) {
        prefs.edit().putString("access_token", token).apply()
    }

    fun putRefreshToken(token: String) {
        prefs.edit().putString("refresh_token", token).apply()
    }

    fun getAccessToken(): String? {
        return prefs.getString("access_token", null)
    }

    fun getRefreshToken(): String? {
        return prefs.getString("refresh_token", null)
    }
}
```

---

## 2. DataStore 래퍼 구성

> `SharedPreferencesMigration`은 기존 SharedPreferences에 저장된 데이터를 DataStore로 자동 이전할 수 있도록 도와주는 클래스입니다.  
> 앱 최초 실행 시 한 번만 수행되며, 기존 SharedPreferences 파일 이름과 동일하게 지정하면 기존의 키-값 쌍이 그대로 복사됩니다.  
>
> 하지만 기존 데이터를 유지할 필요가 없다면 이 마이그레이션은 생략할 수 있으며, DataStore만 사용하는 방식으로도 충분히 설정이 가능합니다.  
> 아래 예시는 SharedPreferencesMigration 없이 DataStore를 구성한 코드입니다.

```kotlin
val Context.authDataStore: DataStore<Preferences> by preferencesDataStore(
    name = "auth"
)

class AuthPreferences(private val context: Context) {
    private val dataStore = context.authDataStore

    private object Keys {
        val ACCESS_TOKEN = stringPreferencesKey("access_token")
        val REFRESH_TOKEN = stringPreferencesKey("refresh_token")
    }

    suspend fun putAccessToken(token: String) {
        dataStore.edit { prefs ->
            prefs[Keys.ACCESS_TOKEN] = token
        }
    }

    suspend fun putRefreshToken(token: String) {
        dataStore.edit { prefs ->
            prefs[Keys.REFRESH_TOKEN] = token
        }
    }

    suspend fun getAccessToken(): String? {
        return dataStore.data
            .catch { emit(emptyPreferences()) }
            .map { it[Keys.ACCESS_TOKEN] }
            .firstOrNull()
    }

    suspend fun getRefreshToken(): String? {
        return dataStore.data
            .catch { emit(emptyPreferences()) }
            .map { it[Keys.REFRESH_TOKEN] }
            .firstOrNull()
    }

    val accessTokenFlow: Flow<String?> = dataStore.data
        .catch { emit(emptyPreferences()) }
        .map { prefs -> prefs[Keys.ACCESS_TOKEN] }

    val refreshTokenFlow: Flow<String?> = dataStore.data
        .catch { emit(emptyPreferences()) }
        .map { prefs -> prefs[Keys.REFRESH_TOKEN] }
}
```

---

## 2-1. SharedPreferencesMigration 사용 여부에 따른 차이점

> SharedPreferencesMigration은 기존 데이터를 유지하고 싶은 경우에만 필요하며, 전혀 사용하지 않아도 DataStore는 정상 작동합니다.  
> 아래 표는 두 방식의 차이를 정리한 내용입니다.  

| 항목            | SharedPreferencesMigration 사용  | 사용 안 함                 |
| ------------- | ------------------------------ | ---------------------- |
| **목적**        | 기존 SharedPreferences 데이터 자동 이전 | 완전히 새로 시작              |
| **기존 데이터 보존** | 유지됨 (예: accessToken, 설정 등 복사됨) | 모두 무시됨 (초기화)           |
| **파일 이름**     | 같아야 함 (`auth`)                 | 자유롭게 지정 가능             |
| **사용 시점**     | 최초 1회 실행 시 자동 동작               | 항상 DataStore에서 새로 시작   |
| **추천 상황**     | 운영 중 앱에서 이전 데이터 유지할 때          | 신규 프로젝트 또는 데이터 리셋 필요 시 |

---

## 3. Repository와의 연결 유지

> 마이그레이션 전후로 `AuthPreferences`의 public 메서드 시그니처를 그대로 유지했기 때문에, Repository의 구현에는 아무런 변경이 필요하지 않습니다.  
> 내부 구현이 SharedPreferences에서 DataStore로 바뀌었지만, 외부에서는 동일한 방식으로 사용할 수 있습니다.    

```kotlin
class AuthRepository(private val authPreferences: AuthPreferences) {
    suspend fun setAccessToken(token: String) {
        authPreferences.putAccessToken(token)
    }

    suspend fun setRefreshToken(token: String) {
        authPreferences.putRefreshToken(token)
    }

    suspend fun getAccessToken(): String? {
        return authPreferences.getAccessToken()
    }

    suspend fun getRefreshToken(): String? {
        return authPreferences.getRefreshToken()
    }

    val accessTokenFlow = authPreferences.accessTokenFlow
    val refreshTokenFlow = authPreferences.refreshTokenFlow
}
```

---

## 4. ViewModel 사용 예시

> Repository에서 제공하는 인터페이스가 그대로 유지되므로, ViewModel의 코드 또한 변경할 필요가 없습니다.
> 마이그레이션 여부에 상관없이 기존 방식대로 데이터를 불러오고 저장할 수 있습니다.

```kotlin
class AuthViewModel(
    private val repository: AuthRepository
) : ViewModel() {

    val accessToken = repository.accessTokenFlow
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), null)

    val refreshToken = repository.refreshTokenFlow
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), null)

    fun updateTokens(access: String, refresh: String) {
        viewModelScope.launch {
            repository.setAccessToken(access)
            repository.setRefreshToken(refresh)
        }
    }
}
```

---
