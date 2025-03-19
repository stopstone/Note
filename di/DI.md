# DI

종속 항목 삽입(Dependency Injection, DI)은 코드의 의존성을 외부에서 주입하여 관리하는 기법입니다.   
이를 통해 다음과 같은 이점을 얻을 수 있습니다.  

- **클래스 재사용 가능 및 종속 항목 분리**:   
종속 항목 구현을 쉽게 교체할 수 있습니다. 컨트롤 반전으로 인해 코드 재사용이 개선되었으며 클래스가 더 이상 종속 항목 생성 방법을 제어하지 않지만 대신 모든 구성에서 작동합니다.

- **리팩터링 편의성**:   
종속 항목은 API 노출 영역의 검증 가능한 요소가 되므로 구현 세부정보로 숨겨지지 않고 객체 생성 시간 또는 컴파일 시간에 확인할 수 있습니다.

- **테스트 편의성**:   
클래스는 종속 항목을 관리하지 않으므로 테스트 시 다양한 구현을 전달하여 다양한 모든 사례를 테스트할 수 있습니다.  

해당 문서에서는 DI의 개념과 종류에 대한 설명을 하는 것을 목표로 합니다.  

---

## 1. 종속 항목 삽입이란?

클래스는 다른 클래스의 기능을 사용해야 하는 경우가 많습니다.  
예를 들어 자동차(`Car`) 클래스는 엔진(`Engine`)이 필요할 수 있습니다.  

클래스가 필요한 객체를 얻는 방법에는 다음 세 가지가 있습니다.  

1. **클래스가 직접 필요한 객체를 생성하는 방법**
2. **객체를 외부에서 가져오는 방법 (예: Android의 `getSystemService()`)**
3. **객체를 외부에서 주입받는 방법** → 종속 항목 삽입(DI)

세 번째 방법이 DI입니다. 
DI를 사용하면 클래스 내부에서 직접 객체를 생성하지 않고, 생성자를 통해 외부에서 객체를 주입받습니다.

---

## 2. DI 없이 직접 종속 항목을 생성하는 코드
![image](https://github.com/user-attachments/assets/b3a99cc9-111e-47c5-9115-9425236190e3)

```kotlin
class Engine {
    fun start() {
        println("Engine started")
    }
}

class Car {
    private val engine = Engine()
    
    fun start() {
        engine.start()
    }
}

fun main() {
    val car = Car()
    car.start()
}
```

위 코드에서는 `Car` 클래스가 직접 `Engine` 객체를 생성하기 때문에 두 클래스가 강하게 결합되어 있습니다.   
이렇게 되면 다음과 같은 문제가 발생합니다.    

- `Car`는 `Engine`의 구체적인 구현에 의존하므로, 다른 종류의 엔진(예: 전기 엔진)으로 변경하기 어렵습니다.  
- `Engine`이 복잡한 종속성을 가진 경우, `Car`를 테스트할 때 `Engine`의 모든 동작을 고려해야 합니다.

---

## 3. DI를 사용한 코드
![image](https://github.com/user-attachments/assets/6672f367-674f-4dd8-b66e-afba958db2f9)

DI를 사용하면 객체를 생성자의 매개변수로 전달받습니다.

```kotlin
class Engine {
    fun start() {
        println("Engine started")
    }
}

class Car(private val engine: Engine) {
    fun start() {
        engine.start()
    }
}

fun main() {
    val engine = Engine()
    val car = Car(engine)
    car.start()
}
```

이 방식의 장점은 다음과 같습니다.

- `Car`는 `Engine`의 특정 구현을 알 필요가 없으므로, 쉽게 대체할 수 있습니다.
- 테스트할 때 `Engine` 대신 `FakeEngine` 같은 테스트용 객체를 주입할 수 있습니다.

---

## 4. Android에서 DI 적용 방법

### 4.1 생성자 주입(Constructor Injection)

대부분의 경우, 생성자 주입이 가장 좋은 방법입니다.

```kotlin
class Car(private val engine: Engine) {
    fun start() {
        engine.start()
    }
}
```

그러나 `Activity`, `Fragment`와 같은 Android 프레임워크 클래스는 시스템이 직접 인스턴스화하므로 생성자 주입을 사용할 수 없습니다.

### 4.2 필드 주입(Field Injection)

이 경우, `lateinit var` 키워드를 사용하여 필드 주입을 사용할 수 있습니다.

```kotlin
class Car {
    lateinit var engine: Engine

    fun start() {
        engine.start()
    }
}
```

이러한 DI를 관리하기 위해 Dagger와 같은 라이브러리를 사용할 수 있습니다.

---

## 5. DI 대안: 서비스 로케이터

DI의 대안으로 **서비스 로케이터(Service Locator)** 패턴이 있습니다.  
서비스 로케이터 패턴은 종속 항목을 중앙 저장소(전역 객체)에서 관리하고, 필요할 때 해당 객체를 제공하는 방식입니다.   
이는 종속 항목을 직접 생성하는 것이 아니라, 미리 정의된 저장소에서 검색하여 가져오는 방식으로 동작합니다.

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
```

### DI vs. 서비스 로케이터

| 비교 항목          | 종속 항목 삽입(DI) | 서비스 로케이터 |
|------------------|----------------|----------------|
| 유지보수 용이성 | 높음           | 낮음          |
| 테스트 용이성   | 높음           | 낮음          |
| 코드 재사용성   | 높음           | 낮음          |

---


## 6. 참고 자료

- [Android 공식 문서: 종속 항목 삽입](https://developer.android.com/training/dependency-injection?hl=ko)

## 7. 관련 자료
Hilt:   
Koin:  
