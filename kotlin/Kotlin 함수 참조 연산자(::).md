# Kotlin 함수 참조 연산자(::)

> Kotlin의 `::` 연산자는 함수 참조(Function Reference)와 클래스 참조(Class Reference)를 위해 사용됩니다.  
> 이 연산자를 사용하면 함수나 프로퍼티, 클래스를 값처럼 다룰 수 있어 고차 함수에 전달하거나 변수에 할당할 수 있습니다.  
> 이 글에서는 `::` 연산자의 다양한 활용법과 예제를 살펴보겠습니다.

---

## 1. `::` 연산자란?

`::` 연산자는 **함수, 프로퍼티, 생성자를 참조**할 때 사용하는 Kotlin의 특별한 연산자입니다.  
이를 통해 함수를 직접 호출하지 않고, 함수 자체를 객체처럼 전달할 수 있습니다.

### 1.1 기본 개념

```kotlin
// 함수를 직접 호출
fun printMessage(message: String) {
    println(message)
}

// 함수를 호출
printMessage("안녕하세요") // 출력: 안녕하세요

// 함수를 참조 (호출하지 않음)
val functionReference = ::printMessage // 함수 참조 저장
functionReference("반가워요") // 출력: 반가워요
```

---

## 2. `::` 연산자의 주요 사용 사례

### 2.1 최상위 함수 참조

최상위 레벨에 정의된 함수를 참조할 수 있습니다.

```kotlin
// 최상위 함수 정의
fun isEven(number: Int): Boolean {
    return number % 2 == 0
}

// 함수 참조를 사용하여 filter에 전달
val numbers = listOf(1, 2, 3, 4, 5, 6)
val evenNumbers = numbers.filter(::isEven)

println(evenNumbers) // 출력: [2, 4, 6]
```

### 2.2 클래스 멤버 함수 참조

클래스의 멤버 함수를 참조할 때는 `클래스명::함수명` 형태를 사용합니다.

```kotlin
class StringProcessor {
    fun toUpperCase(text: String): String {
        return text.uppercase()
    }
}

// 인스턴스를 통한 멤버 함수 참조
val processor = StringProcessor()
val upperCaseFunction = processor::toUpperCase

println(upperCaseFunction("hello")) // 출력: HELLO

// 클래스를 통한 멤버 함수 참조
val texts = listOf("hello", "world")
val upperTexts = texts.map(processor::toUpperCase)

println(upperTexts) // 출력: [HELLO, WORLD]
```

### 2.3 생성자 참조

생성자도 `::` 연산자로 참조할 수 있습니다.

```kotlin
// 데이터 클래스 정의
data class User(val name: String, val age: Int)

// 생성자 참조
val createUser = ::User

// 생성자 참조를 사용하여 객체 생성
val user = createUser("신짱구", 5)
println(user) // 출력: User(name=신짱구, age=5)

// map과 함께 사용
val names = listOf("신짱구", "김철수", "한유리")
val users = names.map { name -> User(name, 5) }

// 생성자 참조로 간단하게
data class SimpleUser(val name: String)
val simpleUsers = names.map(::SimpleUser)
println(simpleUsers) 
// 출력: [SimpleUser(name=신짱구), SimpleUser(name=김철수), SimpleUser(name=한유리)]
```

### 2.4 프로퍼티 참조

프로퍼티도 참조할 수 있으며, `KProperty` 타입의 객체를 반환합니다.

```kotlin
data class Person(val name: String, val age: Int)

// 프로퍼티 참조
val nameProperty = Person::name

val person = Person("신짱구", 5)

// 프로퍼티 값 가져오기
println(nameProperty.get(person)) // 출력: 신짱구

// 프로퍼티 이름 가져오기
println(nameProperty.name) // 출력: name

// map과 함께 사용
val people = listOf(
    Person("신짱구", 5),
    Person("김철수", 5),
    Person("한유리", 5)
)
val names = people.map(Person::name)
println(names) // 출력: [신짱구, 김철수, 한유리]
```

### 2.5 확장 함수 참조

확장 함수도 `::` 연산자로 참조할 수 있습니다.

```kotlin
// String 확장 함수 정의
fun String.addExclamation(): String {
    return "$this!"
}

// 확장 함수 참조
val addExclamationFunction = String::addExclamation

val words = listOf("안녕", "반가워", "좋아")
val excitedWords = words.map(String::addExclamation)

println(excitedWords) // 출력: [안녕!, 반가워!, 좋아!]
```

### 2.6 바인딩된 함수 참조 (Bound Function Reference)

특정 인스턴스에 바인딩된 함수 참조를 만들 수 있습니다.

```kotlin
class Calculator {
    fun add(a: Int, b: Int): Int {
        return a + b
    }
}

val calculator = Calculator()

// 인스턴스에 바인딩된 함수 참조
val addFunction = calculator::add

// 인스턴스 없이 바로 호출 가능
println(addFunction(5, 3)) // 출력: 8
```

---

## 3. Android에서의 활용

### 3.1 리스너에서 함수 참조 사용

```kotlin
class MainActivity : AppCompatActivity() {
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        
        // 람다 대신 함수 참조 사용
        binding.button.setOnClickListener(::onButtonClick)
    }
    
    // 버튼 클릭 핸들러 함수
    private fun onButtonClick(view: View) {
        Toast.makeText(this, "버튼 클릭!", Toast.LENGTH_SHORT).show()
    }
}
```

