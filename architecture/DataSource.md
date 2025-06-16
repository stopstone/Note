# DataSource란?

> 데이터 소스를 명확하게 정의하면, 계층 분리와 테스트가 쉬워집니다.  
> 이 글에서는 DataSource의 정의, 사용하는 이유, 계층 구조 내 위치, 그리고 예시 코드를 설명합니다.  

---

## 1. DataSource란?

DataSource는 말 그대로 "**데이터의 출처(source)**"를 추상화한 계층입니다.  
보통 Repository 패턴에서, Repository가 여러 DataSource들을 조합하여 데이터를 제공합니다.  

예를 들어,

* 로컬에 저장된 데이터 → LocalDataSource
* 원격 서버에서 받아오는 데이터 → RemoteDataSource

이렇게 나눌 수 있습니다.

---

## 2. 왜 분리해서 쓰는가?

### 관심사의 분리 (Separation of Concerns)

* 로컬과 원격의 구현을 분리하면 코드가 더 명확해집니다.

### 테스트 용이성

* 테스트 시에는 FakeRemoteDataSource, FakeLocalDataSource를 주입할 수 있습니다.

### 유지보수 편의

* 예를 들어, 서버 API가 바뀌면 RemoteDataSource만 수정하면 됩니다.

---

## 3. 계층 구조 내 위치와 역할

Clean Architecture 기준으로 보면:

```
Presentation
    ↓
UseCase / Interactor
    ↓
Repository Interface (Domain)
    ↓
Repository Implementation (Data Layer)
    ↓
DataSource (Remote / Local)
```

* DataSource는 실제로 데이터를 가져오거나 저장하는 계층입니다.
* Repository는 DataSource들을 조합하여 데이터 흐름을 결정합니다.

---

## 4. Repository와 DataSource의 차이

| 구분  | Repository                        | DataSource                               |
| --- | --------------------------------- | ---------------------------------------- |
| 목적  | 여러 데이터 소스를 통합해 도메인 계층에 일관된 API 제공 | 특정 출처(로컬/원격 등)에서 데이터를 가져오거나 저장           |
| 위치  | Domain과 Data 계층 사이, UseCase에서 호출됨 | Data 계층 내부에 존재하며 Repository에서 호출함        |
| 역할  | 데이터 소스 선택 및 조합, 도메인에 비즈니스 룰 반영    | 네트워크, DB 등 구체적이고 실제적인 데이터 처리 수행          |
| 테스트 | 인터페이스를 통해 Mock Repository로 대체 가능  | Fake DataSource로 대체 가능                   |
| 예시  | 캐싱 여부에 따라 로컬/원격 선택                | Retrofit, Room, DataStore 등 API/DB 호출 구현 |

Repository는 외부에 데이터를 제공할 때 "어디서 데이터를 가져올지"를 고민하고 결정하는 중간 관리자이며, DataSource는 실제 데이터를 가져오는 실행자에 가깝습니다.

---

## 5. 예시 코드 (LocalDataSource & RemoteDataSource)

```kotlin
// Domain Layer
interface UserRepository {
    suspend fun getUser(id: String): User
}
```

```kotlin
// DataSource Interface
interface UserRemoteDataSource {
    suspend fun fetchUser(id: String): User
}

interface UserLocalDataSource {
    suspend fun getUser(id: String): User?
    suspend fun saveUser(user: User)
}
```

```kotlin
// Repository Implementation
class UserRepositoryImpl(
    private val remoteDataSource: UserRemoteDataSource,
    private val localDataSource: UserLocalDataSource
) : UserRepository {
    override suspend fun getUser(id: String): User {
        val cached = localDataSource.getUser(id)
        return if (cached != null) {
            cached
        } else {
            val remoteUser = remoteDataSource.fetchUser(id)
            localDataSource.saveUser(remoteUser)
            remoteUser
        }
    }
}
```
