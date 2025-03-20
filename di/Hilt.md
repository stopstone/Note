# Hilt

## 1. Hilt란?
Hilt는 **Android용 종속 항목 삽입(DI, Dependency Injection) 라이브러리**로, Dagger를 기반으로 한 Google의 공식 DI 솔루션입니다. 

**Hilt는 어노테이션 기반 DI 프레임워크로, `@Inject`, `@Module`, `@Provides` 등을 활용하여 의존성을 주입하는 방식을 사용합니다.**

### 1.1 Dagger와 Hilt의 차이점
- **컴파일 타임 DI**: Hilt는 Dagger와 동일하게 컴파일 타임에 의존성 그래프를 구축하여 런타임 성능을 최적화합니다.
- **보일러플레이트 감소**: Hilt는 Dagger보다 더욱 간결한 코드 작성이 가능하며, Android 컴포넌트와의 통합이 원활합니다.
- **의존성 주입 방식**: `@Module`, `@Provides`, `@Inject` 등을 활용하여 객체를 생성하고 의존성을 관리합니다.

Hilt를 사용하면 Android 프로젝트에서 **일관된 방식으로 DI를 설정하고 관리할 수 있습니다**.

---

## 2. Hilt 설정 (Gradle Setup)
### 프로젝트 수준(`build.gradle.kts` or `build.gradle`)
```kotlin
plugins {
    id("com.google.dagger.hilt.android") version "2.51.1" apply false
}
```

### 모듈 수준(`app/build.gradle.kts` or `app/build.gradle`)
```kotlin
plugins {
    id("com.google.devtools.ksp")
    id("com.google.dagger.hilt.android")
}

dependencies {
    implementation("com.google.dagger:hilt-android:2.51.1")
    ksp("com.google.dagger:hilt-android-compiler:2.51.1")
}

```

---

## 3. Hilt 기본 사용법

### 3.1 `@HiltAndroidApp`
Hilt를 사용하려면 `@HiltAndroidApp`을 Application 클래스에 추가해야 합니다. 
```kotlin
@HiltAndroidApp
class MyApplication : Application()
```
> `@HiltAndroidApp`은 **Hilt의 DI 시스템을 초기화**하는 역할을 합니다.

---

## 4. Hilt 어노테이션

### 4.1 `@Inject`
- 생성자 또는 필드에 사용하여 **Hilt가 객체를 주입할 수 있도록 설정**
```kotlin
class AnalyticsService @Inject constructor()
```

### 4.2 `@AndroidEntryPoint`
- Activity, Fragment, View, Service, BroadcastReceiver에서 **의존성 주입을 받을 경우 필수**
```kotlin
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {
    @Inject lateinit var analyticsService: AnalyticsService
}
```

### 4.3 `@HiltViewModel`
- Jetpack ViewModel에서 Hilt를 사용할 때 필요
```kotlin
@HiltViewModel
class MyViewModel @Inject constructor(
    private val analyticsService: AnalyticsService
) : ViewModel()
```

### 4.4 `@Module` + `@InstallIn`
- Hilt 모듈을 정의하여 **직접 제공할 객체를 설정**
```kotlin
@Module
@InstallIn(SingletonComponent::class)
object AppModule {
    @Provides
    @Singleton
    fun provideAnalyticsService(): AnalyticsService {
        return AnalyticsService()
    }
}
```

### 4.5 `@Qualifier`
- 같은 타입의 객체를 여러 개 제공할 경우 구분을 위해 사용
- 예를 들어, `OkHttpClient`가 다른 용도로 두 개 이상 필요할 경우 사용됩니다.
- 일반적으로 네트워크 요청을 할 때, **인증이 필요한 요청과 일반 요청을 분리**하고 싶을 때 활용됩니다.
- `@Qualifier`를 사용하면 **특정한 목적에 맞는 객체를 정확히 주입**할 수 있습니다.
```kotlin
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class AuthInterceptor

@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    @AuthInterceptor
    @Provides
    fun provideAuthInterceptorOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder().build()
    }
}
```

### 4.6 `@Inject` vs `@Binds` vs `@Provides`
| 어노테이션 | 용도 | 특징 | 예제 |
|------------|------|------|------|
| `@Inject` | 생성자 주입 | 객체 인스턴스 직접 생성 가능, 의존성 간단할 때 사용 | `class Example @Inject constructor(...)` |
| `@Binds` | 인터페이스 타입 주입 | 추상 메서드를 사용해 구현체를 지정할 때 사용 | `abstract fun bindRepository(...): Repository` |
| `@Provides` | 생성자 주입이 불가능한 객체 제공 | 외부 라이브러리, 빌더 패턴 등을 사용할 때 활용 | `fun provideApiService(): ApiService {...}` |

---

## 5. Hilt의 수명주기 컴포넌트
Hilt는 **Android의 생명주기와 연계된 DI 컴포넌트**를 제공합니다.

| Component                 | Scope                    | 생성 시점         | 소멸 시점        |
|---------------------------|--------------------------|--------------------|-----------------|
| `SingletonComponent`      | `@Singleton`             | Application 시작  | Application 종료|
| `ActivityRetainedComponent` | `@ActivityRetainedScoped` | Activity 시작 | Activity 종료 (config 유지) |
| `ActivityComponent`       | `@ActivityScoped`       | Activity 시작      | Activity 종료   |
| `FragmentComponent`       | `@FragmentScoped`       | Fragment 시작      | Fragment 종료   |
| `ViewModelComponent`      | `@ViewModelScoped`      | ViewModel 생성     | ViewModel 종료  |
| `ServiceComponent`        | `@ServiceScoped`        | Service 시작       | Service 종료    |

**예제: `@Singleton`을 사용한 DI**
```kotlin
@Module
@InstallIn(SingletonComponent::class)
object RepositoryModule {
    @Singleton
    @Provides
    fun provideRepository(): UserRepository {
        return UserRepository()
    }
}
```

---

## 6. 참고 자료
- [Hilt 공식 문서](https://developer.android.com/training/dependency-injection/hilt-android?hl=ko)
- [Dagger 공식 문서](https://dagger.dev/dev-guide/)
- [패스트캠퍼스 클린 아키텍처 강의](https://fastcampus.co.kr/dev_online_clean)

## 7. 관련 자료
- [Hilt를 사용한 Android DI 블로그](https://hyperconnect.github.io/2020/07/28/android-dagger-hilt.html)
- [Hilt와 Dagger-Android 비교](https://proandroiddev.com/exploring-dagger-hilt-and-what-s-main-differences-from-dagger-android-84a46e775b1a)

