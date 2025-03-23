# Service Locator

> 이 글은 **Service Locator 패턴에 대해 학습한 내용을 정리한 문서**입니다.  
> DI(Dependency Injection)와의 개념적 차이를 이해하는 데 역할을 하며,  
> 안드로이드 환경에서 실제로 활용되는 방식(Koin 등)과도 관련이 있어 정리해보았습니다.

## 1. Service Locator란?
- Service Locator는 객체 간 의존 관계를 해결하는 디자인 패턴 중 하나로,   
애플리케이션에서 필요한 객체를 **중앙 집중화된 레지스트리(Central Registry)**로부터 가져오는 방식입니다.   
- 이 레지스트리는 필요한 객체를 생성하거나 반환하며, 객체를 사용하는 클래스는 이를 직접 호출하여 의존 객체를 얻습니다.

## 2. 동작 원리
- **의존성 등록**: 필요한 객체를 생성하고 Service Locator에 저장
- **의존성 요청**: 클래스가 Service Locator에 의존성을 요청하면 해당 객체를 반환

Service Locator는 단순한 전역 변수처럼 동작하지만,  
의존성 생성을 중앙에서 처리함으로써 코드의 결합도를 줄여주는 장점이 있습니다.

## 3. 장점과 단점
| 구분 | 내용 |
|------|------|
| **장점** | - 중앙화된 의존성 관리: ServiceLocator만 수정하면 전체 반영 가능 <br> - 구현의 유연성: 의존성을 다른 구현체로 교체 쉬움 <br> - 간단한 설정: 전역에서 객체를 단일 지점에서 관리 <br> - 보일러플레이트 코드가 적음 <br> - 소규모 프로젝트에서 빠르게 적용 가능 <br> - 구조가 단순하여 구현이 쉬움 |
| **단점** | - 테스트 어려움: 의존성이 숨겨져 있어 Mock 주입이 어려움 <br> - 명시적인 의존성 부재: 클래스 외부에서 어떤 의존성이 필요한지 알기 어려움 <br> - DIP, ISP 위반: 상위 모듈이 하위 구현체에 직접 접근 <br> - 객체 생명주기 관리 어려움: 의도치 않은 공유 객체가 생길 수 있음 |


## 4. Android에서의 활용
안드로이드에서는 시스템이 Activity나 Fragment를 생성하므로, 생성자 주입이 어렵습니다.  
이러한 상황에서 Koin과 같은 프레임워크는 편리함을 제공하지만, 실제로는 Service Locator 패턴에 가깝게 동작하게 됩니다.

예:
```kotlin
class MainActivity : AppCompatActivity() {
    private val viewModel: MainViewModel by viewModel()
}
```

`by viewModel()`은 내부적으로 중앙 레지스트리에서 ViewModel을 가져오므로,  
이 방식은 getter를 통해 객체를 요청하는 Service Locator의 접근과 동일합니다.

## 5. ServiceLocator 예제
```kotlin
object ServiceLocator {
    fun getEngine(): Engine = Engine()
}

class Car {
    private val engine = ServiceLocator.getEngine()

    fun start() {
        engine.start()
    }
}

fun main(args: Array<String>) {
    val car = Car()
    car.start()
}
```

복잡한 객체가 많아질수록 아래와 같은 제네릭 기반 방식이 사용됩니다:

```kotlin
object ServiceLocator {
    private val services = mutableMapOf<String, Any>()

    fun <T> registerService(key: String, service: T) {
        services[key] = service as Any
    }

    fun <T> getService(key: String): T {
        return services[key] as T
    }
}

class UserRepository {
    private val database: Database = ServiceLocator.getService("database")
    private val networkClient: NetworkClient = ServiceLocator.getService("network")

    fun getUserData() {
        // use database and networkClient
    }
}
```

또는 Kotlin의 `lazy`를 활용한 방식도 자주 쓰입니다:

```kotlin
object ServiceLocator {
    val database by lazy { Database() }
    val networkClient by lazy { NetworkClient() }
}

class DataRepository {
    private val database = ServiceLocator.database
    private val networkClient = ServiceLocator.networkClient

    fun loadData() {
        // use database and networkClient
    }
}
```

## 6. 언제 사용하면 좋은가?
- **소규모 프로젝트**나 **프로토타입**처럼 빠르게 기능을 만들고 싶은 경우
- **DI 프레임워크 도입이 과한 경우** (예: 객체 수가 적거나, 생명주기 관리가 단순한 경우)
- **학습용, 데모용** 앱에서 간단히 구조를 짤 때
- **코드 구조를 간단히 유지하고자 할 때**

## 7. 결론
- Service Locator는 간단하지만 테스트와 구조적 유연성 측면에서 불리하며, 종종 anti-pattern으로 간주됩니다
- 규모가 있는 앱에서는 Hilt나 Dagger처럼 명시적인 DI 프레임워크가 더 적합합니다
- Service Locator는 의존성을 "찾아오는" 방식이고, DI는 외부에서 "주입받는" 방식이다

## 참고 자료
- [매쉬업 기술블로그](https://mashup-android.vercel.app/mashup-10th/blackjin/post3/dependency-injection-service-location/)
- [기술 면접 질문 서비스 로케이터(Service Locator) 패턴](https://velog.io/@sunjoo9912/기술-면접-질문-서비스-로케이터Service-Locator-패턴이란-무엇인가)
- [패스트 캠퍼스 클린아키텍처](https://fastcampus.co.kr/dev_online_clean)
