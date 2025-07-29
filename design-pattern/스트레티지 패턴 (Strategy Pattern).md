# 스트레티지 패턴 (Strategy Pattern)

> 스트레티지 패턴은 알고리즘을 캡슐화하여 런타임에 교체할 수 있게 해주는 디자인 패턴입니다.  
> 안드로이드 개발에서는 결제 방식, 이미지 로딩 라이브러리, 네트워크 통신 방식 등 다양한 전략을 상황에 맞게 선택할 때 유용합니다.  
> 이 글은 **안드로이드 개발할 때 스트레티지 패턴 구현과 실제 활용법**을 정리한 문서입니다.

## 1. 개념

- **알고리즘을 캡슐화하여 런타임에 교체할 수 있게 해주는 디자인 패턴**
- 동일한 인터페이스를 구현하는 여러 전략 클래스를 만들고, 상황에 따라 적절한 전략을 선택하여 사용
- **장점:** 새로운 알고리즘 추가가 쉬우며, 기존 코드 수정 없이 확장 가능
- **단점:** 클래스 수가 증가하고, 모든 전략이 같은 인터페이스를 구현해야 함

---

## 2. 기본 구조

```kotlin
// 전략 인터페이스
interface Strategy {
    fun execute(): String
}

// 구체적인 전략들
class ConcreteStrategyA : Strategy {
    override fun execute(): String = "전략 A 실행"
}

class ConcreteStrategyB : Strategy {
    override fun execute(): String = "전략 B 실행"
}

// 컨텍스트 (전략을 사용하는 클래스)
class Context(private var strategy: Strategy) {
    fun setStrategy(strategy: Strategy) {
        this.strategy = strategy
    }
    
    fun executeStrategy(): String {
        return strategy.execute()
    }
}
```

---

## 3. 안드로이드에서의 활용 예제

### 1. 결제 방식 선택

```kotlin
// 결제 전략 인터페이스
interface PaymentStrategy {
    fun pay(amount: Int): String
}

// 신용카드 결제
class CreditCardPayment : PaymentStrategy {
    override fun pay(amount: Int): String {
        return "신용카드로 ${amount}원 결제 완료"
    }
}

// 카카오페이 결제
class KakaoPayPayment : PaymentStrategy {
    override fun pay(amount: Int): String {
        return "카카오페이로 ${amount}원 결제 완료"
    }
}

// 네이버페이 결제
class NaverPayPayment : PaymentStrategy {
    override fun pay(amount: Int): String {
        return "네이버페이로 ${amount}원 결제 완료"
    }
}

// 결제 관리자
class PaymentManager(private var paymentStrategy: PaymentStrategy) {
    
    fun setPaymentStrategy(strategy: PaymentStrategy) {
        this.paymentStrategy = strategy
    }
    
    fun processPayment(amount: Int): String {
        return paymentStrategy.pay(amount)
    }
}

// 사용 예시
fun main() {
    val paymentManager = PaymentManager(CreditCardPayment())
    
    // 신용카드로 결제
    println(paymentManager.processPayment(10000))
    
    // 카카오페이로 변경
    paymentManager.setPaymentStrategy(KakaoPayPayment())
    println(paymentManager.processPayment(5000))
}

// Factory 패턴과 결합한 더 복잡한 시나리오
enum class PaymentType {
    CREDIT_CARD, KAKAO_PAY, NAVER_PAY, APPLE_PAY, GOOGLE_PAY
}

class PaymentStrategyFactory {
    fun createStrategy(type: PaymentType): PaymentStrategy {
        return when (type) {
            PaymentType.CREDIT_CARD -> CreditCardPayment()
            PaymentType.KAKAO_PAY -> KakaoPayPayment()
            PaymentType.NAVER_PAY -> NaverPayPayment()
            PaymentType.APPLE_PAY -> ApplePayPayment()
            PaymentType.GOOGLE_PAY -> GooglePayPayment()
        }
    }
}

// 추가 결제 전략들
class ApplePayPayment : PaymentStrategy {
    override fun pay(amount: Int): String {
        return "Apple Pay로 ${amount}원 결제 완료"
    }
}

class GooglePayPayment : PaymentStrategy {
    override fun pay(amount: Int): String {
        return "Google Pay로 ${amount}원 결제 완료"
    }
}

// 결제 설정 관리자
class PaymentConfigurationManager {
    private val factory = PaymentStrategyFactory()
    private var currentStrategy: PaymentStrategy? = null
    
    fun configurePayment(type: PaymentType) {
        currentStrategy = factory.createStrategy(type)
    }
    
    fun processPayment(amount: Int): String {
        return currentStrategy?.pay(amount) ?: throw IllegalStateException("결제 방식이 설정되지 않았습니다")
    }
}
```

