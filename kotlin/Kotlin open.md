# Kotlin open

> Kotlin의 `open` 키워드는 **상속과 오버라이딩을 허용**하는 핵심 키워드입니다.  
> Java와 달리 Kotlin은 기본적으로 **final**이므로, 상속받으려면 명시적으로 `open`을 선언해야 합니다.  

---

## 1. Java와 Kotlin의 차이점

### Java의 기본 동작
```java
// Java - 기본적으로 상속 가능
public class Animal {
    public void makeSound() {
        System.out.println("동물이 소리를 냅니다");
    }
}

public class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("멍멍!");
    }
}
```

### Kotlin의 기본 동작
```kotlin
// 컴파일 에러 - 기본적으로 final
class Animal {
    fun makeSound() {
        println("동물이 소리를 냅니다")
    }
}

class Dog : Animal() { // This type is final, so it cannot be inherited from
    override fun makeSound() {
        println("멍멍!")
    }
}
```

### Kotlin에서 올바른 사용법
```kotlin
// open 키워드로 상속 허용
open class Animal {
    open fun makeSound() {
        println("동물이 소리를 냅니다")
    }
}

class Dog : Animal() {
    override fun makeSound() {
        println("멍멍!")
    }
}
```

---

## 2. 왜 Kotlin은 기본적으로 final일까?

### 1. 보안성 향상
```kotlin
// 의도하지 않은 상속을 방지
class DatabaseConfig {
    val password = "secret123"
    
    fun connect() {
        // 중요한 연결 로직
    }
}

// open이 없으면 상속 불가 -> 보안 향상
```

### 2. 성능 최적화
```kotlin
// final 클래스/메서드는 컴파일러 최적화 가능
class Calculator {
    fun add(a: Int, b: Int): Int = a + b // 인라인 최적화 가능
}
```

### 3. 의도 명확화
```kotlin
// 개발자의 의도를 명확히 표현
open class BaseViewModel        // 상속받아서 사용하세요
class UserRepository           // 상속하지 말고 그대로 사용하세요
```

---

## 3. Open 키워드 사용법

### 클래스에 적용
```kotlin
// 상속 가능한 클래스
open class BaseActivity {
    open fun onCreate() {
        // 기본 구현
    }
}

class MainActivity : BaseActivity() {
    override fun onCreate() {
        super.onCreate()
        // 추가 구현
    }
}
```

### 메서드에 적용
```kotlin
open class Animal {
    // 오버라이드 가능한 메서드
    open fun makeSound() = "소리"
    
    // 오버라이드 불가능한 메서드 (기본 final)
    fun breathe() = "숨쉬기"
}

class Dog : Animal() {
    override fun makeSound() = "멍멍!"
    
    // breathe()는 오버라이드 불가
}
```

### 프로퍼티에 적용
```kotlin
open class Vehicle {
    open val maxSpeed: Int = 100
    open var color: String = "white"
}

class Car : Vehicle() {
    override val maxSpeed: Int = 200
    override var color: String = "red"
}
```

---

## 4. 실무에서의 활용 사례

### 1. BaseViewModel 패턴
```kotlin
// 공통 기능을 가진 부모 클래스
open class BaseViewModel : ViewModel() {
    protected val _isLoading = MutableLiveData<Boolean>()
    val isLoading: LiveData<Boolean> = _isLoading
    
    open fun showLoading() {
        _isLoading.value = true
    }
    
    open fun hideLoading() {
        _isLoading.value = false
    }
}

// 특정 기능을 가진 자식 클래스
class UserViewModel : BaseViewModel() {
    override fun showLoading() {
        super.showLoading()
        // 추가 로딩 로직
    }
}
```

### 2. Repository 추상 클래스
```kotlin
// 공통 로직을 가진 추상 Repository
abstract class BaseRepository<T> {
    open suspend fun save(item: T): Result<T> {
        return try {
            val saved = performSave(item)
            Result.success(saved)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
    
    // 하위 클래스에서 구현
    abstract suspend fun performSave(item: T): T
}

class UserRepository : BaseRepository<User>() {
    override suspend fun performSave(item: User): User {
        // User 저장 로직
        return userDao.insert(item)
    }
}
```

---

## 5. 언제 사용해야 할까?

**1. Framework/Library 개발**
```kotlin
// 라이브러리에서 확장 가능한 기본 클래스 제공
open class BaseAdapter<T> : RecyclerView.Adapter<RecyclerView.ViewHolder>() {
    open fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int, item: T) {
        // 기본 바인딩 로직
    }
}
```

**2. 공통 기능을 가진 부모 클래스**
```kotlin
open class BaseFragment : Fragment() {
    open fun showProgressBar() {
        progressBar.visibility = View.VISIBLE
    }
    
    open fun hideProgressBar() {
        progressBar.visibility = View.GONE
    }
}
```

### Open을 사용하지 말아야 하는 경우

**1. 데이터 클래스**
```kotlin
// 데이터 클래스는 final로 유지
data class User(val id: Long, val name: String)

// 불변 객체는 상속하지 않음
class ApiResponse(val success: Boolean, val data: Any?)
```

**2. 유틸리티 클래스**
```kotlin
// 정적 메서드만 있는 클래스
object StringUtils {
    fun isValidEmail(email: String): Boolean {
        // 이메일 검증 로직
    }
}
```

**3. 비즈니스 로직 클래스**
```kotlin
// 핵심 비즈니스 로직은 상속보다 조합 선호
class PaymentProcessor {
    fun processPayment(amount: Double): PaymentResult {
        // 결제 처리 로직
    }
}
```

## 참고 자료

- [Kotlin 공식 문서 - Classes and Inheritance](https://kotlinlang.org/docs/classes.html)
- [Effective Java - Item 19: Design and document for inheritance or else prohibit it](https://www.oreilly.com/library/view/effective-java-3rd/9780134686097/)