### 3.2 ViewModel에서 함수 참조 사용

```kotlin
class UserViewModel : ViewModel() {
    
    private val _users = MutableLiveData<List<User>>()
    val users: LiveData<List<User>> = _users
    
    // 사용자 데이터 로드
    fun loadUsers() {
        viewModelScope.launch {
            val userList = userRepository.getUsers()
            
            // map과 함께 함수 참조 사용
            val processedUsers = userList
                .filter(::isValidUser) // 유효한 사용자 필터링
                .map(::enrichUser) // 사용자 정보 보강
            
            _users.value = processedUsers
        }
    }
    
    // 유효한 사용자인지 확인
    private fun isValidUser(user: User): Boolean {
        return user.name.isNotBlank() && user.age > 0
    }
    
    // 사용자 정보 보강
    private fun enrichUser(user: User): User {
        return user.copy(
            name = user.name.trim(),
            displayName = "${user.name} (${user.age}세)"
        )
    }
}
```

### 3.3 RecyclerView Adapter에서 활용

```kotlin
class UserAdapter(
    private val onItemClick: (User) -> Unit // 함수를 파라미터로 받음
) : RecyclerView.Adapter<UserAdapter.UserViewHolder>() {
    
    private val users = mutableListOf<User>()
    
    override fun onBindViewHolder(holder: UserViewHolder, position: Int) {
        val user = users[position]
        holder.bind(user)
        
        // 아이템 클릭 시 전달받은 함수 참조 실행
        holder.itemView.setOnClickListener { onItemClick(user) }
    }
    
    // ... 나머지 코드
}

// Activity/Fragment에서 사용
class UserListFragment : Fragment() {
    
    private val adapter = UserAdapter(::onUserClick)
    
    // 함수 참조로 전달할 메서드
    private fun onUserClick(user: User) {
        Toast.makeText(context, "${user.name} 선택됨", Toast.LENGTH_SHORT).show()
        // 상세 화면으로 이동
        navigateToDetail(user.id)
    }
}
```

---

## 4. `::` 연산자 사용 시 장점

### 4.1 코드 가독성 향상

```kotlin
// 람다 사용 (가독성 낮음)
val numbers = listOf(1, 2, 3, 4, 5)
val doubled = numbers.map { number -> number * 2 }

// 함수 참조 사용 (가독성 높음)
fun double(number: Int) = number * 2
val doubled2 = numbers.map(::double)
```

### 4.2 코드 재사용성

```kotlin
// 재사용 가능한 함수 정의
fun isAdult(age: Int): Boolean = age >= 18

// 여러 곳에서 함수 참조로 재사용
val ages1 = listOf(15, 20, 17, 25, 13)
val adults1 = ages1.filter(::isAdult)

val ages2 = listOf(30, 16, 22, 14, 19)
val adults2 = ages2.filter(::isAdult)
```

### 4.3 테스트 용이성

```kotlin
class UserProcessor {
    
    // 함수를 분리하여 테스트하기 쉽게 구성
    fun processUsers(users: List<User>): List<User> {
        return users
            .filter(::isActiveUser)
            .map(::sanitizeUser)
            .sortedBy(::getUserScore)
    }
    
    // 각 함수를 개별적으로 테스트 가능
    fun isActiveUser(user: User): Boolean {
        return user.isActive && user.lastLoginDate.isAfter(LocalDate.now().minusDays(30))
    }
    
    fun sanitizeUser(user: User): User {
        return user.copy(name = user.name.trim())
    }
    
    fun getUserScore(user: User): Int {
        return user.activityCount + user.purchaseCount * 10
    }
}

// 테스트 코드
class UserProcessorTest {
    
    private val processor = UserProcessor()
    
    @Test
    fun `활성 사용자 필터링 테스트`() {
        // Given
        val activeUser = User(isActive = true, lastLoginDate = LocalDate.now())
        val inactiveUser = User(isActive = false, lastLoginDate = LocalDate.now().minusDays(60))
        
        // When
        val isActive = processor.isActiveUser(activeUser)
        val isInactive = processor.isActiveUser(inactiveUser)
        
        // Then
        assertTrue(isActive)
        assertFalse(isInactive)
    }
}
```

---

## 5. `::class`를 통한 클래스 참조

`::class`를 사용하면 클래스 자체를 참조할 수 있습니다.

```kotlin
class User(val name: String)

// 클래스 참조
val userClass = User::class

println(userClass.simpleName) // 출력: User
println(userClass.qualifiedName) // 출력: com.example.User

// Android에서 Intent로 Activity 시작할 때
val intent = Intent(this, MainActivity::class.java)
startActivity(intent)

// Gson에서 타입 참조
val json = """{"name": "신짱구"}"""
val user = gson.fromJson(json, User::class.java)
```

### 5.1 reified와 함께 사용

```kotlin
// inline 함수와 reified를 함께 사용
inline fun <reified T> Gson.fromJson(json: String): T {
    return this.fromJson(json, T::class.java)
}

// 타입 파라미터 명시 없이 사용 가능
val json = """{"name": "신짱구", "age": 5}"""
val user: User = gson.fromJson(json)
```

---
