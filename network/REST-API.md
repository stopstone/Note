## 1. 안드로이드 개발자가 REST API를 왜 알아야 할까요?

REST API를 알아보기 앞서, 안드로이드 개발자가 왜 REST API에 대해 알아야 할까요?  
안드로이드 개발자는 서버와 통신해야 할 일이 많습니다.  
현대 앱은 대부분의 데이터를 서버에서 받아와 사용자에게 보여줍니다.  
이때 서버와 통신하는 방식이 대부분 REST API를 기반으로 합니다.

> `REST API`는 서버와 클라이언트의 **약속**이라고 볼 수 있습니다. 

REST API로 다음과 같은 이점을 얻을 수 있습니다:  

**1. 데이터 통신**: 앱이 서버에 요청을 보내고, 응답을 받아서 화면에 표시하거나 처리합니다.  
**2. 표준화된 방식**: REST API를 사용하면 서버와 클라이언트 간 커뮤니케이션이 표준화되어 협업이 수월해집니다.  
**3. API 문서 이해**: Swagger 같은 백엔드 개발자가 작성한 API 문서를 해석할 때 REST에 대한 이해가 필요합니다.  
**4. 에러 핸들링**: HTTP 상태 코드(200, 400, 401, 500 등)를 해석하고 상황에 맞게 대응할 수 있습니다.  


  REST API에 대해 전반적으로 이야기하고, Status Code, HTTP Method와 같이 통신을 하면서 알야아 할 것을을 소개하고,  
  안드로이드 개발을 하며 서버와의 통신에 있어 더 이해도 높은 개발을 할 수 있도록 하는 것을 목표로 합니다.
  
## 2. REST API는 무엇일까요?

**REST(Representational State Transfer)** 는 웹의 장점을 최대한 활용할 수 있도록 설계된 아키텍처 스타일입니다.  
REST API는 이 REST 아키텍처 스타일을 따르는 인터페이스를 의미합니다.  
**리소스(자원)를 URL로 표현하고, HTTP 메서드(GET, POST, PUT, DELETE 등)를 통해 리소스를 조작**하는 방식입니다.

- **리소스(Resource)** : 데이터 자체 (예: 사용자, 게시글)
- **URI(Uniform Resource Identifier)** : 리소스에 접근하기 위한 경로
- **HTTP Method** : 리소스에 수행할 작업 (CRUD)

**예시:**

| 작업 | HTTP 메서드 | URL 예시 | 의미 |
|:---|:---|:---|:---|
| 조회 | GET | `/users/1` | ID가 1인 사용자 조회 |
| 생성 | POST | `/users` | 새 사용자 생성 |
| 수정 | PUT | `/users/1` | ID가 1인 사용자 정보 수정 |
| 삭제 | DELETE | `/users/1` | ID가 1인 사용자 삭제 |

## 3. REST API의 아키텍처 조건

REST 아키텍처는 6가지 제약 조건을 충족해야 합니다. 이를 만족할 때 `RESTful`하다고 이야기합니다.

- **클라이언트-서버 구조(Client-Server)**: 클라이언트와 서버를 분리하여 서로 독립적으로 개발 및 배포할 수 있습니다.
- **무상태성(Stateless)**: 서버는 요청 간 클라이언트 상태를 저장하지 않습니다. 모든 요청은 독립적이어야 합니다.
- **캐시 처리 가능(Cacheable)**: 서버의 응답은 명시적으로 캐시 가능하거나 불가능함을 나타내야 합니다.
- **계층화된 시스템(Layered System)**: 클라이언트는 서버를 직접 알 필요 없이 중간 서버를 통해 통신할 수 있습니다.
- **일관된 인터페이스(Uniform Interface)**: API 설계는 일관성과 표준을 유지해야 합니다.
- **Code on Demand(선택적)**: 필요에 따라 서버가 클라이언트로 실행 가능한 코드를 전송할 수 있습니다. (실제 사용 빈도는 낮음)


## 4. HTTP Method 종류와 특징

HTTP Method는 리소스에 대해 어떤 동작을 수행할 것인지를 나타내는 역할을 합니다.  
각각의 메서드가 의미하는 바를 정확히 이해하고 사용해야 합니다.

- **GET**: 리소스를 조회할 때 사용합니다. 서버에 어떠한 변경도 일으키지 않습니다.
- **POST**: 새로운 리소스를 생성할 때 사용합니다. 서버에 데이터가 추가됩니다.
- **PUT**: 기존 리소스를 수정할 때 사용합니다. 전체 데이터를 업데이트할 때 주로 사용합니다.
- **PATCH**: 기존 리소스의 일부만 수정할 때 사용합니다.
- **DELETE**: 리소스를 삭제할 때 사용합니다.


## 5. HTTP Status Code 종류

HTTP Status Code는 클라이언트의 요청에 대한 서버의 응답 결과를 나타냅니다. 주로 사용하는 코드들은 다음과 같습니다.
StatusCode와 설명, 안드로이드 개발을 하면서 요청시 일어날 예시를 들었습니다.

