# Unit Test의 Mock, Spy, Stub, SUT

> 유닛 테스트와 관련하여 Mock, Spy, Stub, SUT 같은 용어들이 있습니다.    
> 이 용어들은 테스트 환경에서 실제 객체를 대체하거나 테스트 대상을 지칭하는 개념입니다.  

---

## 1. Test Double이란?

Test Double은 **테스트를 위해 실제 객체를 대신하는 모든 종류의 가짜 객체**를 의미합니다.  
영화 촬영에서 위험한 장면을 찍을 때 배우 대신 스턴트 더블이 연기하는 것처럼, 테스트에서도 실제 객체 대신 가짜 객체를 사용합니다.

### Test Double의 종류

- **Dummy**: 파라미터를 채우기 위해서만 사용되는 객체
- **Stub**: 미리 정의된 응답을 반환하는 객체
- **Spy**: 실제 객체의 일부 기능을 사용하면서 호출 정보를 기록하는 객체
- **Mock**: 호출에 대한 기대값을 명세하고 검증하는 객체
- **Fake**: 실제 동작하는 구현을 가지지만 프로덕션에는 적합하지 않은 객체

---

## 2. SUT (System Under Test)

SUT는 **현재 테스트하고 있는 대상 시스템**을 의미합니다.  
즉, 우리가 검증하려는 실제 클래스나 메서드를 말합니다.

```kotlin
class UserRepository(
    private val apiService: ApiService,
    private val userDao: UserDao
) {
    // 이 메서드를 테스트한다면, UserRepository가 SUT입니다
    suspend fun getUserById(userId: String): Result<User> {
        return try {
            val user = apiService.getUser(userId)
            userDao.insertUser(user)
            Result.success(user)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}

// 테스트 코드
class UserRepositoryTest {
    
    // SUT: 실제로 테스트하려는 대상
    private lateinit var userRepository: UserRepository
    
    // Dependencies: Mock이나 Stub으로 대체할 의존성들
    private lateinit var apiService: ApiService
    private lateinit var userDao: UserDao
    
    @Before
    fun setUp() {
        // 의존성을 Mock으로 생성
        apiService = mockk()
        userDao = mockk()
        
        // SUT 생성 - 실제 객체를 사용
        userRepository = UserRepository(apiService, userDao)
    }
    
    @Test
    fun `사용자 조회 성공 테스트`() = runTest {
        // Given: 테스트 준비
        val expectedUser = User("1", "홍길동")
        coEvery { apiService.getUser("1") } returns expectedUser
        coEvery { userDao.insertUser(any()) } just Runs
        
        // When: SUT의 메서드 실행
        val result = userRepository.getUserById("1")
        
        // Then: 결과 검증
        assertTrue(result.isSuccess)
        assertEquals(expectedUser, result.getOrNull())
    }
}
```

### SUT의 특징

- SUT는 실제 구현된 클래스의 인스턴스입니다
- 테스트의 주인공이며, 우리가 검증하려는 대상입니다
- 의존성들은 Test Double로 대체하지만, SUT는 실제 객체를 사용합니다

---

## 3. Stub (스텁)

Stub은 **미리 정의된 답변을 반환하도록 프로그래밍된 객체**입니다.  
테스트 중에 호출되면 항상 같은 답변을 제공하며, 호출에 대한 검증은 하지 않습니다.

```kotlin
interface WeatherApi {
    suspend fun getCurrentWeather(city: String): WeatherResponse
}

class WeatherViewModel(
    private val weatherApi: WeatherApi
) : ViewModel() {
    
    private val _weatherState = MutableStateFlow<WeatherState>(WeatherState.Idle)
    val weatherState: StateFlow<WeatherState> = _weatherState.asStateFlow()
    
    fun loadWeather(city: String) {
        viewModelScope.launch {
            _weatherState.value = WeatherState.Loading
            try {
                // API 호출
                val response = weatherApi.getCurrentWeather(city)
                _weatherState.value = WeatherState.Success(response)
            } catch (e: Exception) {
                _weatherState.value = WeatherState.Error(e.message ?: "Unknown error")
            }
        }
    }
}

// Stub을 사용한 테스트
class WeatherViewModelTest {
    
    private lateinit var viewModel: WeatherViewModel
    private lateinit var weatherApi: WeatherApi
    
    @Before
    fun setUp() {
        weatherApi = mockk() // MockK를 사용하여 Stub 생성
        viewModel = WeatherViewModel(weatherApi)
    }
    
    @Test
    fun `날씨 정보 로딩 성공 테스트`() = runTest {
        // Given: Stub 설정 - 특정 입력에 대해 미리 정의된 응답 반환
        val stubResponse = WeatherResponse(
            temperature = 25.0,
            description = "맑음"
        )
        coEvery { weatherApi.getCurrentWeather("서울") } returns stubResponse
        
        // When: 메서드 실행
        viewModel.loadWeather("서울")
        
        // Then: 상태 검증 (Stub 호출 여부는 검증하지 않음)
        val state = viewModel.weatherState.value
        assertTrue(state is WeatherState.Success)
        assertEquals(25.0, (state as WeatherState.Success).data.temperature)
    }
}
```