### 2. 이미지 로딩 라이브러리 선택

```kotlin
// 이미지 로딩 전략 인터페이스
interface ImageLoadingStrategy {
    fun loadImage(url: String, imageView: ImageView)
}

// Glide 사용
class GlideStrategy : ImageLoadingStrategy {
    override fun loadImage(url: String, imageView: ImageView) {
        Glide.with(imageView.context)
            .load(url)
            .into(imageView)
    }
}

// Coil 사용
class CoilStrategy : ImageLoadingStrategy {
    override fun loadImage(url: String, imageView: ImageView) {
        imageView.load(url) {
            crossfade(true)
            placeholder(R.drawable.placeholder)
            error(R.drawable.error)
        }
    }
}

// Picasso 사용
class PicassoStrategy : ImageLoadingStrategy {
    override fun loadImage(url: String, imageView: ImageView) {
        Picasso.get()
            .load(url)
            .placeholder(R.drawable.placeholder)
            .error(R.drawable.error)
            .into(imageView)
    }
}

// 이미지 로더 관리자
class ImageLoader(private var strategy: ImageLoadingStrategy) {
    
    fun setStrategy(strategy: ImageLoadingStrategy) {
        this.strategy = strategy
    }
    
    fun loadImage(url: String, imageView: ImageView) {
        strategy.loadImage(url, imageView)
    }
}
```

### 3. 네트워크 통신 방식 선택

```kotlin
// 네트워크 전략 인터페이스
interface NetworkStrategy {
    suspend fun request(url: String): Response
}

// OkHttp 사용
class OkHttpStrategy : NetworkStrategy {
    private val client = OkHttpClient()
    
    override suspend fun request(url: String): Response {
        return withContext(Dispatchers.IO) {
            val request = Request.Builder()
                .url(url)
                .build()
            client.newCall(request).execute()
        }
    }
}

// Retrofit 사용
class RetrofitStrategy : NetworkStrategy {
    private val retrofit = Retrofit.Builder()
        .baseUrl("https://api.example.com/")
        .addConverterFactory(GsonConverterFactory.create())
        .build()
    
    private val service = retrofit.create(ApiService::class.java)
    
    override suspend fun request(url: String): Response {
        return service.getData()
    }
}

// 네트워크 매니저
class NetworkManager(private var strategy: NetworkStrategy) {
    
    fun setStrategy(strategy: NetworkStrategy) {
        this.strategy = strategy
    }
    
    suspend fun makeRequest(url: String): Response {
        return strategy.request(url)
    }
}
```

### 4. 복잡한 실제 시나리오: 쇼핑몰 앱의 결제 시스템

