# ViewModel을 쓰는 이유

> 안드로이드 개발을 하다 보면 화면 회전이나 액티비티 재생성 시 데이터 손실 문제를 경험해보셨을 거예요.  
> **ViewModel** 은 이런 문제를 해결하고, UI 로직과 비즈니스 로직을 분리하여  
> 더 유지보수하기 좋은 코드를 만들 수 있게 도와주는 안드로이드 아키텍처 컴포넌트입니다.  
> MVVM 패턴의 핵심 구성요소로, 클린 아키텍처 구현에 필수적인 역할을 합니다.

---

## 1. ViewModel이란?

**ViewModel**은 안드로이드 아키텍처 컴포넌트 중 하나로, UI 관련 데이터를 수명주기를 인식하여 저장하고 관리하는 클래스입니다.

### 주요 특징

- **수명주기 인식**: Activity/Fragment의 구성 변경에서 살아남음
- **UI와 데이터 분리**: Presentation 로직과 비즈니스 로직 분리
- **메모리 누수 방지**: 액티비티/프래그먼트에 대한 참조를 갖지 않음
- **데이터 공유**: 여러 Fragment 간 데이터 공유 가능

```kotlin
// ViewModel 기본 구조
class UserViewModel : ViewModel() {
    
    private val _users = MutableLiveData<List<User>>()
    val users: LiveData<List<User>> = _users
    
    private val _isLoading = MutableLiveData<Boolean>()
    val isLoading: LiveData<Boolean> = _isLoading
    
    fun loadUsers() {
        _isLoading.value = true
        
        // 비즈니스 로직 실행
        viewModelScope.launch {
            try {
                val userList = userRepository.getUsers()
                _users.value = userList
            } catch (e: Exception) {
                // 에러 처리
            } finally {
                _isLoading.value = false
            }
        }
    }
    
    // ViewModel이 소멸될 때 호출
    override fun onCleared() {
        super.onCleared()
        // 리소스 정리
    }
}
```

---

## 2. ViewModel의 수명주기

### 수명주기 비교

| 구성 요소 | 수명주기 | 구성 변경 시 | 메모리에서 제거 시점 |
|-----------|----------|--------------|-------------------|
| **Activity** | onCreate ~ onDestroy | 재생성됨 | onDestroy 호출 시 |
| **Fragment** | onCreateView ~ onDestroyView | 재생성됨 | onDestroyView 호출 시 |
| **ViewModel** | 첫 생성 ~ onCleared | **유지됨** | Activity 완전 종료 시 |

### 수명주기 다이어그램

```
Activity 생성 → ViewModel 생성
     ↓              ↓
Activity 실행 ← ViewModel 데이터 보존
     ↓              ↓
화면 회전/재생성 → ViewModel 유지
     ↓              ↓
Activity 재생성 ← 기존 ViewModel 사용
     ↓              ↓
Activity 완전 종료 → ViewModel.onCleared() 호출
```

### 실제 동작 예제

```kotlin
class LifecycleTestViewModel : ViewModel() {
    
    private var creationTime: Long = System.currentTimeMillis()
    private var accessCount: Int = 0
    
    init {
        Log.d("ViewModel", "ViewModel 생성: $creationTime")
    }
    
    fun incrementAccess(): String {
        accessCount++
        return "생성 시간: $creationTime, 접근 횟수: $accessCount"
    }
    
    override fun onCleared() {
        super.onCleared()
        Log.d("ViewModel", "ViewModel 소멸: onCleared() 호출")
    }
}

class TestActivity : AppCompatActivity() {
    
    private val viewModel: LifecycleTestViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_test)
        
        // 화면 회전해도 같은 ViewModel 인스턴스 사용
        val info = viewModel.incrementAccess()
        Log.d("Activity", "onCreate: $info")
    }
}
```

---

## 3. ViewModel과 다른 패턴 비교

### 3-1. MVC vs MVVM (ViewModel 사용)

| 항목 | MVC | MVVM + ViewModel |
|------|-----|------------------|
| **View-Controller 결합도** | 높음 | 낮음 |
| **테스트 용이성** | 어려움 | 쉬움 |
| **비즈니스 로직 위치** | Controller | ViewModel |
| **데이터 바인딩** | 수동 | 자동 (LiveData/Flow) |
| **구성 변경 대응** | 복잡함 | 자동 처리 |

```kotlin
// MVC 패턴 (Activity가 Controller 역할)
class MVCActivity : AppCompatActivity() {
    
    private lateinit var userModel: UserModel
    private lateinit var adapter: UserAdapter
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        userModel = UserModel()
        
        // Controller가 직접 View 조작
        loadButton.setOnClickListener {
            progressBar.visibility = View.VISIBLE
            
            userModel.loadUsers { users ->
                runOnUiThread {
                    progressBar.visibility = View.GONE
                    adapter.updateUsers(users)
                }
            }
        }
    }
}

// MVVM 패턴 (ViewModel 사용)
class MVVMActivity : AppCompatActivity() {
    
    private val viewModel: UserViewModel by viewModels()
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        // 데이터 바인딩으로 자동 UI 업데이트
        viewModel.users.observe(this) { users ->
            adapter.submitList(users)
        }
        
        viewModel.isLoading.observe(this) { isLoading ->
            progressBar.isVisible = isLoading
        }
        
        // 사용자 액션을 ViewModel에 위임
        loadButton.setOnClickListener {
            viewModel.loadUsers()
        }
    }
}
```