### Stub의 특징

- **목적**: 테스트에 필요한 데이터를 제공하는 것
- **검증**: 호출되었는지 검증하지 않음 (간접 출력에만 관심)
- **동작**: 항상 같은 답변을 반환
- **사용 시점**: 의존성의 반환값이 테스트에 필요할 때

---

## 4. Mock (목)

Mock은 **호출에 대한 기대값(expectation)을 명세하고, 그 기대값대로 호출되었는지 검증하는 객체**입니다.  
Stub과 달리 "어떻게 호출되어야 하는가"에 관심이 있습니다.

```kotlin
class UserRegistrationService(
    private val userDao: UserDao,
    private val emailService: EmailService,
    private val analyticsTracker: AnalyticsTracker
) {
    suspend fun registerUser(email: String, password: String): Result<Unit> {
        return try {
            // 1. 사용자 저장
            val user = User(email = email, password = password)
            userDao.insertUser(user)
            
            // 2. 환영 이메일 전송
            emailService.sendWelcomeEmail(email)
            
            // 3. 분석 이벤트 전송
            analyticsTracker.trackEvent("user_registered", mapOf("email" to email))
            
            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}

// Mock을 사용한 테스트
class UserRegistrationServiceTest {
    
    private lateinit var registrationService: UserRegistrationService
    private lateinit var userDao: UserDao
    private lateinit var emailService: EmailService
    private lateinit var analyticsTracker: AnalyticsTracker
    
    @Before
    fun setUp() {
        // Mock 객체 생성
        userDao = mockk(relaxed = true)
        emailService = mockk(relaxed = true)
        analyticsTracker = mockk(relaxed = true)
        
        registrationService = UserRegistrationService(
            userDao,
            emailService,
            analyticsTracker
        )
    }
    
    @Test
    fun `사용자 등록 시 모든 단계가 올바르게 실행되는지 검증`() = runTest {
        // Given
        val testEmail = "test@example.com"
        val testPassword = "password123"
        
        // When
        val result = registrationService.registerUser(testEmail, testPassword)
        
        // Then: Mock 검증 - 각 메서드가 올바르게 호출되었는지 확인
        // 1. 사용자가 정확히 한 번 저장되었는지
        coVerify(exactly = 1) {
            userDao.insertUser(
                match { it.email == testEmail && it.password == testPassword }
            )
        }
        
        // 2. 환영 이메일이 정확한 주소로 전송되었는지
        coVerify(exactly = 1) {
            emailService.sendWelcomeEmail(testEmail)
        }
        
        // 3. 분석 이벤트가 올바른 파라미터로 전송되었는지
        coVerify(exactly = 1) {
            analyticsTracker.trackEvent(
                "user_registered",
                match { it["email"] == testEmail }
            )
        }
        
        // 4. 호출 순서도 검증 가능
        coVerifyOrder {
            userDao.insertUser(any())
            emailService.sendWelcomeEmail(testEmail)
            analyticsTracker.trackEvent(any(), any())
        }
        
        assertTrue(result.isSuccess)
    }
    
    @Test
    fun `DAO 실패 시 이메일과 분석 이벤트는 호출되지 않아야 함`() = runTest {
        // Given: DAO가 예외를 던지도록 설정
        coEvery { userDao.insertUser(any()) } throws Exception("DB Error")
        
        // When
        val result = registrationService.registerUser("test@example.com", "password")
        
        // Then: 실패 이후의 메서드들은 호출되지 않았는지 검증
        assertTrue(result.isFailure)
        
        // 이메일과 분석 이벤트는 호출되지 않아야 함
        coVerify(exactly = 0) { emailService.sendWelcomeEmail(any()) }
        coVerify(exactly = 0) { analyticsTracker.trackEvent(any(), any()) }
    }
}
```

### Mock의 특징

- **목적**: 객체 간의 상호작용을 검증하는 것
- **검증**: 특정 메서드가 호출되었는지, 몇 번 호출되었는지, 어떤 파라미터로 호출되었는지 검증
- **동작**: 기대값을 명세하고, 그에 따라 동작
- **사용 시점**: 의존성과의 상호작용이 중요할 때

---

## 5. Spy (스파이)

Spy는 **실제 객체를 감싸서 일부는 실제 동작을 수행하고, 일부는 가짜 동작을 수행하며, 모든 호출을 기록하는 객체**입니다.  
실제 객체의 특정 메서드만 오버라이드하고 싶을 때 유용합니다.

