# API

> 프로그래밍을 하면서 "API"라는 용어를 자주 접하게 됩니다.  
> 이번 글에서는 API의 기본 개념에 대해 알아보겠습니다.

---

## 1. API란 무엇일까요?

**API(Application Programming Interface)**는 **애플리케이션 간의 소통을 위한 인터페이스**입니다.

쉽게 말해서, API는 **"서로 다른 소프트웨어가 대화할 수 있게 해주는 번역기"**라고 생각하면 됩니다.

### 1.1 일상생활로 비유해보기

**레스토랑의 웨이터**를 생각해보세요:

- **고객(클라이언트)**: 음식을 주문하고 싶어함
- **웨이터(API)**: 고객의 주문을 주방에 전달하고, 완성된 음식을 고객에게 가져다줌
- **주방(서버)**: 실제로 음식을 만드는 곳

고객은 주방에 직접 가서 음식을 만들 수 없습니다. 웨이터라는 **인터페이스**를 통해 주문하고 결과를 받는 것이죠.

### 1.2 프로그래밍에서의 API

```kotlin
// 안드로이드에서 간단한 예시
val sharedPreferences = getSharedPreferences("MyPrefs", Context.MODE_PRIVATE)
val editor = sharedPreferences.edit()
editor.putString("username", "신짱구")
editor.apply() // API를 통해 데이터 저장
```

여기서 `SharedPreferences`의 `edit()`, `putString()`, `apply()` 메서드들이 **Android API**입니다.

---

## 2. API의 종류

### 2.1 운영체제 API (System API)

**운영체제가 제공하는 기능을 사용할 수 있게 해주는 API**

```kotlin
// Android System API 예시
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // Android API를 사용한 권한 요청
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA) 
            != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(this, 
                arrayOf(Manifest.permission.CAMERA), 100)
        }
        
        // Android API를 사용한 네트워크 상태 확인
        val connectivityManager = getSystemService(Context.CONNECTIVITY_SERVICE) as ConnectivityManager
        val networkInfo = connectivityManager.activeNetworkInfo
        val isConnected = networkInfo?.isConnected ?: false
    }
}
```

### 2.2 라이브러리 API (Library API)

**외부 라이브러리가 제공하는 기능을 사용할 수 있게 해주는 API**

```kotlin
// Retrofit API 예시
interface ApiService {
    @GET("users/{id}")
    suspend fun getUser(@Path("id") userId: Int): User
}

// Glide API 예시
Glide.with(context)
    .load("https://example.com/image.jpg")
    .into(imageView)
```

### 2.3 Web API (웹 API)

**웹 서버가 제공하는 데이터나 기능을 사용할 수 있게 해주는 API**

```kotlin
// Web API 호출 예시
class UserRepository {
    private val apiService = RetrofitClient.apiService
    
    suspend fun fetchUser(userId: Int): Result<User> {
        return try {
            val user = apiService.getUser(userId)
            Result.success(user)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}
```

---

## 3. API의 구성 요소

### 3.1 엔드포인트 (Endpoint)

**API가 제공하는 특정 기능에 접근할 수 있는 URL**

```kotlin
// 예시: 사용자 관련 API 엔드포인트들
interface UserApi {
    @GET("api/users")           // 모든 사용자 조회
    suspend fun getUsers(): List<User>
    
    @GET("api/users/{id}")      // 특정 사용자 조회
    suspend fun getUser(@Path("id") id: Int): User
    
    @POST("api/users")          // 새 사용자 생성
    suspend fun createUser(@Body user: User): User
    
    @PUT("api/users/{id}")      // 사용자 정보 수정
    suspend fun updateUser(@Path("id") id: Int, @Body user: User): User
    
    @DELETE("api/users/{id}")   // 사용자 삭제
    suspend fun deleteUser(@Path("id") id: Int): Response<Unit>
}
```

### 3.2 요청 (Request)

**클라이언트가 서버에 보내는 데이터**

```kotlin
// POST 요청 예시
data class CreateUserRequest(
    val name: String,
    val email: String,
    val age: Int
)

// 요청 보내기
val newUser = CreateUserRequest(
    name = "홍길동",
    email = "hong@example.com",
    age = 25
)
val createdUser = apiService.createUser(newUser)
```

### 3.3 응답 (Response)

**서버가 클라이언트에게 보내는 데이터**

```kotlin
// 응답 데이터 모델
data class User(
    val id: Int,
    val name: String,
    val email: String,
    val age: Int,
    val createdAt: String
)

// 응답 처리
suspend fun fetchUser(userId: Int): User? {
    return try {
        val response = apiService.getUser(userId)
        response // User 객체 반환
    } catch (e: HttpException) {
        when (e.code()) {
            404 -> {
                Log.e("API", "사용자를 찾을 수 없습니다")
                null
            }
            500 -> {
                Log.e("API", "서버 오류가 발생했습니다")
                null
            }
            else -> {
                Log.e("API", "알 수 없는 오류: ${e.code()}")
                null
            }
        }
    }
}
```

---
