# MVI Pattern

> 안드로이드 앱의 상태 관리를 위한 아키텍처 패턴 중 하나인 **MVI(Model-View-Intent)** 는 단방향 데이터 플로우를 기반으로 합니다.  
> MVC, MVP, MVVM과 달리 **예측 가능한 상태 관리**와 **테스트 용이성**을 제공하여 복잡한 UI 상태를 가진 앱에서 매우 유용합니다.  

---

## 1. MVI 패턴이란?

MVI는 **Model-View-Intent**의 약자로, 안드로이드 앱의 상태 관리를 위한 아키텍처 패턴입니다.

### 핵심 구성요소

| 구성요소 | 역할 | 특징 |
|---------|------|------|
| **Model** | 앱의 상태를 나타내는 불변 객체 | UI가 렌더링할 데이터 |
| **View** | UI를 담당하며 Intent를 받아서 Model을 표시 | 사용자 액션을 Intent로 변환 |
| **Intent** | 사용자 액션이나 시스템 이벤트를 나타내는 객체 | 단방향 데이터 플로우의 시작점 |

### 데이터 플로우
```
User Action → Intent → Model → View → UI Update
```



## 2. MVI 패턴의 핵심 구성요소

### Model (상태)
```kotlin
data class UserState(
    val isLoading: Boolean = false,
    val users: List<User> = emptyList(),
    val error: String? = null,
    val isRefreshing: Boolean = false
)
```

### Intent (의도)
```kotlin
sealed class UserIntent {
    object LoadUsers : UserIntent()
    object RefreshUsers : UserIntent()
    data class SearchUsers(val query: String) : UserIntent()
    data class DeleteUser(val userId: String) : UserIntent()
}
```

### Effect (부작용)
```kotlin
sealed class UserEffect {
    data class ShowToast(val message: String) : UserEffect()
    data class NavigateToDetail(val userId: String) : UserEffect()
    object ShowErrorDialog : UserEffect()
}
```

---

## 3. 실제 구현 예제

### ViewModel 구현

```kotlin
class UserViewModel : ViewModel() {
    
    private val _state = MutableStateFlow(UserState())
    val state: StateFlow<UserState> = _state.asStateFlow()
    
    private val _effect = MutableSharedFlow<UserEffect>()
    val effect: SharedFlow<UserEffect> = _effect.asSharedFlow()
    
    fun processIntent(intent: UserIntent) {
        when (intent) {
            is UserIntent.LoadUsers -> loadUsers()
            is UserIntent.RefreshUsers -> refreshUsers()
            is UserIntent.SearchUsers -> searchUsers(intent.query)
            is UserIntent.DeleteUser -> deleteUser(intent.userId)
        }
    }
    
    private fun loadUsers() {
        viewModelScope.launch {
            _state.value = _state.value.copy(isLoading = true)
            
            try {
                val users = userRepository.getUsers()
                _state.value = _state.value.copy(
                    isLoading = false,
                    users = users,
                    error = null
                )
            } catch (e: Exception) {
                _state.value = _state.value.copy(
                    isLoading = false,
                    error = e.message
                )
                _effect.emit(UserEffect.ShowToast("사용자 목록을 불러오는데 실패했습니다."))
            }
        }
    }
    
    private fun refreshUsers() {
        viewModelScope.launch {
            _state.value = _state.value.copy(isRefreshing = true)
            
            try {
                val users = userRepository.refreshUsers()
                _state.value = _state.value.copy(
                    isRefreshing = false,
                    users = users,
                    error = null
                )
                _effect.emit(UserEffect.ShowToast("새로고침 완료"))
            } catch (e: Exception) {
                _state.value = _state.value.copy(
                    isRefreshing = false,
                    error = e.message
                )
                _effect.emit(UserEffect.ShowErrorDialog)
            }
        }
    }
    
    private fun searchUsers(query: String) {
        viewModelScope.launch {
            val filteredUsers = userRepository.searchUsers(query)
            _state.value = _state.value.copy(users = filteredUsers)
        }
    }
    
    private fun deleteUser(userId: String) {
        viewModelScope.launch {
            try {
                userRepository.deleteUser(userId)
                val updatedUsers = _state.value.users.filter { it.id != userId }
                _state.value = _state.value.copy(users = updatedUsers)
                _effect.emit(UserEffect.ShowToast("사용자가 삭제되었습니다."))
            } catch (e: Exception) {
                _effect.emit(UserEffect.ShowToast("삭제에 실패했습니다."))
            }
        }
    }
}
```

