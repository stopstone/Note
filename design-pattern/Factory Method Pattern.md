# Factory Method Pattern

> 팩토리 메소드 패턴은 객체 생성의 복잡성을 캡슐화하고, 객체 생성 로직을 서브클래스에 위임하는 디자인 패턴입니다.  
> 객체를 직접 생성하는 대신, 팩토리 메소드를 통해 객체를 생성함으로써 유연성과 확장성을 높일 수 있습니다.  
> 안드로이드 개발에서는 다양한 UI 컴포넌트 생성, 네트워크 요청 객체 생성, 데이터베이스 연결 객체 생성 등에서 활용됩니다.  

---

## 1. 팩토리 메소드 패턴, 무엇일까요?

피자 가게를 예로 들어보면 고객이 "페퍼로니 피자 주세요"라고 주문하면,   
피자 가게는 주문에 따라 적절한 피자를 만들어서 제공합니다.   
이때 고객은 피자가 어떻게 만들어지는지 알 필요가 없고, 단지 원하는 피자를 주문하기만 하면 됩니다.  

팩토리 메소드 패턴도 이와 유사합니다.  
클라이언트는 구체적인 객체 생성 과정을 알 필요 없이,  
팩토리에게 "이런 종류의 객체를 만들어줘"라고 요청하면 됩니다.

---

## 2. 팩토리 메소드 패턴의 구현 원리

- 객체 생성 로직을 별도의 팩토리 클래스로 분리합니다.
- 팩토리는 추상 메소드를 통해 객체를 생성하는 인터페이스를 제공합니다.
- 구체적인 객체 생성은 서브클래스에서 담당합니다.
- 클라이언트는 구체적인 클래스를 알지 못해도 객체를 생성할 수 있습니다.

---

## 3. 팩토리 메소드 패턴의 장단점

### 장점
- **캡슐화**: 객체 생성 로직이 캡슐화되어 클라이언트는 생성 과정을 알 필요가 없습니다.
- **확장성**: 새로운 제품을 추가할 때 기존 코드를 수정하지 않고 새로운 팩토리만 추가하면 됩니다.
- **유연성**: 런타임에 어떤 객체를 생성할지 결정할 수 있습니다.
- **의존성 감소**: 클라이언트는 구체적인 클래스가 아닌 추상 클래스나 인터페이스에 의존합니다.
- **테스트 용이성**: Mock 객체를 쉽게 생성할 수 있습니다.

### 단점
- **복잡성 증가**: 클래스 수가 증가하여 시스템이 복잡해질 수 있습니다.
- **성능 오버헤드**: 팩토리 메소드 호출로 인한 약간의 성능 저하가 있을 수 있습니다.
- **디버깅 어려움**: 객체 생성 과정이 숨겨져 있어 디버깅이 어려울 수 있습니다.

---

## 4. 기본 구조와 구현

### 추상 제품 (Product)
```kotlin
abstract class Robot {
    abstract fun move()
    abstract fun attack()
}
```

### 구체 제품 (Concrete Product)
```kotlin
class SuperRobot : Robot() {
    override fun move() {
        println("슈퍼 로봇이 빠르게 이동합니다!")
    }
    
    override fun attack() {
        println("슈퍼 로봇이 강력한 공격을 합니다!")
    }
}

class PowerRobot : Robot() {
    override fun move() {
        println("파워 로봇이 천천히 이동합니다.")
    }
    
    override fun attack() {
        println("파워 로봇이 무거운 공격을 합니다!")
    }
}
```

### 추상 팩토리 (Creator)
```kotlin
abstract class RobotFactory {
    abstract fun createRobot(name: String): Robot
    
    fun orderRobot(name: String): Robot {
        val robot = createRobot(name)
        robot.move()
        robot.attack()
        return robot
    }
}
```

### 구체 팩토리 (Concrete Creator)
```kotlin
class SuperRobotFactory : RobotFactory() {
    override fun createRobot(name: String): Robot {
        return when (name) {
            "super" -> SuperRobot()
            "power" -> PowerRobot()
            else -> throw IllegalArgumentException("알 수 없는 로봇 타입: $name")
        }
    }
}

class ModifiedSuperRobotFactory : RobotFactory() {
    override fun createRobot(name: String): Robot {
        return when (name) {
            "super" -> SuperRobot()
            "power" -> PowerRobot()
            else -> throw IllegalArgumentException("알 수 없는 로봇 타입: $name")
        }
    }
}
```

