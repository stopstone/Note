# TDD와 BDD의 차이

## 면접 질문
> "TDD와 BDD의 차이를 아시나요?"

## 답변

### 1. 핵심 요약
**TDD(Test-Driven Development)** 는 테스트 코드를 먼저 작성하고 그 테스트를 통과하는 코드를 구현하는 개발 방법론이고,  
**BDD(Behavior-Driven Development)** 는 사용자의 행동과 비즈니스 요구사항을 중심으로 테스트를 작성하는 방법론입니다.

간단히 말하면:
- **TDD**: "이 함수가 제대로 동작하는가?"
- **BDD**: "사용자가 이 기능을 사용할 때 올바른 결과를 얻는가?"

---

## 2. TDD (Test-Driven Development)

### 정의
테스트 주도 개발은 코드를 작성하기 전에 먼저 테스트를 작성하는 개발 방식입니다.

### TDD 사이클 (Red-Green-Refactor)

```
1. Red: 실패하는 테스트 작성
    ↓
2. Green: 테스트를 통과하는 최소한의 코드 작성
    ↓
3. Refactor: 코드 개선 및 리팩토링
    ↓
   (반복)
```

### TDD의 특징
- **개발자 중심**: 코드의 정확성과 품질에 초점
- **기술적 관점**: 함수, 메서드, 클래스 단위의 테스트
- **How(어떻게)** 에 집중
- Unit Test 중심

### Android에서의 TDD 예시

#### 사용자 정보 검증 유틸리티

```kotlin
// 1. Red: 실패하는 테스트 먼저 작성
class UserValidatorTest {
    
    private lateinit var userValidator: UserValidator
    
    @Before
    fun setUp() {
        userValidator = UserValidator()
    }
    
    @Test
    fun `이메일 형식이 올바르면 true를 반환한다`() {
        // Arrange
        val inputEmail = "test@example.com"
        
        // Act
        val actualResult = userValidator.isValidEmail(inputEmail)
        
        // Assert
        assertTrue(actualResult)
    }
    
    @Test
    fun `이메일 형식이 올바르지 않으면 false를 반환한다`() {
        // Arrange
        val inputEmail = "invalid-email"
        
        // Act
        val actualResult = userValidator.isValidEmail(inputEmail)
        
        // Assert
        assertFalse(actualResult)
    }
    
    @Test
    fun `비밀번호가 8자 이상이면 true를 반환한다`() {
        // Arrange
        val inputPassword = "password123"
        
        // Act
        val actualResult = userValidator.isValidPassword(inputPassword)
        
        // Assert
        assertTrue(actualResult)
    }
    
    @Test
    fun `비밀번호가 8자 미만이면 false를 반환한다`() {
        // Arrange
        val inputPassword = "pass"
        
        // Act
        val actualResult = userValidator.isValidPassword(inputPassword)
        
        // Assert
        assertFalse(actualResult)
    }
}
```

```kotlin
// 2. Green: 테스트를 통과하는 최소한의 코드 작성
class UserValidator {
    
    /**
     * 이메일 형식이 유효한지 검증한다
     * @param email 검증할 이메일 주소
     * @return 유효하면 true, 그렇지 않으면 false
     */
    fun isValidEmail(email: String): Boolean {
        val emailPattern = "[a-zA-Z0-9._-]+@[a-z]+\\.+[a-z]+"
        return email.matches(emailPattern.toRegex())
    }
    
    /**
     * 비밀번호가 유효한지 검증한다 (최소 8자 이상)
     * @param password 검증할 비밀번호
     * @return 유효하면 true, 그렇지 않으면 false
     */
    fun isValidPassword(password: String): Boolean {
        return password.length >= 8
    }
}
```

```kotlin
// 3. Refactor: 더 나은 구조로 개선
class UserValidator {
    
    companion object {
        private const val EMAIL_PATTERN = "[a-zA-Z0-9._-]+@[a-z]+\\.+[a-z]+"
        private const val MIN_PASSWORD_LENGTH = 8
    }
    
    /**
     * 이메일 형식이 유효한지 검증한다
     * @param email 검증할 이메일 주소
     * @return 유효하면 true, 그렇지 않으면 false
     */
    fun isValidEmail(email: String): Boolean {
        if (email.isBlank()) return false
        return email.matches(EMAIL_PATTERN.toRegex())
    }
    
    /**
     * 비밀번호가 유효한지 검증한다 (최소 8자 이상)
     * @param password 검증할 비밀번호
     * @return 유효하면 true, 그렇지 않으면 false
     */
    fun isValidPassword(password: String): Boolean {
        return password.length >= MIN_PASSWORD_LENGTH
    }
}
```

---

## 3. BDD (Behavior-Driven Development)

### 정의
행동 주도 개발은 사용자의 행동과 비즈니스 요구사항을 중심으로 테스트를 작성하는 개발 방식입니다. TDD를 확장한 개념으로, 비개발자도 이해할 수 있는 자연어로 테스트를 작성합니다.

