# Access 토큰과 Refresh 토큰의 특징

> 안드로이드 앱에서 사용자 인증을 구현할 때 토큰 기반 인증은 많이 사용되는 방식 중 하나입니다.    
> 그런데 Access 토큰과 Refresh 토큰의 차이점과 역할에 대해 공부해보겠습니다.  

---

## 1. 기본 개념부터 이해하기

### Access 토큰이란?
**Access 토큰은 인증된 사용자의 리소스에 대한 접근 권한을 나타내는 토큰**입니다.  
클라이언트가 API 요청을 할 때마다 이 토큰을 함께 전송하여, 서버가 "이 사용자가 누구인지, 어떤 권한을 가지고 있는지"를 확인할 수 있게 해줍니다.

```kotlin
// API 요청 시 Access 토큰 사용
val request = originalRequest.newBuilder()
    .addHeader("Authorization", "Bearer $accessToken")
    .build()
```

### Refresh 토큰이란?
**Refresh 토큰은 Access 토큰의 갱신을 위한 토큰**입니다.  
Access 토큰이 만료되었을 때, 사용자가 다시 로그인하지 않고도 새로운 Access 토큰을 발급받을 수 있게 해주는 역할을 합니다.

```kotlin
// Access 토큰 갱신 요청
suspend fun refreshAccessToken(): Result<TokenResponse> {
    return try {
        val response = authApi.refreshToken(refreshToken)
        Result.success(response)
    } catch (e: Exception) {
        Result.failure(e)
    }
}
```

---

## 2. 핵심 차이점 정리

| 구분 | Access 토큰 | Refresh 토큰 |
|:---|:---|:---|
| **목적** | API 요청 시 인증 | Access 토큰 갱신 |
| **유효 기간** | 짧음 (몇 분~몇 시간) | 김 (몇 일~몇 주) |
| **사용 빈도** | 매 API 요청마다 사용 | Access 토큰 만료 시만 사용 |
| **보안 수준** | 상대적으로 낮음 (자주 노출) | 높음 (가끔만 사용) |
| **저장 위치** | 메모리 권장 | 안전한 저장소 (암호화) |

---

## 3. 왜 두 개의 토큰을 사용할까?

### 보안과 편의성의 균형

만약 Access 토큰만 사용한다면 어떻게 될까요?

#### Access 토큰만 사용하는 경우 (문제점)
```kotlin
// 문제: 긴 유효 기간 → 보안 위험
val longLivedAccessToken = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." // 1주일 유효

// 문제: 짧은 유효 기간 → 사용자 불편
val shortLivedAccessToken = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." // 15분 유효
// → 15분마다 다시 로그인해야 함
```

#### 두 토큰을 함께 사용하는 경우 (해결책)
```kotlin
// Access 토큰: 짧은 유효 기간으로 보안 강화
val accessToken = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." // 15분 유효

// Refresh 토큰: 긴 유효 기간으로 편의성 확보
val refreshToken = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..." // 7일 유효
```

**결과**: 보안은 강화하면서도 사용자 경험은 유지할 수 있습니다!

---

## 4. 실제 동작 흐름

### 4-1. 최초 로그인
```kotlin
// 1. 사용자 로그인
suspend fun login(email: String, password: String): Result<AuthResponse> {
    return try {
        val response = authApi.login(LoginRequest(email, password))
        
        // 2. 두 토큰 모두 저장
        tokenStorage.saveAccessToken(response.accessToken)
        tokenStorage.saveRefreshToken(response.refreshToken)
        
        Result.success(response)
    } catch (e: Exception) {
        Result.failure(e)
    }
}
```

### 4-2. API 요청 (정상 케이스)
```kotlin
// Access 토큰으로 API 요청
suspend fun fetchUserProfile(): Result<User> {
    return try {
        val accessToken = tokenStorage.getAccessToken()
        val response = userApi.getProfile("Bearer $accessToken")
        Result.success(response)
    } catch (e: HttpException) {
        if (e.code() == 401) {
            // Access 토큰 만료 → 자동 갱신 시도
            handleTokenExpiration()
        } else {
            Result.failure(e)
        }
    }
}
```

### 4-3. 토큰 갱신 (만료 시)
```kotlin
private suspend fun handleTokenExpiration(): Result<User> {
    return try {
        // 1. Refresh 토큰으로 새 Access 토큰 발급
        val refreshToken = tokenStorage.getRefreshToken()
        val tokenResponse = authApi.refreshToken(RefreshRequest(refreshToken))
        
        // 2. 새 Access 토큰 저장
        tokenStorage.saveAccessToken(tokenResponse.accessToken)
        
        // 3. 원래 요청 재시도
        val newAccessToken = tokenResponse.accessToken
        val response = userApi.getProfile("Bearer $newAccessToken")
        Result.success(response)
        
    } catch (e: Exception) {
        // Refresh 토큰도 만료 → 재로그인 필요
        redirectToLogin()
        Result.failure(e)
    }
}
```

---

## 5. 안드로이드에서의 구현 예시

### 5-1. 토큰 저장소 구현
```kotlin
@Singleton
class TokenStorage @Inject constructor(
    private val encryptedPrefs: EncryptedSharedPreferences
) {
    
    fun saveAccessToken(token: String) {
        encryptedPrefs.edit()
            .putString("access_token", token)
            .apply()
    }
    
    fun saveRefreshToken(token: String) {
        encryptedPrefs.edit()
            .putString("refresh_token", token)
            .apply()
    }
    
    fun getAccessToken(): String? {
        return encryptedPrefs.getString("access_token", null)
    }
    
    fun getRefreshToken(): String? {
        return encryptedPrefs.getString("refresh_token", null)
    }
    
    fun clearTokens() {
        encryptedPrefs.edit()
            .remove("access_token")
            .remove("refresh_token")
            .apply()
    }
}
```

### 5-2. OkHttp Interceptor로 자동 토큰 관리
```kotlin
class AuthInterceptor @Inject constructor(
    private val tokenStorage: TokenStorage
) : Interceptor {
    
    override fun intercept(chain: Interceptor.Chain): Response {
        val originalRequest = chain.request()
        val accessToken = tokenStorage.getAccessToken()
        
        // Access 토큰이 있으면 헤더에 추가
        val newRequest = if (accessToken != null) {
            originalRequest.newBuilder()
                .addHeader("Authorization", "Bearer $accessToken")
                .build()
        } else {
            originalRequest
        }
        
        val response = chain.proceed(newRequest)
        
        // 401 응답 시 토큰 갱신 시도
        if (response.code == 401 && accessToken != null) {
            response.close()
            
            // 토큰 갱신 로직은 별도 처리
            // (여기서는 단순화)
            return response.newBuilder()
                .code(401)
                .message("Token expired")
                .build()
        }
        
        return response
    }
}
```

---