### 사용 예시
```kotlin
fun main() {
    val factory = SuperRobotFactory()
    
    val superRobot = factory.orderRobot("super")
    val powerRobot = factory.orderRobot("power")
}
```

---

## 5. 안드로이드에서의 활용 사례

### 5.1 Dialog 생성 팩토리

안드로이드에서 다양한 다이얼로그를 생성할 때 팩토리 메소드 패턴을 활용할 수 있습니다.

```kotlin
abstract class DialogFactory {
    abstract fun createDialog(context: Context, message: String): AlertDialog
    
    fun showDialog(context: Context, message: String) {
        val dialog = createDialog(context, message)
        dialog.show()
    }
}

class InfoDialogFactory : DialogFactory() {
    override fun createDialog(context: Context, message: String): AlertDialog {
        return AlertDialog.Builder(context)
            .setTitle("정보")
            .setMessage(message)
            .setPositiveButton("확인") { _, _ -> }
            .create()
    }
}

class ErrorDialogFactory : DialogFactory() {
    override fun createDialog(context: Context, message: String): AlertDialog {
        return AlertDialog.Builder(context)
            .setTitle("오류")
            .setMessage(message)
            .setPositiveButton("확인") { _, _ -> }
            .setNegativeButton("취소") { _, _ -> }
            .create()
    }
}
```

### 5.2 네트워크 요청 팩토리

API 요청 객체를 생성할 때도 활용할 수 있습니다.

```kotlin
abstract class ApiRequestFactory {
    abstract fun createRequest(endpoint: String): ApiRequest
    
    fun executeRequest(endpoint: String, callback: (Result<String>) -> Unit) {
        val request = createRequest(endpoint)
        request.execute(callback)
    }
}

class GetRequestFactory : ApiRequestFactory() {
    override fun createRequest(endpoint: String): ApiRequest {
        return GetRequest(endpoint)
    }
}

class PostRequestFactory : ApiRequestFactory() {
    override fun createRequest(endpoint: String): ApiRequest {
        return PostRequest(endpoint)
    }
}
```

### 5.3 Fragment 생성 팩토리

Navigation Component와 함께 사용하여 Fragment를 생성할 때 활용할 수 있습니다.

```kotlin
abstract class FragmentFactory {
    abstract fun createFragment(args: Bundle?): Fragment
    
    fun createFragmentWithArgs(args: Bundle?): Fragment {
        return createFragment(args)
    }
}

class HomeFragmentFactory : FragmentFactory() {
    override fun createFragment(args: Bundle?): Fragment {
        return HomeFragment().apply {
            arguments = args
        }
    }
}

class ProfileFragmentFactory : FragmentFactory() {
    override fun createFragment(args: Bundle?): Fragment {
        return ProfileFragment().apply {
            arguments = args
        }
    }
}
```

### 5.4 ViewHolder 생성 팩토리

RecyclerView에서 다양한 타입의 ViewHolder를 생성할 때 활용할 수 있습니다.

```kotlin
abstract class ViewHolderFactory {
    abstract fun createViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder
    
    fun createViewHolderWithType(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        return createViewHolder(parent, viewType)
    }
}

class UserViewHolderFactory : ViewHolderFactory() {
    override fun createViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        return when (viewType) {
            USER_TYPE -> UserViewHolder(parent)
            ADMIN_TYPE -> AdminViewHolder(parent)
            else -> throw IllegalArgumentException("알 수 없는 뷰 타입: $viewType")
        }
    }
    
    companion object {
        const val USER_TYPE = 0
        const val ADMIN_TYPE = 1
    }
}
```

---

## 6. 실제 안드로이드 프로젝트에서의 활용

### 6.1 Repository 생성 팩토리

Clean Architecture에서 Repository를 생성할 때 활용할 수 있습니다.