### Fragment 구현

```kotlin
class UserListFragment : Fragment() {
    
    private var _binding: FragmentUserListBinding? = null
    private val binding get() = _binding!!
    
    private val viewModel: UserViewModel by viewModels()
    private val userAdapter = UserAdapter { user ->
        viewModel.processIntent(UserIntent.DeleteUser(user.id))
    }
    
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        _binding = FragmentUserListBinding.inflate(inflater, container, false)
        return binding.root
    }
    
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        setupViews()
        observeState()
        observeEffects()
        
        // 초기 데이터 로드
        viewModel.processIntent(UserIntent.LoadUsers)
    }
    
    private fun setupViews() {
        binding.recyclerView.adapter = userAdapter
        binding.swipeRefresh.setOnRefreshListener {
            viewModel.processIntent(UserIntent.RefreshUsers)
        }
        
        binding.searchView.setOnQueryTextListener(object : SearchView.OnQueryTextListener {
            override fun onQueryTextSubmit(query: String?): Boolean {
                query?.let { viewModel.processIntent(UserIntent.SearchUsers(it)) }
                return true
            }
            
            override fun onQueryTextChange(newText: String?): Boolean {
                newText?.let { viewModel.processIntent(UserIntent.SearchUsers(it)) }
                return true
            }
        })
    }
    
    private fun observeState() {
        viewLifecycleOwner.lifecycleScope.launch {
            viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.state.collect { state ->
                    renderState(state)
                }
            }
        }
    }
    
    private fun observeEffects() {
        viewLifecycleOwner.lifecycleScope.launch {
            viewLifecycleOwner.repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.effect.collect { effect ->
                    handleEffect(effect)
                }
            }
        }
    }
    
    private fun renderState(state: UserState) {
        // 로딩 상태 처리
        binding.progressBar.isVisible = state.isLoading
        binding.swipeRefresh.isRefreshing = state.isRefreshing
        
        // 에러 상태 처리
        state.error?.let { error ->
            binding.errorText.text = error
            binding.errorText.isVisible = true
        } ?: run {
            binding.errorText.isVisible = false
        }
        
        // 사용자 목록 업데이트
        userAdapter.submitList(state.users)
        
        // 빈 상태 처리
        binding.emptyState.isVisible = state.users.isEmpty() && !state.isLoading
    }
    
    private fun handleEffect(effect: UserEffect) {
        when (effect) {
            is UserEffect.ShowToast -> {
                Toast.makeText(requireContext(), effect.message, Toast.LENGTH_SHORT).show()
            }
            is UserEffect.NavigateToDetail -> {
                findNavController().navigate(
                    UserListFragmentDirections.actionUserListToDetail(effect.userId)
                )
            }
            is UserEffect.ShowErrorDialog -> {
                AlertDialog.Builder(requireContext())
                    .setTitle("오류")
                    .setMessage("데이터를 불러오는데 실패했습니다.")
                    .setPositiveButton("확인", null)
                    .show()
            }
        }
    }
    
    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
}
```

---

## 4. MVI 패턴의 장단점

### 장점
- **예측 가능한 상태 관리**: 단방향 플로우로 상태 변화 추적 용이
- **테스트 용이성**: 각 컴포넌트를 독립적으로 테스트 가능
- **디버깅 편의성**: 상태 변화의 흐름을 명확하게 파악 가능
- **재현 가능한 버그**: 특정 상태에서의 버그를 쉽게 재현 가능
- **타입 안전성**: Sealed Class를 사용한 타입 안전한 Intent/Effect 처리

