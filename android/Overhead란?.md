# Overhead란?
> Overhead(오버헤드)는 **주요 작업을 수행하기 위해 필요한 추가적인 비용이나 자원**을 의미합니다.  
> 시스템이 실제 목표 작업 외에 추가로 소모하는 시간, 메모리, CPU 자원 등을 포함합니다.

## Overhead의 핵심 개념

### 1. 정의와 특징
- **정의**: 주요 작업 수행을 위해 필요한 부가적인 비용
- **특징**: 직접적인 결과물을 만들지 않지만 반드시 필요한 과정
- **목적**: 시스템의 안정성, 보안, 성능 등을 위한 추가 처리

### 2. Overhead의 종류
```
시간 오버헤드: 추가 처리 시간
메모리 오버헤드: 추가 메모리 사용량
CPU 오버헤드: 추가 연산 처리량
네트워크 오버헤드: 추가 통신 비용
```

## 언제 Overhead가 발생하는가?

### 1. 동기화 메커니즘 사용 시
```kotlin
class DataProcessor {
    private val mutex = Mutex() // 오버헤드: 락 관리
    private var data = 0
    
    suspend fun processData() {
        mutex.withLock { // 오버헤드: 락 획득/해제
            data++ // 실제 작업
        }
    }
}
```

### 2. 메모리 관리
```kotlin
class MemoryManager {
    private val objects = mutableListOf<Any>() // 오버헤드: 메모리 관리
    
    fun createObject(): Any {
        val obj = Any() // 실제 객체 생성
        objects.add(obj) // 오버헤드: 리스트 관리
        return obj
    }
    
    fun cleanup() {
        objects.clear() // 오버헤드: 가비지 컬렉션 트리거
    }
}
```

### 3. 네트워크 통신
```kotlin
class NetworkClient {
    suspend fun fetchData(url: String): String {
        // 오버헤드: 연결 설정, 헤더 처리
        val connection = establishConnection(url) // 오버헤드
        val headers = prepareHeaders() // 오버헤드
        
        val data = connection.get() // 실제 데이터 전송
        
        connection.close() // 오버헤드: 연결 해제
        return data
    }
}
```

## Android에서의 Overhead

### 1. ViewBinding 사용 시
```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding // 오버헤드: 바인딩 객체
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // 오버헤드: 바인딩 초기화
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        
        // 실제 UI 작업
        binding.textView.text = "Hello World"
    }
    
    override fun onDestroy() {
        super.onDestroy()
        // 오버헤드: 메모리 정리
        binding = null
    }
}
```

### 2. LiveData 사용 시
```kotlin
class UserViewModel : ViewModel() {
    private val _userData = MutableLiveData<User>() // 오버헤드: LiveData 관리
    val userData: LiveData<User> = _userData
    
    fun updateUser(user: User) {
        // 오버헤드: LiveData 업데이트 처리
        _userData.value = user // 실제 데이터 업데이트
    }
}
```

### 3. Room Database 사용 시
```kotlin
@Entity
data class User(
    @PrimaryKey val id: Int,
    val name: String
) // 오버헤드: 어노테이션 처리

@Dao
interface UserDao {
    @Query("SELECT * FROM user")
    suspend fun getAllUsers(): List<User> // 오버헤드: 쿼리 컴파일
}

@Database(entities = [User::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
} // 오버헤드: 데이터베이스 스키마 관리
```

## Overhead 최적화 방법

### 1. 불필요한 오버헤드 제거
```kotlin
// 비효율적: 매번 새로운 객체 생성
fun inefficientProcessing() {
    repeat(1000) {
        val processor = DataProcessor() // 오버헤드: 객체 생성
        processor.process()
    }
}

// 효율적: 객체 재사용
class EfficientProcessor {
    private val processor = DataProcessor() // 한 번만 생성
    
    fun efficientProcessing() {
        repeat(1000) {
            processor.process() // 재사용
        }
    }
}
```

### 2. 지연 초기화 사용
```kotlin
class LazyInitialization {
    // 오버헤드: 필요할 때만 초기화
    private val expensiveResource by lazy {
        createExpensiveResource()
    }
    
    fun useResource() {
        expensiveResource.doSomething() // 실제 사용 시점에 초기화
    }
}
```
