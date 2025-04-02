# MVP 패턴

> 안드로이드 개발을 하며 MVC 패턴을 활용하기에는 다소 무리가 있습니다. 그의 대안으로 MVP 패턴을 고려할 수 있습니다.  
> MVP는 MVC의 단점을 보완하여 등장했으며, 특히 테스트 용이성과 책임 분리 측면에서 강점을 가집니다.  
> MVP는 View와 Model이 서로 직접 통신하지 않고 Presenter를 통해서만 상호작용하는 구조를 가지고 있습니다.  
> 이 문서에서는 MVP 패턴에 대해 알아보고, 기존 코드에서 MVP로 마이그레이션 해보겠습니다.


## 1. MVP 패턴은 뭔가요?

![MVP 구조 다이어그램](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*avwwp2InWqnl1LVNFjudYg.jpeg)
- **Model - View - Presenter** 로 이루어진 디자인 패턴이며, MVC에서 Controller가 하던 역할을 Presenter가 수행합니다.
- MVC는 Model과 View가 서로 연결되어 있어 의존관계를 갖게 되지만, MVP는 Model과 View가 분리되어 있고 오직 Presenter를 통해 상태나 변화를 전달합니다.
- **MVC는 model이 view를 직접 컨트롤 할 수 있지만, MVP는 직접 컨트롤 할 수 없습니다.**
- **즉, MVP의 Presenter는 MVC의 Controller 역할을 대신 수행합니다.**

Presenter와 View는 일반적으로 1:1 관계를 가지지만, 구현에 따라 Presenter가 여러 View를 참조하거나 공유될 수도 있습니다.
> Android의 Activity는 View와 Controller의 특성을 동시에 지니고 있어,  
> MVP에서는 View 역할만을 맡기고 제어는 Presenter가 담당함으로써 역할 분리를 명확히 할 수 있습니다.

### 구성 요소

#### Model
> 데이터를 담당하는 부분으로, 데이터를 가져오고 저장하는 역할을 수행합니다.  
> 데이터베이스, 네트워크 요청, 파일 시스템 등과 상호 작용하며, 다른 요소에 의존하지 않는 독립적인 영역입니다.  

#### View
> 사용자 인터페이스를 담당하는 부분입니다. Activity나 Fragment가 이에 해당하며, Presenter를 통해 전달받은 데이터를 사용자에게 보여주고 사용자 입력을 Presenter에 전달합니다.
> View는 모델의 데이터를 저장해서는 안 되며, 화면에 표시할 정보만을 책임집니다.
> 실제 view에 대한 직접 접근을 담당하고, 사용자로부터 발생한 이벤트는 **Presenter에 위임**합니다.  

#### Presenter
> Model과 View 사이의 중재자 역할을 하며, View에서 발생한 이벤트를 받아 Model에 요청을 전달하고, 그 결과를 다시 View에 전달합니다.  
> 사용자 입력을 받아 처리하고 화면을 갱신하는 책임을 가지며, 데이터와 UI를 분리해 코드의 유지보수성과 가독성을 향상시킵니다.  
> ViewController 역할. **View와 Model을 연결**하며, View에서 이벤트가 발생하면 Presenter가 처리 후 View에 전달합니다.  

#### Contract
> Contract는 계층 그 자체는 아니며, Presenter와 View 사이 동작을 명시하는 역할을 합니다.  
> Presenter와 View 사이에서 어떤 함수를 사용할지 명확히 정의하는 인터페이스이며, **인터페이스에 의존하도록 하여 구현체 간의 결합도를 낮춥니다.**  
> Contract는 각 구성 요소의 역할을 명확히 하며, 특히 View와 Presenter 간의 통신 규칙을 정의합니다.  

> MVP에서는 Model과 View가 서로 독립적이며, Presenter만이 양쪽을 연결합니다.  
> Presenter는 사용자 입력을 받아 Model에 요청하고, Model의 변경사항을 View에 반영하는 **통역자** 같은 역할을 합니다.  

## 2. MVP의 장단점

| 구분 | 항목 | 설명 |
|------|------|------|
| 장점 | 확장성 | 역할이 분리되어 새로운 기능 추가/변경이 수월하다 |
|      | 작업 명확성 | UI와 비즈니스 로직이 분리되어 개발 범위가 명확하다 |
|      | 재사용성 및 모듈화 | 컴포넌트 간 결합도가 낮고 구조화가 잘 되어 있다 |
|      | 복잡한 UI 처리 용이 | Presenter가 UI 로직을 집중 처리할 수 있다 |
|      | 상태 관리의 명확성 | Presenter에서 상태를 일관되게 처리할 수 있어 상태 변경이 예측 가능함 |
|      | 상태 추적 용이성 | 상태 전이 과정을 Presenter 내에서 추적할 수 있어 디버깅 및 유지보수가 쉬움 |
| 단점 | Fat Presenter | 기능이 많아지면 Presenter가 비대해짐 |
|      | 클래스 증가 | View 수 증가에 따라 Presenter도 함께 증가 |
|      | 추상 계층 증가 | 데이터 흐름 파악이 어려워지고 개발 시간 증가 |

### Fat Presenter는 어떻게 개션할 수 있을까요?

MVP의 대표적인 단점으로 Fat Presenter가 자주 언급됩니다.  
Fat Presenter란, Presenter가 너무 많은 책임을 가지게 되어 하나의 클래스에 UI 로직과 비즈니스 로직이 모두 집중된 상태를 말합니다.  
기능이 많아질수록 Presenter가 비대해지면서 가독성과 유지보수성이 떨어지고, 테스트도 어려워질 수 있습니다.  