```kotlin
// 결제 요청 데이터
data class PaymentRequest(
    val amount: Int,
    val currency: String = "KRW",
    val orderId: String,
    val userId: String,
    val items: List<OrderItem>
)

data class OrderItem(
    val productId: String,
    val quantity: Int,
    val price: Int
)

// 결제 결과
sealed class PaymentResult {
    data class Success(val transactionId: String, val amount: Int) : PaymentResult()
    data class Failure(val errorCode: String, val message: String) : PaymentResult()
}

// 향상된 결제 전략 인터페이스
interface PaymentStrategy {
    suspend fun processPayment(request: PaymentRequest): PaymentResult
    fun isAvailable(): Boolean
    fun getSupportedCurrencies(): List<String>
}

// 신용카드 결제 (실제 구현)
class CreditCardPayment : PaymentStrategy {
    override suspend fun processPayment(request: PaymentRequest): PaymentResult {
        return try {
            // 실제 신용카드 결제 로직
            val transactionId = generateTransactionId()
            PaymentResult.Success(transactionId, request.amount)
        } catch (e: Exception) {
            PaymentResult.Failure("CARD_ERROR", "신용카드 결제 실패: ${e.message}")
        }
    }
    
    override fun isAvailable(): Boolean = true
    
    override fun getSupportedCurrencies(): List<String> = listOf("KRW", "USD", "EUR")
    
    private fun generateTransactionId(): String = "TXN_${System.currentTimeMillis()}"
}

// 카카오페이 결제 (실제 구현)
class KakaoPayPayment : PaymentStrategy {
    override suspend fun processPayment(request: PaymentRequest): PaymentResult {
        return try {
            // 카카오페이 API 호출 로직
            val transactionId = "KAKAO_${System.currentTimeMillis()}"
            PaymentResult.Success(transactionId, request.amount)
        } catch (e: Exception) {
            PaymentResult.Failure("KAKAO_ERROR", "카카오페이 결제 실패: ${e.message}")
        }
    }
    
    override fun isAvailable(): Boolean = true
    
    override fun getSupportedCurrencies(): List<String> = listOf("KRW")
}

// 결제 전략 팩토리 (개선된 버전)
class PaymentStrategyFactory {
    fun createStrategy(type: PaymentType): PaymentStrategy {
        return when (type) {
            PaymentType.CREDIT_CARD -> CreditCardPayment()
            PaymentType.KAKAO_PAY -> KakaoPayPayment()
            PaymentType.NAVER_PAY -> NaverPayPayment()
            PaymentType.APPLE_PAY -> ApplePayPayment()
            PaymentType.GOOGLE_PAY -> GooglePayPayment()
        }
    }
    
    fun getAvailableStrategies(): List<PaymentType> {
        return PaymentType.values().filter { type ->
            createStrategy(type).isAvailable()
        }
    }
}

// 결제 서비스 (실제 앱에서 사용할 클래스)
class PaymentService {
    private val factory = PaymentStrategyFactory()
    private var currentStrategy: PaymentStrategy? = null
    
    fun setPaymentStrategy(type: PaymentType) {
        currentStrategy = factory.createStrategy(type)
    }
    
    suspend fun processPayment(request: PaymentRequest): PaymentResult {
        val strategy = currentStrategy ?: throw IllegalStateException("결제 방식이 설정되지 않았습니다")
        
        // 결제 전 검증
        if (!strategy.isAvailable()) {
            return PaymentResult.Failure("UNAVAILABLE", "선택된 결제 방식이 현재 사용할 수 없습니다")
        }
        
        if (!strategy.getSupportedCurrencies().contains(request.currency)) {
            return PaymentResult.Failure("CURRENCY_ERROR", "지원하지 않는 통화입니다: ${request.currency}")
        }
        
        return strategy.processPayment(request)
    }
    
    fun getAvailablePaymentMethods(): List<PaymentType> {
        return factory.getAvailableStrategies()
    }
}

// 사용 예시
class ShoppingCartActivity : AppCompatActivity() {
    private val paymentService = PaymentService()
    
    private fun setupPaymentMethods() {
        val availableMethods = paymentService.getAvailablePaymentMethods()
        // UI에 사용 가능한 결제 방법 표시
    }
    
    private fun processOrder() {
        val request = PaymentRequest(
            amount = 50000,
            orderId = "ORDER_123",
            userId = "USER_456",
            items = listOf(OrderItem("PROD_001", 2, 25000))
        )
        
        lifecycleScope.launch {
            try {
                paymentService.setPaymentStrategy(PaymentType.KAKAO_PAY)
                val result = paymentService.processPayment(request)
                
                when (result) {
                    is PaymentResult.Success -> {
                        showSuccessMessage("결제 완료: ${result.transactionId}")
                    }
                    is PaymentResult.Failure -> {
                        showErrorMessage("결제 실패: ${result.message}")
                    }
                }
            } catch (e: Exception) {
                showErrorMessage("결제 중 오류 발생: ${e.message}")
            }
        }
    }
}
```

---

## 4. 실제 안드로이드 앱에서의 활용

### 1. 데이터 저장 방식 선택

