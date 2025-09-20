# Retrofit과 OkHttp의 차이점  

> 안드로이드 개발에서 네트워크 통신을 할 때 Retrofit과 OkHttp를 주로 같이 활용합니다.
> 그런데 정작 두 라이브러리의 차이점과 역할을 명확히 알고 계신가요?  
> 이번 글에서는 Retrofit과 OkHttp의 차이점부터 각각의 역할, 그리고 실무에서 어떻게 활용하는지까지 차근차근 정리해보겠습니다.

---

## 1. 기본 개념부터 이해하기

### Retrofit이란?
**Retrofit은 REST API를 위한 HTTP 클라이언트 라이브러리**입니다.  
Square에서 개발했으며, **인터페이스 기반으로 API를 정의**할 수 있어 코드 작성이 매우 간편합니다.

```kotlin
// Retrofit으로 API 인터페이스 정의
interface UserApi {
    @GET("users/{id}")
    suspend fun getUser(@Path("id") userId: Int): User
    
    @POST("users")
    suspend fun createUser(@Body user: User): Response<User>
}
```

### OkHttp란?
**OkHttp는 HTTP 클라이언트 라이브러리**입니다.  
마찬가지로 Square에서 개발했으며, **실제 HTTP 요청과 응답을 처리**하는 저수준 라이브러리입니다.

```kotlin
// OkHttp로 직접 HTTP 요청
val client = OkHttpClient()
val request = Request.Builder()
    .url("https://api.example.com/users/1")
    .build()

val response = client.newCall(request).execute()
```

---

## 2. 핵심 차이점 정리

| 구분 | Retrofit | OkHttp |
|:---|:---|:---|
| **역할** | API 인터페이스 정의 및 호출 | HTTP 클라이언트 구현 |
| **추상화 레벨** | 고수준 (인터페이스 기반) | 저수준 (직접 HTTP 처리) |
| **코드 작성** | 어노테이션으로 간단하게 | Request/Response 객체 직접 생성 |
| **JSON 변환** | 자동 변환 (Converter 사용) | 수동 변환 필요 |
| **에러 처리** | 자동 예외 처리 | 수동 처리 필요 |

---

## 3. 실제 코드로 비교해보기

### 같은 기능을 Retrofit과 OkHttp로 구현하면?

**목표**: 사용자 정보를 가져오는 API 호출

#### Retrofit 방식
```kotlin
// 1. 인터페이스 정의
interface UserApi {
    @GET("users/{id}")
    suspend fun getUser(@Path("id") userId: Int): User
}

// 2. Retrofit 설정
val retrofit = Retrofit.Builder()
    .baseUrl("https://api.example.com/")
    .addConverterFactory(GsonConverterFactory.create())
    .build()

val userApi = retrofit.create(UserApi::class.java)

// 3. 사용
class UserRepository {
    suspend fun getUser(userId: Int): Result<User> {
        return try {
            val user = userApi.getUser(userId)
            Result.success(user)
        } catch (e: HttpException) {
            Result.failure(e)
        }
    }
}
```

#### OkHttp 방식
```kotlin
class UserRepository {
    private val client = OkHttpClient()
    private val gson = Gson()
    
    suspend fun getUser(userId: Int): Result<User> {
        return withContext(Dispatchers.IO) {
            try {
                // 1. Request 생성
                val request = Request.Builder()
                    .url("https://api.example.com/users/$userId")
                    .get()
                    .build()
                
                // 2. HTTP 요청 실행
                val response = client.newCall(request).execute()
                
                // 3. 응답 처리
                if (response.isSuccessful) {
                    val json = response.body?.string() ?: ""
                    val user = gson.fromJson(json, User::class.java)
                    Result.success(user)
                } else {
                    Result.failure(HttpException(response.code))
                }
            } catch (e: Exception) {
                Result.failure(e)
            }
        }
    }
}
```

**결과**: Retrofit이 훨씬 간결하고 직관적입니다.

---

## 4. 그럼 왜 둘 다 사용할까?

### 실제로는 함께 사용됩니다!

```kotlin
// Retrofit은 내부적으로 OkHttp를 사용합니다
val okHttpClient = OkHttpClient.Builder()
    .addInterceptor(AuthInterceptor()) // 인증 헤더 자동 추가
    .addInterceptor(LoggingInterceptor()) // 요청/응답 로깅
    .build()

val retrofit = Retrofit.Builder()
    .baseUrl("https://api.example.com/")
    .client(okHttpClient) // OkHttp 클라이언트 설정
    .addConverterFactory(GsonConverterFactory.create())
    .build()
```

### 역할 분담
- **Retrofit**: API 인터페이스 정의, 타입 안전성, 자동 변환
- **OkHttp**: 실제 HTTP 통신, Interceptor, 캐싱, 연결 관리

---

## 5. Interceptor 활용 예시

OkHttp의 강력한 기능 중 하나인 Interceptor를 살펴보겠습니다.

```kotlin
// 인증 토큰 자동 추가 Interceptor
class AuthInterceptor(
    private val tokenProvider: () -> String?
) : Interceptor {
    
    override fun intercept(chain: Interceptor.Chain): Response {
        val originalRequest = chain.request()
        val token = tokenProvider()
        
        val newRequest = if (token != null) {
            originalRequest.newBuilder()
                .addHeader("Authorization", "Bearer $token")
                .build()
        } else {
            originalRequest
        }
        
        return chain.proceed(newRequest)
    }
}

// 로깅 Interceptor
class LoggingInterceptor : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()
        
        // 요청 로깅
        Log.d("Network", "Request: ${request.url}")
        Log.d("Network", "Headers: ${request.headers}")
        
        val response = chain.proceed(request)
        
        // 응답 로깅
        Log.d("Network", "Response: ${response.code}")
        
        return response
    }
}
```

---

## 6. 언제 무엇을 사용해야 할까?

### Retrofit을 사용하는 경우
- REST API 통신이 주 목적
- 타입 안전성이 중요
- 코드 간결성을 원함
- JSON 자동 변환이 필요
- 대부분의 안드로이드 앱에서 권장

### OkHttp만 사용하는 경우
- 매우 특수한 HTTP 요청이 필요
- Retrofit이 지원하지 않는 기능 사용
- 저수준 제어가 필요한 경우
- WebSocket 통신 (Retrofit은 WebSocket 미지원)

---