Presenter에 UI 로직과 비즈니스 로직이 몰리면 코드가 비대해지고 유지보수가 어려워집니다.  
이를 완화하기 위한 방법으로 다음과 같은 구조 개선이 논의됩니다:

- **UseCase 분리**: 비즈니스 로직을 Presenter 외부로 이동
- **상위 클래스 추출**: 공통 Presenter 로직을 추출해 재사용
- **유틸 클래스 활용**: 반복되는 기능은 별도 모듈화
- **Clean Architecture 도입**: 역할에 따라 책임 명확히 분리

## 3. 예시 코드

### Contract 정의

```kotlin
// Login 기능의 역할을 명확히 분리하기 위한 계약(Contract) 정의
interface LoginContract {

    // View: 사용자에게 보여지는 UI 처리 담당 (Activity, Fragment 등에서 구현)
    interface View {
        fun showLoading()       // 로딩 표시
        fun hideLoading()       // 로딩 숨기기
        fun showLoginSuccess()  // 로그인 성공 UI 표시
        fun showLoginError()    // 로그인 실패 UI 표시
    }

    // Presenter: View의 요청을 받아 비즈니스 로직을 처리하고 결과를 다시 View에 전달
    interface Presenter {
        fun onLoginButtonClicked(username: String, password: String) // 로그인 버튼 클릭 시 호출
    }
}
```

### LoginModel, LoginCallback 정의

```kotlin
interface LoginCallback {
    fun onSuccess()
    fun onError()
}

class LoginModel {
    fun login(username: String, password: String, callback: LoginCallback) {
        if (username == "admin" && password == "1234") {
            callback.onSuccess()
        } else {
            callback.onError()
        }
    }
}
```

### Presenter 구현
```kotlin
class LoginPresenter(
    private val view: LoginContract.View,
    private val model: LoginModel
) : LoginContract.Presenter {

    override fun onLoginButtonClicked(username: String, password: String) {
        view.showLoading()

        model.login(username, password, object : LoginCallback {
            override fun onSuccess() {
                view.hideLoading()
                view.showLoginSuccess()
            }

            override fun onError() {
                view.hideLoading()
                view.showLoginError()
            }
        })
    }
}
```

## 4. MVP 구조로의 전환 예시 (기존 구조 → MVP 구조)

### 기존 코드 (비MVP, Activity에 로직 집중)
```kotlin
class MainActivity : AppCompatActivity() {

    private val binding by lazy {
        ActivityMainBinding.inflate(layoutInflater)
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(binding.root)

        binding.btnConfirm.setOnClickListener {
            binding.btnConfirm.text = getClickedText()
        }
    }

    private fun getClickedText(): String {
        return "Clicked!!!"
    }
}
```

### MVP 패턴으로 리팩토링

**Contract**
```kotlin
interface MainContract {
    interface View {
        fun updateText(text: String)
    }

    interface Presenter {
        fun onConfirmClicked()
    }
}
```

**Model**
```kotlin
class MainModel {
    fun getClickedText(): String = "Clicked!!!"
}
```

**Presenter**
```kotlin
class MainPresenter(
    private val view: MainContract.View,
    private val model: MainModel
) : MainContract.Presenter {

    override fun onConfirmClicked() {
        val text = model.getClickedText()
        view.updateText(text)
    }
}
```

**View (Activity)**
```kotlin
class MainActivity : AppCompatActivity(), MainContract.View {

    private lateinit var presenter: MainContract.Presenter

    private val binding by lazy {
        ActivityMainBinding.inflate(layoutInflater)
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(binding.root)

        val model = MainModel()
        presenter = MainPresenter(this, model)

        binding.btnConfirm.setOnClickListener {
            presenter.onConfirmClicked()
        }
    }

    override fun updateText(text: String) {
        binding.btnConfirm.text = text
    }
}
```

## 5. MVP는 지금도 유효할까요?

MVP는 MVVM, MVI와 같은 아키텍처가 널리 사용되고 있는 현재에도 여전히 의미 있는 패턴입니다.  

- XML 기반 UI를 유지하는 프로젝트에서 UI 로직과 비즈니스 로직을 명확하게 분리하고자 할 경우
- ViewModel이나 Jetpack Compose를 도입하기 어려운 레거시 환경에서
- 테스트 가능성을 우선시하는 팀 구조나 요구 사항이 있을 경우
- 인터페이스 기반으로 테스트 가능한 구조를 선호하는 경우

## 참고 자료

- [안드로이드 어플리케이션에 MVP패턴을 적용하는 방법](https://brunch.co.kr/@mobiinside/913)
- [MVP pattern](https://brunch.co.kr/@oemilk/75)
- [Android Architecture Patterns — MVC, MVP, MVVM, MVI, Clean Architecture](https://medium.com/droidblogs/android-architecture-patterns-mvc-mvp-mvvm-mvi-clean-architecture-cde8029b8f37)
- [buzzvil Tech: Android MVP Pattern - What, Why and How?](https://tech.buzzvil.com/blog/android-mvp-pattern-what-why-and-how/?utm_source=chatgpt.com)
- [JANDI: Android MVC, MVVM, MVP](https://tosslab.github.io/android/2015/03/01/01.Android-mvc-mvvm-mvp)
- [안드로이드 어플리케이션에 MVP패턴을 적용하는 방법](https://brunch.co.kr/@mobiinside/913)