### BDD 사이클 (Given-When-Then)

```
Given (주어진 상황)
    ↓
When (어떤 행동을 했을 때)
    ↓
Then (예상되는 결과)
```

### BDD의 특징
- **사용자 중심**: 비즈니스 요구사항과 사용자 시나리오에 초점
- **행동 관점**: 전체적인 시나리오와 사용자 경험
- **What(무엇을)** 에 집중
- 비개발자도 이해 가능한 명세
- Acceptance Test 중심

### Android에서의 BDD 예시

#### 로그인 기능 시나리오

```kotlin
// BDD 스타일의 테스트 작성
class LoginFeatureTest {
    
    private lateinit var loginViewModel: LoginViewModel
    private lateinit var mockAuthRepository: AuthRepository
    
    @Before
    fun setUp() {
        mockAuthRepository = mockk()
        loginViewModel = LoginViewModel(mockAuthRepository)
    }
    
    @Test
    fun `사용자가 올바른 이메일과 비밀번호로 로그인하면 홈 화면으로 이동한다`() {
        // Given: 사용자가 로그인 화면에 있고, 올바른 계정 정보가 있다
        val givenEmail = "user@example.com"
        val givenPassword = "password123"
        val expectedUser = User(id = "1", email = givenEmail, name = "홍길동")
        
        coEvery { 
            mockAuthRepository.login(givenEmail, givenPassword) 
        } returns Result.success(expectedUser)
        
        // When: 사용자가 로그인 버튼을 누른다
        loginViewModel.login(givenEmail, givenPassword)
        
        // Then: 로그인이 성공하고 홈 화면으로 이동한다
        val actualState = loginViewModel.loginState.value
        
        assertTrue(actualState is LoginState.Success)
        assertEquals(expectedUser, (actualState as LoginState.Success).user)
    }
    
    @Test
    fun `사용자가 잘못된 비밀번호로 로그인하면 에러 메시지를 표시한다`() {
        // Given: 사용자가 로그인 화면에 있고, 잘못된 비밀번호를 입력한다
        val givenEmail = "user@example.com"
        val givenWrongPassword = "wrongpass"
        val expectedError = "비밀번호가 일치하지 않습니다"
        
        coEvery { 
            mockAuthRepository.login(givenEmail, givenWrongPassword) 
        } returns Result.failure(AuthException(expectedError))
        
        // When: 사용자가 로그인 버튼을 누른다
        loginViewModel.login(givenEmail, givenWrongPassword)
        
        // Then: 에러 메시지가 표시된다
        val actualState = loginViewModel.loginState.value
        
        assertTrue(actualState is LoginState.Error)
        assertEquals(expectedError, (actualState as LoginState.Error).message)
    }
    
    @Test
    fun `사용자가 네트워크 연결 없이 로그인하면 연결 오류 메시지를 표시한다`() {
        // Given: 사용자가 로그인 화면에 있고, 네트워크 연결이 없다
        val givenEmail = "user@example.com"
        val givenPassword = "password123"
        val expectedError = "네트워크 연결을 확인해주세요"
        
        coEvery { 
            mockAuthRepository.login(givenEmail, givenPassword) 
        } returns Result.failure(NetworkException(expectedError))
        
        // When: 사용자가 로그인 버튼을 누른다
        loginViewModel.login(givenEmail, givenPassword)
        
        // Then: 네트워크 오류 메시지가 표시된다
        val actualState = loginViewModel.loginState.value
        
        assertTrue(actualState is LoginState.Error)
        assertEquals(expectedError, (actualState as LoginState.Error).message)
    }
}
```

```kotlin
// ViewModel 구현
class LoginViewModel(
    private val authRepository: AuthRepository
) : ViewModel() {
    
    private val _loginState = MutableStateFlow<LoginState>(LoginState.Idle)
    val loginState: StateFlow<LoginState> = _loginState.asStateFlow()
    
    /**
     * 로그인을 수행한다
     * @param email 사용자 이메일
     * @param password 사용자 비밀번호
     */
    fun login(email: String, password: String) {
        viewModelScope.launch {
            _loginState.value = LoginState.Loading
            
            authRepository.login(email, password)
                .onSuccess { user ->
                    _loginState.value = LoginState.Success(user)
                }
                .onFailure { exception ->
                    _loginState.value = LoginState.Error(
                        message = exception.message ?: "알 수 없는 오류가 발생했습니다"
                    )
                }
        }
    }
}

// State 정의
sealed class LoginState {
    object Idle : LoginState()
    object Loading : LoginState()
    data class Success(val user: User) : LoginState()
    data class Error(val message: String) : LoginState()
}

// Domain Model
data class User(
    val id: String,
    val email: String,
    val name: String
)

// Exception 정의
class AuthException(message: String) : Exception(message)
class NetworkException(message: String) : Exception(message)
```

---
