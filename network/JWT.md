# JWT(Json Web Token)란?

> JWT는 웹 애플리케이션에서 인증과 권한 부여를 위해 사용되는 토큰 기반 인증 방식 중 하나입니다.   
> 특히 모바일, 프론트엔드 단독 애플리케이션에서 자주 사용되며, 서버는 Stateless한 구조를 유지할 수 있게 해줍니다.  

## 1. JWT란?

**JWT(Json Web Token)** 는 JSON 포맷을 사용하여 사용자 인증 정보를 안전하게 전송하기 위한 **토큰 기반 인증 방식**입니다.  
주로 사용자 인증, 권한 부여, 정보 전달 등에 사용됩니다.

JWT는 **Base64로 인코딩된 문자열 3가지**로 구성됩니다:

```
<Header>.<Payload>.<Signature>
```

예시:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.  
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6Ik9wUGEiLCJpYXQiOjE1MTYyMzkwMjJ9.  
TJVA95OrM7E2cBab30RMHrHDcEfxjoYZgeFONFh7HgQ  
```

## 2. 구성 요소

### 2-1. Header

* 어떤 알고리즘으로 서명했는지 명시

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### 2-2. Payload

* 실제 유저 정보와 권한 정보가 담기는 부분 (Claim)

```json
{
  "sub": "1234567890",
  "name": "짱구",
  "admin": true,
  "exp": 1719268432
}
```

### 2-3. Signature

* 헤더와 페이로드를 인코딩하고, 비밀 키로 서명

```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

## 3. 왜 사용하는가?

* **Stateless**: 세션처럼 서버에 상태를 저장할 필요가 없음
* **성능**: 서버가 토큰만 검증하면 되므로 빠름 (DB 조회 불필요)
* **다양한 플랫폼 호환**: 모바일, 웹, 서버 모두에서 사용 가능

## 4. 주의할 점

* **Payload는 암호화되지 않음**: Base64는 단순 인코딩이므로 정보 노출 가능 → 민감한 정보 저장 금지
* **만료 시간(exp) 필수**: 탈취된 토큰이 계속 유효하면 보안에 치명적
* **토큰 재발급 로직 필수**: Refresh Token 등을 활용해 만료 대응

## 5. JWT 사용 예시 (Android)

```kotlin
val token = "eyJhbGciOiJIUz..."
val request = originalRequest.newBuilder()
    .addHeader("Authorization", "Bearer $token")
    .build()
```

> 인터셉터에서 토큰을 자동으로 붙여주는 방식으로 인증을 처리할 수 있습니다.

## 참고자료

* [JWT 공식 사이트](https://jwt.io/introduction)
* [Auth0 - JWT Handbook](https://auth0.com/learn/json-web-tokens/)
* [Android Developers - Authentication](https://developer.android.com/training/id-auth)