```kotlin
// 데이터 저장 전략
interface DataStorageStrategy {
    suspend fun saveData(key: String, value: String)
    suspend fun getData(key: String): String?
}

// SharedPreferences 사용
class SharedPreferencesStrategy(private val context: Context) : DataStorageStrategy {
    private val prefs = context.getSharedPreferences("app_prefs", Context.MODE_PRIVATE)
    
    override suspend fun saveData(key: String, value: String) {
        prefs.edit().putString(key, value).apply()
    }
    
    override suspend fun getData(key: String): String? {
        return prefs.getString(key, null)
    }
}

// DataStore 사용
class DataStoreStrategy(private val context: Context) : DataStorageStrategy {
    private val dataStore = context.createDataStore(name = "app_data")
    
    override suspend fun saveData(key: String, value: String) {
        dataStore.edit { preferences ->
            preferences[stringPreferencesKey(key)] = value
        }
    }
    
    override suspend fun getData(key: String): String? {
        return dataStore.data.first()[stringPreferencesKey(key)]
    }
}

// Room Database 사용
class RoomStrategy(private val database: AppDatabase) : DataStorageStrategy {
    override suspend fun saveData(key: String, value: String) {
        database.dataDao().insert(DataEntity(key, value))
    }
    
    override suspend fun getData(key: String): String? {
        return database.dataDao().getDataByKey(key)?.value
    }
}
```

### 2. 캐시 전략 선택

```kotlin
// 캐시 전략 인터페이스
interface CacheStrategy {
    fun cache(key: String, data: String)
    fun getCached(key: String): String?
    fun clearCache()
}

// 메모리 캐시
class MemoryCacheStrategy : CacheStrategy {
    private val cache = mutableMapOf<String, String>()
    
    override fun cache(key: String, data: String) {
        cache[key] = data
    }
    
    override fun getCached(key: String): String? {
        return cache[key]
    }
    
    override fun clearCache() {
        cache.clear()
    }
}

// 디스크 캐시
class DiskCacheStrategy(private val context: Context) : CacheStrategy {
    private val cacheDir = context.cacheDir
    
    override fun cache(key: String, data: String) {
        val file = File(cacheDir, key)
        file.writeText(data)
    }
    
    override fun getCached(key: String): String? {
        val file = File(cacheDir, key)
        return if (file.exists()) file.readText() else null
    }
    
    override fun clearCache() {
        cacheDir.listFiles()?.forEach { it.delete() }
    }
}
```

---

## 5. Clean Architecture와 함께 사용

### Repository 패턴과 결합

```kotlin
// 데이터 소스 전략
interface DataSourceStrategy {
    suspend fun getData(): List<Data>
    suspend fun saveData(data: Data)
}

// 로컬 데이터 소스
class LocalDataSourceStrategy(
    private val database: AppDatabase
) : DataSourceStrategy {
    
    override suspend fun getData(): List<Data> {
        return database.dataDao().getAll()
    }
    
    override suspend fun saveData(data: Data) {
        database.dataDao().insert(data)
    }
}

// 원격 데이터 소스
class RemoteDataSourceStrategy(
    private val apiService: ApiService
) : DataSourceStrategy {
    
    override suspend fun getData(): List<Data> {
        return apiService.getData()
    }
    
    override suspend fun saveData(data: Data) {
        apiService.postData(data)
    }
}

// Repository 구현
class DataRepository(
    private var dataSource: DataSourceStrategy
) {
    
    fun setDataSource(strategy: DataSourceStrategy) {
        this.dataSource = strategy
    }
    
    suspend fun getData(): List<Data> {
        return dataSource.getData()
    }
    
    suspend fun saveData(data: Data) {
        dataSource.saveData(data)
    }
}
```

---

## 6. 장단점 정리

| 항목 | 설명 | 구분 |
|------|------|------|
| 확장성 | 새로운 전략 추가가 쉬움 | 장점 |
| 유지보수성 | 기존 코드 수정 없이 새로운 알고리즘 추가 가능 | 장점 |
| 테스트 용이성 | 각 전략을 독립적으로 테스트 가능 | 장점 |
| 런타임 유연성 | 실행 중에 전략을 변경할 수 있음 | 장점 |
| 클래스 수 증가 | 전략마다 새로운 클래스가 필요 | 단점 |
| 인터페이스 의존성 | 모든 전략이 같은 인터페이스를 구현해야 함 | 단점 |
| 복잡성 증가 | 간단한 로직에도 패턴 적용 시 과도한 복잡성 | 단점 |

## 참고 자료
[Strategy Pattern - Refactoring Guru](https://refactoring.guru/design-patterns/strategy)    
[Kotlin Design Patterns](https://github.com/dbacinski/Design-Patterns-In-Kotlin) 
