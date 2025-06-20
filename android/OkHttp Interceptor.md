# OkHttp Interceptor

> OkHttp의 Interceptor는 네트워크 요청과 응답을 **중앙에서 제어할 수 있는 도구**입니다.    
인증, 로깅, 에러 처리, 캐싱 등 다양한 목적에 맞게 활용할 수 있으며, 실무에서도 Retrofit과 함께 필수로 사용됩니다.    
이 글에서는 Interceptor의 개념부터 종류, 구현 방법, 실무 예제까지 차근히 정리합니다.  

## 1. Interceptor란?

* OkHttp의 Interceptor는 **요청(Request)과 응답(Response)을 중간에서 가로채고 가공할 수 있는 인터페이스**입니다.
* 요청이 서버로 나가기 전, 또는 응답이 앱 코드로 들어오기 전에 개입하여 로직을 추가할 수 있습니다.
* 보통 다음과 같은 용도로 사용됩니다:

  * 공통 헤더 추가 (Authorization 등)
  * 요청/응답 로깅
  * 에러 코드 공통 처리
  * 캐시 설정

## 2. 종류: Application vs Network Interceptor

| 종류                      | 설명                                                                                   |
| ----------------------- | ------------------------------------------------------------------------------------ |
| `addInterceptor`        | Application Interceptor. 캐시나 리다이렉션 처리 이전에 개입합니다. 항상 호출되며, 요청과 응답 모두 제어 가능합니다.        |
| `addNetworkInterceptor` | Network Interceptor. 네트워크 통신 직전에 호출됩니다. 캐시된 응답에는 호출되지 않으며, TLS 같은 로우레벨 정보 접근이 가능합니다. |

### 특징 요약

* **Application Interceptor**는 앱 레벨 공통 처리 (헤더, 로깅 등)에 적합합니다.
* **Network Interceptor**는 응답 본문을 직접 가공하거나, 실제 네트워크 처리 직전에 개입이 필요할 때 사용됩니다.

## 3. 기본 구현 예제

```kotlin
class MyInterceptor : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val request = chain.request()

        val newRequest = request.newBuilder()
            .header("Authorization", "Bearer abc123")
            .build()

        val response = chain.proceed(newRequest)

        if (response.code == 401) {
            // 예: 토큰 만료 처리
        }

        return response
    }
}
```

## 4. 실전 예제

### 1) 인증 헤더 자동 추가

```kotlin
class AuthInterceptor(private val tokenProvider: () -> String) : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val req = chain.request().newBuilder()
            .header("Authorization", "Bearer ${tokenProvider()}")
            .build()
        return chain.proceed(req)
    }
}
```

### 2) 에러 코드 처리

```kotlin
class ErrorInterceptor : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val resp = chain.proceed(chain.request())
        when (resp.code) {
            401 -> { /* 로그아웃 처리 */ }
            500 -> { /* 서버 오류 핸들링 */ }
        }
        return resp
    }
}
```

### 3) 캐시 제어

```kotlin
class CacheInterceptor : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val resp = chain.proceed(chain.request())
        return resp.newBuilder()
            .header("Cache-Control", "max-age=${60 * 5}")
            .build()
    }
}
```

## 5. 사용 시 주의할 점

* **Interceptor의 순서가 중요**합니다. 요청은 등록된 순서대로, 응답은 반대로 처리됩니다.

```kotlin
val client = OkHttpClient.Builder()
    .addInterceptor(AuthInterceptor(::getToken))
    .addInterceptor(ErrorInterceptor())
    .addNetworkInterceptor(CacheInterceptor())
    .build()
```

* `chain.proceed()`는 반드시 한 번만 호출해야 하며, 이를 생략하면 네트워크 호출이 일어나지 않습니다.

## 6. 로깅을 위한 공식 Interceptor

```kotlin
val logging = HttpLoggingInterceptor().apply {
    level = HttpLoggingInterceptor.Level.BODY
}
val client = OkHttpClient.Builder()
    .addInterceptor(logging)
    .build()
```

* `HttpLoggingInterceptor`는 요청과 응답의 바디를 출력하는 데 유용합니다.
