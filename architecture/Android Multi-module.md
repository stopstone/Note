# Android Multi-module

> 안드로이드 앱이 점점 커지면서 단일 모듈로 개발하기 어려워지는 순간이 옵니다.   
> 빌드 시간은 길어지고, 코드는 복잡해질 수 있습니다.   
> 모듈화(Multi-module)는 하나의 큰 애플리케이션을 여러 개의 독립적인 모듈로 분리하는 아키텍처 패턴입니다.  
> 이 글에서는 모듈화의 장점부터 실제 구현 시 필요한 것들을 정리해보겠습니다.  


## 1. 모듈화란 무엇인가

모듈화(Multi-module)는 하나의 큰 애플리케이션을 여러 개의 독립적인 모듈로 분리하는 아키텍처 패턴입니다.  

```kotlin
// 단일 모듈 구조
app/
├── src/main/java/
│   ├── ui/
│   ├── data/
│   ├── domain/
│   └── di/
└── build.gradle

// 모듈화된 구조
app/                    // 메인 앱 모듈
feature-auth/          // 인증 기능 모듈
feature-profile/       // 프로필 기능 모듈
core-network/          // 네트워크 핵심 모듈
core-database/         // 데이터베이스 핵심 모듈
core-ui/             // 공통 UI 컴포넌트 모듈
```

## 2. 모듈화의 주요 장점

### 1. 빌드 시간 단축

가장 체감되는 장점은 빌드 시간의 단축입니다.  
변경된 모듈만 다시 빌드하기 때문에 전체 프로젝트를 빌드할 필요가 없어집니다.

```kotlin
// settings.gradle.kts
include(":app")
include(":feature-auth")
include(":feature-profile") 
include(":core-network")

// feature-auth 모듈만 수정했다면
// feature-auth 모듈만 다시 빌드됩니다
```


### 2. 코드 재사용성 향상

공통 기능을 별도 모듈로 분리하면 다른 프로젝트에서도 쉽게 재사용할 수 있습니다.

```kotlin
// core-ui 모듈의 공통 컴포넌트
// build.gradle.kts
dependencies {
    implementation("androidx.compose.ui:ui:$compose_version")
    implementation("androidx.compose.material3:material3:$material3_version")
}

// CustomButton.kt
@Composable
fun CustomButton(
    text: String,
    onClick: () -> Unit,
    modifier: Modifier = Modifier
) {
    Button(
        onClick = onClick,
        modifier = modifier
    ) {
        Text(text = text)
    }
}
```

### 3. 팀 협업 효율성 증대

각 팀이나 개발자가 담당하는 모듈을 명확히 분리하여 개발이 가능해집니다.

```kotlin
// 팀 A: feature-payment 담당
// 팀 B: feature-product 담당
// 코어 팀: core-network, core-database 담당

// 각 팀이 독립적으로 개발 가능
```


## 3. 구현 시 주요 고려사항

### 1. 모듈 분리 전략

모듈을 어떻게 나눌지 결정하는 것이 가장 중요합니다. 일반적으로 다음과 같은 전략을 사용합니다:

```kotlin
// 기능별 분리 (Feature Module)
feature-login/
feature-signup/
feature-profile/
feature-payment/

// 계층별 분리 (Layer Module)  
data/
domain/
presentation/

// 플랫폼별 분리 (Platform Module)
android-app/
shared/
ios-app/          // KMP 사용 시
```

### 2. 의존성 관리

모듈 간 의존성은 순환 참조가 발생하지 않도록 주의깊게 설계해야 합니다.

```kotlin
// app/build.gradle.kts
dependencies {
    implementation(project(":feature-auth"))
    implementation(project(":feature-profile"))
    implementation(project(":core-ui"))
}

// feature-auth/build.gradle.kts  
dependencies {
    implementation(project(":core-network"))
    implementation(project(":core-ui"))
    // 순환 참조 금지: feature-profile 의존하면 안됨
}

// core-network/build.gradle.kts
dependencies {
    implementation("com.squareup.retrofit2:retrofit:$retrofit_version")
    // feature 모듈 의존 금지
}
```

### 3. 인터페이스와 추상화

모듈 간 통신은 인터페이스를 통해 이루어져야 합니다.

```kotlin
// core-auth/AuthManager.kt
interface AuthManager {
    suspend fun login(email: String, password: String): Result<User>
    suspend fun logout(): Result<Unit>
    fun isLoggedIn(): Boolean
}

// feature-auth/AuthManagerImpl.kt
class AuthManagerImpl(
    private val authApi: AuthApi,
    private val tokenStorage: TokenStorage
) : AuthManager {
    
    override suspend fun login(email: String, password: String): Result<User> {
        return try {
            val response = authApi.login(LoginRequest(email, password))
            tokenStorage.saveToken(response.token)
            Result.success(response.user)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    override suspend fun logout(): Result<Unit> {
        return try {
            authApi.logout()
            tokenStorage.clearToken()
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    override fun isLoggedIn(): Boolean {
        return tokenStorage.getToken() != null
    }
}
```


### 4. 버전 관리와 호환성

모듈 간 버전 호환성을 관리하기 위해 버전 카탈로그나 buildSrc를 활용합니다.

```kotlin
// gradle/libs.versions.toml
[versions]
kotlin = "1.9.10"
compose = "1.5.4"
retrofit = "2.9.0"

[libraries]
kotlin-stdlib = { module = "org.jetbrains.kotlin:kotlin-stdlib", version.ref = "kotlin" }
compose-ui = { module = "androidx.compose.ui:ui", version.ref = "compose" }
retrofit-core = { module = "com.squareup.retrofit2:retrofit", version.ref = "retrofit" }

[bundles]
compose = ["compose-ui", "compose-material3", "compose-preview"]
networking = ["retrofit-core", "retrofit-gson", "okhttp-logging"]
```


## 참고 자료

- [Android Developers - Modularization](https://developer.android.com/topic/modularization)
- [Dagger Hilt - Multi-module best practices](https://dagger.dev/hilt/modules)
- [Android Architecture Components - Navigation in multi-module apps](https://developer.android.com/guide/navigation/navigation-multi-module)
- [Gradle - Multi-project builds optimization](https://docs.gradle.org/current/userguide/multi_project_builds.html) 