| StatusCode | 설명 | 예시 |
|:---|:---|:---|
| 200 OK | 요청이 성공적으로 완료 | 사용자의 프로필 정보를 조회했을 때 |
| 201 Created | 리소스가 성공적으로 생성 | 새로운 회원가입 요청이 성공했을 때 |
| 400 Bad Request | 잘못된 요청. | 필수 입력값이 누락된 폼 제출 |
| 401 Unauthorized | 인증이 필요하거나 실패 | 로그인하지 않고 사용자 정보 요청을 했을 때 |
| 403 Forbidden | 서버가 요청을 이해했지만 인가되지 않아 거부 | 일반 사용자가 관리 페이지에 접근하려고 할 때 |
| 404 Not Found | 요청한 리소스를 찾을 수 없음 | 존재하지 않는 게시글 ID를 조회했을 때 |
| 500 Internal Server Error | 서버 내부 오류로 요청을 수행할 수 없음 | 서버 코드에 예외가 발생했을 때 |

## 6. RESTful와 REST의 차이

**REST**  
아키텍처 스타일 자체를 의미합니다. 자원을 URI로 표현하고, HTTP 메서드로 자원에 대한 행위를 명시합니다.

**RESTful API**
REST 아키텍처 스타일을 충실히 따르는 API를 의미합니다. 즉, 설계 방식이 REST의 규칙을 잘 지키고 있다면 RESTful하다고 표현합니다.  
RESTful API는 표준화된 설계를 통해 유지보수가 용이하고, 다양한 클라이언트가 일관성 있게 통신할 수 있다는 장점이 있습니다.


## 7. API 통신 예시 (Retrofit2)

Retrofit은 안드로이드에서 REST API 통신을 쉽게 해주는 라이브러리입니다.
Retrofit 공식 문서를 참고한 API 통신 예시는 다음과 같습니다.

**통신 코드**  

```kotlin
interface GitHubService {
    @GET("users/{user}/repos")
    suspend fun listRepos(@Path("user") user: String): List<Repo>
}

val retrofit = Retrofit.Builder()
    .baseUrl("https://api.github.com/")
    .addConverterFactory(GsonConverterFactory.create())
    .build()

val service = retrofit.create(GitHubService::class.java)
```

- `listRepos` 메서드는 특정 사용자의 저장소 목록을 가져오는 API입니다.
- `@GET` 어노테이션을 통해 HTTP GET 요청을 보내고, 결과는 List<Repo> 형태로 받습니다.
- `baseUrl`은 GitHub API 기본 주소를 설정합니다.
- `GsonConverterFactory`를 추가하여 JSON 응답을 Kotlin 객체로 변환할 수 있도록 합니다.

**에러 핸들링**  
```kotlin
suspend fun fetchRepos(user: String): Result<List<Repo>> {
    return try {
        val repos = service.listRepos(user)
        Result.success(repos)
    } catch (e: HttpException) {
        val message = when (e.code()) {
            401 -> "인증 실패: 로그인 정보가 유효하지 않습니다. 다시 로그인해 주세요."
            403 -> "권한 없음: 요청한 리소스에 접근할 권한이 없습니다."
            404 -> "리소스를 찾을 수 없습니다: 요청한 데이터가 존재하지 않습니다."
            500 -> "서버 오류: 문제가 발생했습니다. 잠시 후 다시 시도해 주세요."
            else -> "알 수 없는 오류: 서버로부터 ${e.code()} 응답을 받았습니다."
        }
        Result.failure(Exception(message))
    } catch (e: IOException) {
        Result.failure(Exception("네트워크 오류: 인터넷 연결을 확인해 주세요."))
    } catch (e: Exception) {
        Result.failure(e)
    }
}

다음과 같이 서버에서 내려온 에코드를 분기하여 처리할 수 있습니다.

```

## 8. JSON 데이터 포맷

REST API는 대부분 JSON 포맷을 사용해 데이터를 주고받습니다. (XML로 주고 받는 경우도 있어요)

```json
[
  {
    "id": 1296269,
    "name": "stopstone",
    "full_name": "dolmeng-stopstone",
    "private": false
  }
]
```

JSON은 키-값 쌍으로 이루어진 간결한 데이터 표현 방식으로, 서버와 클라이언트 간 데이터 교환에 적합합니다.

> 직렬화(Serialization): 객체를 JSON 형태로 변환하여 네트워크로 전송하거나 저장하는 과정입니다.
> 
> 역직렬화(Deserialization): JSON 데이터를 다시 객체 형태로 변환하여 앱 내에서 사용하는 과정입니다.


## 9. 서버 인증(Authentication)과 인가(Authorization)

- **Authentication(인증)**: 사용자가 누구인지 확인하는 과정입니다. (예: 로그인)
- **Authorization(인가)**: 인증된 사용자가 특정 리소스에 접근할 수 있는 권한이 있는지를 판단하는 과정입니다. (예: 관리자 계정만 특정 페이지 접근 허용)

보통 OAuth 2.0, JWT 토큰 방식을 사용하여 인증과 인가를 수행합니다.

JWT 토큰 방식에 대해 설명하자면,

**JWT 토큰 사용 흐름:**

1. 사용자가 로그인 정보를 서버에 전송합니다.
2. 서버는 로그인 정보를 확인하고 JWT 토큰을 발급합니다.
3. 클라이언트는 이후 요청마다 JWT 토큰을 Authorization 헤더에 포함하여 서버에 전송합니다.
4. 서버는 토큰을 검증하고 요청을 처리합니다.

