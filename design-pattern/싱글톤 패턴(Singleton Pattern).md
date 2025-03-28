# 싱글톤 패턴 (Singleton Pattern)
> 개발하시면서 자주 마주하게 되는 싱글톤 패턴에 대해 알아보겠습니다.  
> 싱글톤 패턴은 객체가 단 하나만 생성되도록 보장하며, 앱 전역에서 공유되는 상태나 리소스를 관리할 때 유용하게 사용됩니다.  
> 자바에서는 `static` 키워드로 정적 멤버를 정의하지만, 코틀린에서는 `object`나 `companion object`를 통해 유사한 기능을 제공합니다.  
> 이 중 `object`는 진짜 싱글톤 객체를 생성하며, JVM 기준으로 **최초 접근 시점에 초기화**되므로 lazy하게 동작합니다.  
> 이 글은 **안드로이드 개발자의 시각에서 Kotlin을 사용한 싱글톤 패턴 구현과 활용법**을 정리한 문서입니다.


## 개념

- 객체의 인스턴스가 하나만 존재하도록 보장하는 디자인 패턴입니다.
- 이 인스턴스는 전역적으로 접근 가능합니다.
    - **장점:** 메모리 사용량 감소, 코드 간의 일관성 유지
    - **단점:** 글로벌 상태가 생기기 때문에 **테스트 간 의존성이 생기고, Mock 처리도 어려워지는 등 테스트가 어려울 수 있으며**, 의존성 문제 발생 가능성도 있습니다.

---

## 구현 방법

### **Eager Initialization - object 선언 사용**

```kotlin
object Settings {
    val volume = 50
}
```

- `object` 키워드를 사용하여 싱글턴 객체를 선언합니다.
- 객체 안에 프로퍼티와 함수를 정의할 수 있습니다.
- `object` 선언은 클래스 정의와 동일하게 동작하지만, **실제로는 JVM의 lazy semantics에 따라 해당 객체가 최초로 사용될 때 초기화**됩니다.
    - **장점:** 별도의 생성 메서드가 필요하지 않습니다.
    - **단점:** 코드가 간결한 대신 유연성이 떨어질 수 있습니다.

---

### **Companion Object Initialization**

```kotlin
class Singleton private constructor() {
    init {
        println("Singleton Init!!")
    }
    companion object {
        init {
            println("Singleton Object Init!!")
        }

        fun doSomething() {
            println("Doing something...")
        }
    }
}
```

- `companion object`는 클래스 내부에 정적 멤버를 선언할 수 있도록 도와주는 Kotlin만의 문법입니다.
- 클래스가 처음 로딩될 때 함께 초기화됩니다.
    - **장점:** 클래스명으로 직접 정적 메서드에 접근 가능하여 Java의 static block과 유사하게 사용할 수 있습니다.
    - **단점:** 객체가 생성되지 않아도 클래스 접근만으로 초기화되기 때문에 불필요한 초기화가 발생할 수 있습니다.

---

### **Lazy Initialization - by lazy 사용**

```kotlin
class Singleton private constructor() {
   companion object {
         private val INSTANCE: Singleton by lazy {
             Singleton()
         }            
         fun getInstance() = INSTANCE
     }
}
```

- 인스턴스를 실제로 필요할 때 초기화하는 방식입니다.
- `by lazy`는 기본적으로 thread-safe합니다.
    - **장점:** 리소스 낭비를 줄일 수 있으며, 스레드 간 동기화도 보장됩니다.
    - **단점:** 초기화는 늦출 수 있지만, 초기화된 필드에 접근할 때마다 약간의 비용이 추가될 수 있습니다.

---

### **Double-Checked Locking**

```kotlin
class Singleton private constructor() {
    companion object {
        @Volatile
        private var instance: Singleton? = null

        fun getInstance(): Singleton {
            return instance ?: synchronized(this) {
                instance ?: Singleton().also { instance = it }
            }
        }
    }
}
```