```kotlin
abstract class RepositoryFactory {
    abstract fun createUserRepository(): UserRepository
    abstract fun createPostRepository(): PostRepository
    
    fun createRepository(type: String): Repository {
        return when (type) {
            "user" -> createUserRepository()
            "post" -> createPostRepository()
            else -> throw IllegalArgumentException("알 수 없는 Repository 타입: $type")
        }
    }
}

class LocalRepositoryFactory : RepositoryFactory() {
    override fun createUserRepository(): UserRepository {
        return LocalUserRepository()
    }
    
    override fun createPostRepository(): PostRepository {
        return LocalPostRepository()
    }
}

class RemoteRepositoryFactory : RepositoryFactory() {
    override fun createUserRepository(): UserRepository {
        return RemoteUserRepository()
    }
    
    override fun createPostRepository(): PostRepository {
        return RemotePostRepository()
    }
}
```

### 6.2 ViewModel 생성 팩토리

ViewModel을 생성할 때도 활용할 수 있습니다.

```kotlin
abstract class ViewModelFactory {
    abstract fun createViewModel(type: String): ViewModel
    
    fun createViewModelWithType(type: String): ViewModel {
        return createViewModel(type)
    }
}

class MainViewModelFactory : ViewModelFactory() {
    override fun createViewModel(type: String): ViewModel {
        return when (type) {
            "home" -> HomeViewModel()
            "profile" -> ProfileViewModel()
            "settings" -> SettingsViewModel()
            else -> throw IllegalArgumentException("알 수 없는 ViewModel 타입: $type")
        }
    }
}
```

### 6.3 View 생성 팩토리

안드로이드에서 자주 사용하는 View 컴포넌트들을 생성할 때 활용할 수 있습니다.

```kotlin
enum class ViewType {
    BUTTON, TEXT_VIEW, IMAGE_VIEW, EDIT_TEXT, SWITCH
}

class ViewFactory(private val context: Context) {
    fun createView(type: ViewType): View {
        return when (type) {
            ViewType.BUTTON -> Button(context).apply {
                text = "버튼"
                setOnClickListener { /* 클릭 이벤트 */ }
            }
            ViewType.TEXT_VIEW -> TextView(context).apply {
                text = "텍스트"
                textSize = 16f
            }
            ViewType.IMAGE_VIEW -> ImageView(context).apply {
                scaleType = ImageView.ScaleType.CENTER_CROP
            }
            ViewType.EDIT_TEXT -> EditText(context).apply {
                hint = "입력하세요"
                inputType = InputType.TYPE_CLASS_TEXT
            }
            ViewType.SWITCH -> Switch(context).apply {
                isChecked = false
            }
        }
    }
    
    fun createViewWithCustomization(type: ViewType, customizer: (View) -> Unit): View {
        return createView(type).apply(customizer)
    }
}
```

### 6.4 Adapter 생성 팩토리

RecyclerView의 Adapter를 생성할 때도 활용할 수 있습니다.

```kotlin
abstract class AdapterFactory {
    abstract fun createAdapter(data: List<Any>): RecyclerView.Adapter<*>
    
    fun createAdapterWithData(data: List<Any>): RecyclerView.Adapter<*> {
        return createAdapter(data)
    }
}

class UserAdapterFactory : AdapterFactory() {
    override fun createAdapter(data: List<Any>): RecyclerView.Adapter<*> {
        return when {
            data.isNotEmpty() && data.first() is User -> UserAdapter(data as List<User>)
            data.isNotEmpty() && data.first() is Post -> PostAdapter(data as List<Post>)
            else -> EmptyAdapter()
        }
    }
}
```

### 6.5 Intent 생성 팩토리

Activity 간 이동을 위한 Intent를 생성할 때 활용할 수 있습니다.

```kotlin
class IntentFactory(private val context: Context) {
    fun createIntent(activityClass: Class<*>, extras: Bundle? = null): Intent {
        return Intent(context, activityClass).apply {
            extras?.let { putExtras(it) }
        }
    }
    
    fun createIntentWithFlags(activityClass: Class<*>, flags: Int): Intent {
        return Intent(context, activityClass).apply {
            addFlags(flags)
        }
    }
    
    fun createIntentForResult(activityClass: Class<*>, requestCode: Int): Intent {
        return Intent(context, activityClass).apply {
            putExtra("request_code", requestCode)
        }
    }
}
```
