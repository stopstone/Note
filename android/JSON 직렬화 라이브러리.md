#  JSON 직렬화 라이브러리 비교하기

> 모바일 앱이 서버와 통신할 때 주고받는 데이터는 대부분 JSON입니다.  
> 따라서 "어떤 라이브러리로 JSON을 다룰 것인가"는 앱의 성능과 안정성과 큰 연관이 있습니다.  
> JSON으로 받아온 데이터를 앱에서 객체로 활용하기 위해서는 변환이 필요합니다. 이 과정을 `직렬화` 라고 합니다.  
>
> 이 문서는 직렬화를 위한 라이브러리를 소개하고, 각각을 비교하여 본인의 프로젝트에 맞게 선택하는 것을 목표로 합니다.

## 1. 직렬화 주요 라이브러리 소개

- **Gson**: Google에서 개발한 `Java 기반` JSON 직렬화/역직렬화 라이브러리입니다.  
- **Moshi**: Square에서 만든 현대적인 JSON 라이브러리입니다. `Kotlin 친화성`과 성능 최적화에 초점을 맞췄습니다.  
- **Serialization**: JetBrains가 공식 지원하는 Kotlin 전용 직렬화 라이브러리로, `타입 안전성`과 `경량화`가 강점입니다.  

## 2. 주요 차이점 비교

| 항목 | Gson | Moshi | Serialization |
|:---|:---|:---|:---|
| 개발사 | Google | Square | JetBrains |
| 출시 시기 | 2008년 | 2015년 | 2019년 |
| Kotlin 호환성 | 제한적 (Java 기반) | Kotlin 친화적 | Kotlin 전용 |
| 직렬화 방식 | 리플렉션 기반 | Codegen 기반 | Codegen 기반 |
| 성능 | 소규모 안정적, 대규모 비효율적 | 빠르고 메모리 효율적 | 매우 빠르고 경량화됨 |
| 유지보수 상태 | 유지보수 모드 | 활발한 업데이트 진행 중 | 활발한 업데이트 진행 중 |
| 커스터마이징 | TypeAdapter 필요 | JsonAdapter 사용 | 커스터마이징 가능하지만 복잡할 수 있음 |

**리플렉션 기반**  
런타임에 객체의 구조(Class, Field, Method 등)를 분석하여 데이터를 읽거나 쓰는 방식입니다.  
`직렬화/역직렬화 시점`에 매번 클래스 정보를 탐색하기 때문에, 속도가 느리고 메모리 사용량이 많아질 수 있습니다.

**Codegen 기반**  
`컴파일 시점`에 필요한 변환 코드를 미리 생성해두고, 런타임에는 생성된 코드를 직접 호출하는 방식입니다.  
런타임 리플렉션이 필요 없기 때문에 변환 속도가 빠르고, 메모리 효율이 뛰어납니다.

## 3. 사용 예시 비교

### Gson 예시

```kotlin
data class User(
    @SerializedName("nickname") val name: String,
    @SerializedName("yearsOld") val age: Int
)

val gson = Gson()
val json = gson.toJson(User("신짱구", 5))
val user = gson.fromJson(json, User::class.java)
```

### Moshi 예시

```kotlin
@JsonClass(generateAdapter = true)
data class User(
    @Json(name = "nickname") val name: String,
    @Json(name = "yearsOld") val age: Int
)

val moshi = Moshi.Builder()
    .add(KotlinJsonAdapterFactory())
    .build()
val adapter = moshi.adapter(User::class)
val json = adapter.toJson(User("신짱구", 5))
val user = adapter.fromJson(json)
```

`@JsonClass(generateAdapter = true)`는 Moshi에서 JSON 직렬화를 위해 컴파일 타임에 변환용 어댑터 코드를 자동 생성하도록 하는 어노테이션입니다.  
이로 인해, 런타임 리플렉션 없이 빠르고 메모리 효율적인 변환이 가능합니다.
Codegen을 활성화하지 않으면 Moshi가 런타임 리플렉션을 사용해 성능과 메모리 효율이 떨어집니다.

### Serialization 예시

```kotlin
@Serializable
data class User(
    @SerialName("nickname") val name: String,
    @SerialName("yearsOld") val age: Int
)

val json = Json.encodeToString(User("신짱구", 5))
val user = Json.decodeFromString<User>(json)
```

## 4. 무엇을 사용해야 할까요?

**Gson을 선택할 경우**:
  - 레거시 Java 프로젝트를 유지해야 할 때
  - 이미 Gson 기반 코드가 많아 교체 비용이 클 때

**Moshi를 선택할 경우**:
  - Kotlin 기반 신규 프로젝트를 만들 때
  - 성능과 메모리 최적화를 고려할 때
  - 코드 생성(Codegen)을 적극 활용하고 싶을 때

**Serialization을 선택할 경우**:
  - Kotlin Multiplatform(KMP) 프로젝트를 진행할 때
  - 가장 빠른 직렬화/역직렬화 성능을 원할 때
  - 순수 Kotlin 코드베이스를 유지하고 싶을 때

## 참고자료
[IMQA JSON 라이브러리 성능 비교](https://blog.imqa.io/json-moshi/)  
[Kotlin 공식문서 - serialization](https://kotlinlang.org/docs/serialization.html)  
