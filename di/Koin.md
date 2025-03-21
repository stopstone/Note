# Koin

Android에서 의존성 주입 라이브러리를 Hilt만 적용해보다가 KMP에 대해 알게 되어  
Koin 라이브러리에 대해 알아보았습니다.  

이 문서에서는 Koin이란 무엇이며, Android에서 어떻게 활용하는지에 대해 설명합니다.

## Koin이란?

Koin은 Kotlin 기반의 의존성 주입(Dependency Injection, DI) 프레임워크입니다.  
이 프레임워크는 Android, 백엔드, 그리고 Kotlin 멀티플랫폼(KMP) 프로젝트들을 지원하며,  
개발자가 의존성을 관리할 수 있도록 돕습니다.   
Koin은 Kotlin을 잘 활용하여 외부 설정 없이 단순하고 직관적인 방법으로 DI를 구현할 수 있게 해줍니다.

### Koin의 특징
- **Kotlin DSL 사용**: Kotlin의 DSL(Domain-Specific Language)을 활용하여 직관적인 방식으로 의존성을 정의할 수 있습니다.
- **Kotlin 친화적**: Kotlin 문법과 원활하게 통합됩니다.
- **멀티플랫폼 지원**: Android, iOS, 백엔드 간 의존성을 공유할 수 있습니다.
- **성능**: 순수 Kotlin으로 작성되어 성능에 영향을 미치지 않습니다.
- **간단한 설정**: 추가적인 플러그인이나 복잡한 설정 없이 바로 사용할 수 있습니다.

## Koin을 알기 전에 알면 좋은 개념

- **싱글톤(Singleton)**: 하나의 객체만 생성하여 애플리케이션 전체에서 공유
- **DI**: 의존성 주입에 대한 기본적인 개념
- **Service Locator**: 의존성 주입에 대한 기본적인 개념
 
## Android에서 Koin 사용하기

### 1. Application 설정

`startKoin()` 함수를 사용하여 Koin을 설정하고, 애플리케이션의 필요 모듈을 등록합니다. 예를 들어:

```kotlin
class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        startKoin {
            androidContext(this@MyApp)
            modules(appModule)
        }
    }
}
```

### 2. Module 설정

Koin에서는 `module` 키워드를 사용해 의존성을 정의하고 관리합니다. 예를 들어, 데이터 레포지토리와 ViewModel을 등록하는 방법은 다음과 같습니다.

```kotlin
val dataModule = module {
    single<MovieRepository> { MovieRepositoryImpl() }
}

val viewModelModule = module {
    viewModel { MovieViewModel(get()) }
}

startKoin {
    androidContext(this@MyApp)
    modules(dataModule, viewModelModule)
}
```

### 3. DI 받기

Koin은 `by inject()` 또는 `by viewModel()`을 통해 의존성을 쉽게 주입받을 수 있습니다. 예시:

```kotlin
class MainActivity : AppCompatActivity() {
    val movieRepository: MovieRepository by inject()
    val movieViewModel: MovieViewModel by viewModel()
}
```

### 4. single, factory, viewModel

- `single`: 싱글톤 객체로 인스턴스를 생성합니다. 애플리케이션 전체에서 동일한 객체를 사용합니다.
- `factory`: 매번 새로운 객체를 생성합니다.
- `viewModel`: ViewModel을 주입받을 때 사용합니다.

### 5. 테스트에서 Koin 사용하기

Koin을 테스트에서 사용하려면 `KoinTest`와 `KoinTestRule`을 활용하여 모듈을 주입받고 테스트를 진행할 수 있습니다.

```kotlin
class MovieRepositoryTest : KoinTest {
    @get:Rule
    val koinTestRule = KoinTestRule.create {
        modules(dataModule)
    }

    val movieRepository: MovieRepository by inject()

    @Test
    fun testMovieRepository() {
        assertNotNull(movieRepository)
    }
}
```

## Koin은 Service Locator 패턴인가?

- 과거에는 `get()`과 같은 API가 런타임에 의존성을 조회하는 방식이라 **Service Locator 패턴**과 유사하다는 의견이 많았음.
- 그러나 Koin은 **DI(Dependency Injection) 프레임워크**이며, 최근 버전에서는 **Constructor Injection**을 적극 지원하여 DI의 특성을 강화함.

## Koin은 둘 다 지원한다

- Koin 공식 문서에서는 Koin이 **DI 패턴을 지원하면서도 Service Locator 패턴을 선택적으로 사용할 수 있음**을 명시하고 있음.
- 초기 Koin은 DI 원칙을 완전히 따르지는 않았지만, 업데이트를 거치며 **Constructor Injection**과 같은 DI 원칙을 따르도록 개선됨.
- **Koin 2.0**: Constructor Injection을 적극적으로 지원
- **Koin 3.0**: DI 프레임워크로서의 입장을 더욱 강화

## Koin의 장점

- **간단한 설정**: 별도의 설정 없이 Koin을 바로 사용할 수 있습니다.
- **경량화된 DI**: 코드 생성이나 리플렉션을 사용하지 않아 성능에 부담을 주지 않습니다.
- **멀티플랫폼 지원**: Android, iOS, Backend에서 동일한 DI 패턴을 적용할 수 있습니다.


## 참고 자료

- [Koin 공식 문서](https://insert-koin.io/)
- [Koin 사용하기](https://gus0000123.medium.com/koin-library-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0-1%ED%8E%B8-%EC%84%B8%ED%8C%85%ED%8E%B8-830977c80d40)
- [Koin과 Android DI (Spoqa 기술블로그)](https://spoqa.github.io/2020/11/02/android-dependency-injection-with-koin.html)
- [Koin에서 Hilt로 마이그레이션 (BankSalad 기술블로그)](https://blog.banksalad.com/tech/migrate-from-koin-to-hilt/)
- [패스트캠퍼스 클린 아키텍처 강의](https://fastcampus.co.kr/dev_online_clean)

## 관련 자료 (DI)

- [DI 개념 정리](https://github.com/stopstone/Note/blob/main/di/DI.md)
- [Hilt 개념 정리](https://github.com/stopstone/Note/blob/main/di/Hilt.md)

