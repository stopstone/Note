# Repository Pattern
> 안드로이드를 개발할 때 계층 구조를 나누려고 하면 Repository Pattern은 필수적으로 활용되는 패턴입니다.  
> Repository Pattern은 안드로이드 MVVM / MVI 아키텍처에서 데이터 흐름을 명확하게 하고, 모듈 간의 결합도를 낮추며 테스트 가능성을 높여줍니다.  
> 특히 클린 아키텍처나 Hilt와 함께 사용하면 더 깔끔하고 유지보수가 쉬운 앱 구조를 설계할 수 있습니다.  
> 이번 문서에서는 Repository Pattern이 무엇이고, 왜 사용하는지, 클린아키텍쳐 관점에서의 Repository Pattern을 살펴봅니다.  

## 1. Repository Pattern이란?

Repository Pattern은 **앱의 데이터 소스를 중앙에서 관리하고, 일관된 API로 외부에 제공**하는 디자인 패턴입니다.   
로컬(Room), 원격(Retrofit 등) 데이터를 직접 View나 ViewModel에서 접근하지 않고,  
**중간에 Repository를 두어 간접적으로 접근**합니다.  

이를 통해 다음과 같은 이점을 얻을 수 있습니다:
- 데이터 소스를 추상화하여, ViewModel이나 UI 레이어가 구체적인 구현을 알 필요 없음
- 코드의 응집도 향상, 결합도 감소
- 테스트 용이성 증가
- 유지보수와 확장성 개선
![9e528301efd49aea_1440](https://github.com/user-attachments/assets/bbd4326e-bd06-45cc-b80c-fc6567f7c65e)
저장소는 데이터 소스(예: 영구 모델, 웹 서비스, 캐시) 간의 충돌을 해결하고 이 데이터의 변경사항을 중앙 집중화할 수 있습니다.
아래 다이어그램은 활동과 같은 앱 구성요소가 저장소를 통해 데이터 소스와 상호작용하는 방법을 보여줍니다.
![69021c8142d29198_1440](https://github.com/user-attachments/assets/8cd293e3-5c16-41e0-ab91-7374389a7b98)


---

## 2. Repository Pattern을 사용하는 이유
앱은 보통 다양한 데이터 소스(로컬 DB, 네트워크, 캐시 등)를 사용합니다. 이를 View나 ViewModel에서 직접 관리하면 코드가 복잡해지고, 유지보수가 어려워집니다.  
Repository Pattern은 이러한 문제를 해결하기 위해 관심사 분리(Separation of Concerns) 원칙에 따라 데이터 접근 로직을 별도 계층으로 분리합니다.

### 저장소의 역할

- 저장소는 여러 출처(Local, Remote 등)의 데이터를 통합하고, 필요한 데이터를 적절한 시점에 제공합니다.  
- 일반적인 앱에서 저장소는 네트워크에서 데이터를 가져올지, 로컬에 캐시된 데이터를 사용할지를 판단합니다.  
- 이렇게 하면 ViewModel은 데이터의 출처를 신경 쓰지 않고 Repository만 통해 데이터를 받으면 됩니다.

### 저장소 사용의 이점
- **단일 진실의 원천(Single Source of Truth)**: 하나의 책임 있는 데이터 원천으로 데이터 충돌을 방지합니다.
- **데이터 소스 분리 및 캡슐화**: 로컬(Room), 원격(Retrofit 등) 데이터를 내부적으로 조합하여 제공합니다.
- **ViewModel과의 깔끔한 연결**: ViewModel에서는 Repository만 사용하므로, 데이터 흐름이 명확합니다.
- **테스트 용이성**: 인터페이스로 추상화되어 있어 mock/fake repository를 사용해 유닛 테스트가 쉬워집니다.

### 캐싱

캐시는 네트워크가 불안정하거나 연결이 끊겼을 때 임시 데이터 저장소로 작동할 수 있습니다.  
Android에서 캐싱 방법은 다음과 같습니다:

| 캐싱 기법 | 사용 |
|-----------|------|
| **Retrofit** | 네트워크 결과를 로컬에 저장. 간단한 요청에 적합 |
| **DataStore** | 키-값 저장에 적합. 앱 설정 등 |
| **내부 저장소 파일** | 파일 기반 데이터 저장. 앱 제거 시 삭제됨 |
| **Room (SQLite)** | 구조화된 데이터 저장에 적합한 공식 권장 방식 |

> 일반적으로 Room을 Repository와 함께 사용하여 오프라인 캐시를 구현합니다.

---

## 3. 안드로이드에서 Repository 패턴 적용 예시

Android 공식 Codelab 예제를 기반으로 Repository 패턴을 구현한 예시입니다.

### Repository 클래스
```kotlin
class VideosRepository(private val database: VideosDatabase) {

    val videos: LiveData<List<DevByteVideo>> = 
        Transformations.map(database.videoDao.getVideos()) {
            it.asDomainModel()
        }

    suspend fun refreshVideos() {
        withContext(Dispatchers.IO) {
            val playlist = DevByteNetwork.devbytes.getPlaylist()
            database.videoDao.insertAll(playlist.asDatabaseModel())
        }
    }
}
```

### ViewModel에서 Repository 사용
```kotlin
class DevByteViewModel(application: Application) : AndroidViewModel(application) {

    private val videosRepository = VideosRepository(getDatabase(application))

    val playlist = videosRepository.videos

    init {
        refreshDataFromRepository()
    }

    private fun refreshDataFromRepository() {
        viewModelScope.launch {
            try {
                videosRepository.refreshVideos()
            } catch (e: Exception) {
                // 예외 처리
            }
        }
    }
}
```

---

## 4. 클린 아키텍처에서의 Repository 역할

클린 아키텍처에서는 Repository를 **도메인 계층의 인터페이스로 정의하고**, **데이터 계층에서 이를 구현**합니다.  
이 구조 덕분에 비즈니스 로직은 데이터 접근 방법(Room, Retrofit 등)의 세부사항에 의존하지 않게 됩니다.  

### Repository 추상화 예시
```kotlin
// Domain Layer
interface VideoRepository {
    suspend fun getVideos(): List<Video>
}

// Data Layer
class VideoRepositoryImpl(
    private val localDataSource: LocalDataSource,
    private val remoteDataSource: RemoteDataSource
) : VideoRepository {
    override suspend fun getVideos(): List<Video> {
        val cached = localDataSource.getCachedVideos()
        return if (cached.isNotEmpty()) cached else remoteDataSource.fetchVideos()
    }
}
```

이처럼 인터페이스를 통해 **구현체를 유연하게 교체할 수 있는 구조**를 갖추게 되며,  
테스트 시 `FakeRepository`로도 쉽게 대체 가능합니다.

### 계층 구조 예시

![계층 구조](https://miro.medium.com/v2/resize:fit:1000/format:webp/0*IJ_FkHUGMfWQAC1K.png)

```
View → ViewModel → UseCase → Repository(Interface) → DataSource(구현체)
```

이러한 구조는 SRP, DIP 등 SOLID 원칙을 따르는 설계와도 잘 맞으며, 유지보수와 확장에 강한 구조를 만듭니다.

---

## Repository Pattern 면접 질문 & 답변

### 1. Repository Pattern의 주요 목적은 무엇인가요?

**답변:**  
Repository Pattern은 데이터 소스를 추상화하여 앱의 나머지 계층과 분리합니다.  
이를 통해 ViewModel이나 UseCase는 데이터가 어디서 왔는지 알 필요 없이 일관된 방식으로 데이터에 접근할 수 있으며, 코드의 모듈화, 테스트 용이성, 유지보수성이 향상됩니다.

---

### 2. MVVM/MVI 아키텍처에서 Repository는 어떤 역할을 하나요?

**답변:**  
MVVM이나 MVI에서 Repository는 ViewModel(StateHolder)과 DataSource(Local, Remote) 사이에서 데이터 흐름을 중재하는 역할을 합니다.  
Repository는 다양한 데이터 소스를 통합하여 UI 계층에 필요한 데이터를 제공합니다.    
이를 통해 관심사 분리와 단일 진실의 원천(Single Source of Truth)을 실현할 수 있습니다.  
단일 진실의 원천 : 하나의 데이터 소스만이 진짜(진실) 데이터를 제공해야 한다는 개념.  

---

### 3. 로컬 데이터와 원격 데이터를 Repository에서 어떻게 조합하여 사용할 수 있나요?

**답변:**  
보통 Repository 내부에서 캐싱 전략을 사용합니다. 예를 들어 로컬(Room) 데이터를 먼저 반환한 뒤, 원격(Retrofit) 데이터를 가져와 업데이트할 수 있습니다.  
또는 네트워크 상태에 따라 로컬 데이터만 제공하거나, 새 데이터를 받아 캐시에 저장하는 등의 전략을 사용할 수 있습니다.

---



## 참고 자료
- [Android 공식 문서: 앱 아키텍처 가이드](https://developer.android.com/jetpack/guide/data-layer)
- [Codelab: Repository 패턴](https://developer.android.com/codelabs/basic-android-kotlin-training-repository-pattern)