### 단점
- **보일러플레이트 코드**: 많은 클래스와 인터페이스 정의 필요
- **학습 곡선**: 복잡한 패턴으로 인한 높은 학습 곡선
- **오버엔지니어링**: 작은 기능에도 과도한 구조 적용 가능성
- **메모리 사용량**: 많은 객체 생성으로 인한 메모리 오버헤드

---

## 5. 실무에서 MVI 적용 시 주의사항

### 적절한 사용 범위 설정

```kotlin
```kotlin
// 좋은 예: 복잡한 상태 관리가 필요한 화면
class ProductListViewModel : ViewModel() {
    // 복잡한 필터링, 정렬, 검색 기능이 있는 경우
}

// 나쁜 예: 단순한 화면에 과도한 적용
class SimpleDetailViewModel : ViewModel() {
    // 단순한 데이터 표시만 하는 경우
}
```
```

### Intent 설계 시 주의사항

```kotlin
// 좋은 예: 명확하고 구체적인 Intent
sealed class UserIntent {
    object LoadUsers : UserIntent()
    data class SearchUsers(val query: String) : UserIntent()
    data class DeleteUser(val userId: String) : UserIntent()
}

// 나쁜 예: 너무 세분화된 Intent
sealed class UserIntent {
    object LoadUsers : UserIntent()
    object LoadUsersFromCache : UserIntent()
    object LoadUsersFromNetwork : UserIntent()
    object LoadUsersFromDatabase : UserIntent()
    // ... 너무 많은 세분화
}
```

### Effect 사용 시 주의사항

```kotlin
// 좋은 예: UI 관련 부작용만 Effect로 처리
sealed class UserEffect {
    data class ShowToast(val message: String) : UserEffect()
    data class NavigateToDetail(val userId: String) : UserEffect()
}

// 나쁜 예: 비즈니스 로직을 Effect로 처리
sealed class UserEffect {
    object SaveToDatabase : UserEffect() // 비즈니스 로직
object CallApi : UserEffect() // 비즈니스 로직
}
```

---

## 6. MVI 패턴 최적화 팁

### 상태 최적화
```kotlin
// 좋은 예: 불변 객체 사용
data class UserState(
    val isLoading: Boolean = false,
    val users: List<User> = emptyList(),
    val error: String? = null
)

// 나쁜 예: 가변 객체 사용
class UserState {
    var isLoading: Boolean = false
    var users: List<User> = emptyList()
    var error: String? = null
}
```

### 성능 최적화
```kotlin
// 좋은 예: Flow 최적화
class UserViewModel : ViewModel() {
    
    private val _state = MutableStateFlow(UserState())
    val state: StateFlow<UserState> = _state.asStateFlow()
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000),
            initialValue = UserState()
        )
    
    // 중복 Intent 방지
    private var isProcessing = false
    
    fun processIntent(intent: UserIntent) {
        if (isProcessing) return
        
        when (intent) {
            is UserIntent.LoadUsers -> {
                if (_state.value.users.isEmpty()) {
                    loadUsers()
                }
            }
            // ... 다른 Intent 처리
        }
    }
}
```

---

## 7. 언제 MVI를 사용해야 할까?

### MVI 사용이 적합한 상황
1. **복잡한 상태 관리가 필요한 화면**
2. **여러 사용자 액션이 있는 화면**
3. **실시간 데이터 업데이트가 필요한 화면**
4. **테스트가 중요한 화면**

### 다른 패턴 사용이 적합한 상황
1. **단순한 데이터 표시 화면**
2. **폼 입력 화면**
3. **단일 기능 화면**

---

## 참고 자료

- [Android Architecture Components](https://developer.android.com/topic/architecture)
- [MVI Architecture with Android](https://hannesdorfmann.com/android/model-view-intent/)