```kotlin
class OrderService(
    private val paymentGateway: PaymentGateway,
    private val inventoryService: InventoryService,
    private val notificationService: NotificationService
) {
    fun processOrder(order: Order): OrderResult {
        // 1. 재고 확인
        val hasStock = inventoryService.checkStock(order.productId, order.quantity)
        if (!hasStock) {
            return OrderResult.OutOfStock
        }
        
        // 2. 결제 처리
        val paymentSuccess = paymentGateway.processPayment(order.totalAmount)
        if (!paymentSuccess) {
            return OrderResult.PaymentFailed
        }
        
        // 3. 재고 감소
        inventoryService.decreaseStock(order.productId, order.quantity)
        
        // 4. 알림 전송
        notificationService.sendOrderConfirmation(order.userId)
        
        return OrderResult.Success
    }
}

class NotificationService {
    // 실제 구현이 있는 메서드들
    fun sendOrderConfirmation(userId: String) {
        val message = createConfirmationMessage(userId)
        sendNotification(message)
    }
    
    fun createConfirmationMessage(userId: String): String {
        return "주문이 완료되었습니다. 사용자 ID: $userId"
    }
    
    fun sendNotification(message: String) {
        // 실제 알림 전송 로직 (네트워크 호출 등)
        println("Sending: $message")
    }
}

// Spy를 사용한 테스트
class OrderServiceTest {
    
    private lateinit var orderService: OrderService
    private lateinit var paymentGateway: PaymentGateway
    private lateinit var inventoryService: InventoryService
    private lateinit var notificationService: NotificationService
    
    @Before
    fun setUp() {
        paymentGateway = mockk()
        inventoryService = mockk()
        
        // Spy 생성: 실제 NotificationService 객체를 감싸서 사용
        notificationService = spyk(NotificationService())
        
        orderService = OrderService(
            paymentGateway,
            inventoryService,
            notificationService
        )
    }
    
    @Test
    fun `주문 처리 시 실제 메시지 생성 로직은 사용하되 전송은 모의로 처리`() {
        // Given
        val testOrder = Order(
            productId = "P001",
            quantity = 2,
            totalAmount = 50000,
            userId = "USER123"
        )
        
        every { inventoryService.checkStock(any(), any()) } returns true
        every { paymentGateway.processPayment(any()) } returns true
        every { inventoryService.decreaseStock(any(), any()) } just Runs
        
        // Spy 설정: sendNotification만 모의로 처리 (실제 네트워크 호출 방지)
        every { notificationService.sendNotification(any()) } just Runs
        // createConfirmationMessage는 실제 로직을 사용 (Spy의 특징)
        
        // When
        val result = orderService.processOrder(testOrder)
        
        // Then
        assertEquals(OrderResult.Success, result)
        
        // Spy 검증: 메서드 호출 여부 확인
        verify(exactly = 1) {
            // 실제 메시지 생성 메서드가 호출되었는지 확인
            notificationService.createConfirmationMessage("USER123")
        }
        
        verify(exactly = 1) {
            // 모의 전송 메서드가 실제 생성된 메시지로 호출되었는지 확인
            notificationService.sendNotification(
                "주문이 완료되었습니다. 사용자 ID: USER123"
            )
        }
        
        // 전체 sendOrderConfirmation도 호출되었는지 확인
        verify(exactly = 1) {
            notificationService.sendOrderConfirmation("USER123")
        }
    }
    
    @Test
    fun `Spy를 통해 실제 메서드 호출 횟수 추적`() {
        // Given
        val testOrder = Order(
            productId = "P001",
            quantity = 1,
            totalAmount = 30000,
            userId = "USER456"
        )
        
        every { inventoryService.checkStock(any(), any()) } returns true
        every { paymentGateway.processPayment(any()) } returns true
        every { inventoryService.decreaseStock(any(), any()) } just Runs
        every { notificationService.sendNotification(any()) } just Runs
        
        // When: 같은 주문을 두 번 처리
        orderService.processOrder(testOrder)
        orderService.processOrder(testOrder)
        
        // Then: Spy를 통해 실제 호출 횟수 검증
        verify(exactly = 2) {
            notificationService.createConfirmationMessage("USER456")
        }
        
        verify(exactly = 2) {
            notificationService.sendNotification(any())
        }
    }
}
```

### Spy의 특징

- **목적**: 실제 객체의 동작은 유지하면서 특정 부분만 제어하고 호출을 추적
- **검증**: Mock처럼 호출을 검증할 수 있음
- **동작**: 기본적으로 실제 메서드를 호출하며, 필요한 부분만 오버라이드 가능
- **사용 시점**: 
  - 실제 구현의 일부는 사용하고 싶지만 특정 부분(예: 네트워크 호출)은 모의로 처리하고 싶을 때
  - 레거시 코드를 테스트할 때
  - 부분적으로만 모의 객체가 필요할 때

---