- 멀티 쓰레드 환경에서도 안전한 지연 초기화를 위한 고전적인 방식입니다.
- 최초 초기화 시에만 synchronized 블록을 거치고, 이후에는 바로 반환하여 성능 저하를 방지합니다.
    - `@Volatile`은 메인 메모리에 항상 최신 값을 반영하도록 보장합니다.

---

### **Bill Pugh 방식 (Lazy Holder Idiom)**

```kotlin
class Singleton private constructor() {
    companion object {
        private object LazyHolder {
            val INSTANCE = Singleton()
        }

        val instance: Singleton
            get() = LazyHolder.INSTANCE
    }

    fun doSomething() {
        println("Doing something...")
    }
}
```

- 내부 클래스의 로딩 시점을 활용한 방식입니다.
- JVM의 클래스 로더 메커니즘 덕분에 **스레드 세이프하면서도 지연 초기화가 가능합니다.**
    - **장점:** 성능과 안정성을 동시에 확보할 수 있습니다.
    - **단점:** 구조를 이해하기에 약간 복잡할 수 있습니다.

---

### **Enum 방식**

```kotlin
enum class Singleton {
    INSTANCE;

    fun doSomething() {
        println("Doing something...")
    }
}
```

- Kotlin에서도 Enum을 활용하여 싱글턴을 구현할 수 있습니다.
- JVM에서는 Enum이 인스턴스를 하나만 보장하므로 안전합니다.
    - **장점:** 직렬화(Serialization) 환경에서도 안전하고, 간결합니다.
    - **단점:** Enum은 클래스 상속이 불가능하므로 유연성이 떨어집니다.

---

## 언제 어떤 싱글턴 방식이 적합할까?

- **가장 간단하고 직관적인 구현이 필요하다면 `object` 선언**을 사용하는 것이 좋습니다. 테스트나 유연성이 중요한 상황이 아니라면 대부분의 케이스에서 적합합니다.
- **지연 초기화가 필요하고, 사용 빈도가 낮은 리소스를 관리해야 한다면 `by lazy` 방식**이 효과적입니다.
- **멀티쓰레드 환경에서 더 세밀한 초기화 제어가 필요하다면 Double-Checked Locking 또는 Bill Pugh 방식**이 적합합니다.
- **직렬화, 리플렉션, 멀티쓰레드 환경 등 모든 면에서 강력한 보장을 원한다면 Enum 기반 싱글턴**이 가장 안정적입니다.
- **정적 메서드만 필요하고, 객체 자체의 상태를 관리할 필요가 없다면 Companion Object로 충분**합니다.

---

## 싱글톤 패턴 사용 시 주의할 점

- **테스트의 어려움:** 싱글톤은 전역 상태를 가지므로, 테스트 간 의존성이 생기고 테스트 순서에 따라 결과가 달라질 수 있습니다. 단위 테스트에서는 특히 Mocking이 어려워질 수 있습니다.
- **숨겨진 의존성:** 싱글톤을 남용하면 의존성 주입 없이 전역 상태에 접근하게 되어 코드의 응집도가 낮아지고 유지보수가 어려워질 수 있습니다.
- **메모리 누수 위험:** 앱 컨텍스트를 잘못 보관하거나, 리스너/콜백 등의 레퍼런스를 싱글톤 내부에 계속 보관하면 GC 대상이 되지 않아 누수가 발생할 수 있습니다.
- **멀티모듈 환경 주의:** 여러 모듈에서 같은 싱글톤 인스턴스를 의도하지 않게 따로 만들 수 있으므로 주의하셔야 합니다.
- **스코프 관리:** Android에서는 Activity/Fragment 생명주기와 관련된 객체를 싱글톤으로 잘못 관리하면 메모리 누수나 불필요한 객체 유지가 발생할 수 있습니다.

> 따라서 싱글톤은 반드시 "진짜 전역적으로 관리할 필요가 있는 객체인지"를 고민한 뒤, 적절한 방식으로 신중하게 사용하시는 것이 중요합니다.

