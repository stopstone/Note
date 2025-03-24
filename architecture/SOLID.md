# SOLID 원칙

SOLID는 객체지향 설계에서 지켜야 할 5가지 핵심 원칙의 앞글자를 따 만든 용어입니다.  
이 원칙들을 지키면 유지보수성과 확장성이 높은, 유연한 소프트웨어를 만들 수 있습니다.  
또한 클린 아키텍처의 핵심이 되며, 면접에서도 단골 질문으로 자주 등장합니다.

---

## SOLID란?
- **S**: SRP - 단일 책임 원칙 (Single Responsibility Principle)
- **O**: OCP - 개방-폐쇄 원칙 (Open/Closed Principle)
- **L**: LSP - 리스코프 치환 원칙 (Liskov Substitution Principle)
- **I**: ISP - 인터페이스 분리 원칙 (Interface Segregation Principle)
- **D**: DIP - 의존 역전 원칙 (Dependency Inversion Principle)

> SOLID는 기법(Technique)이 아닌 원칙(Principle)입니다.   
> 각 원칙은 객체지향 설계를 더 견고하고 유연하게 만들기 위한 방향성을 제시하며,  
> 그 자체가 목표가 되기보단 더 나은 아키텍처를 위한 지침으로 받아들이는 것이 중요합니다.

---

## SOLID 원칙을 따르면 어떤 점이 더 좋을까요?

SOLID 원칙은 단순한 이론이 아닌, 실무에서 유연하고 이해하기 쉬운 구조를 만들기 위한 **가이드라인**입니다.

1. **변경에 유연한 구조**: 결합도는 낮고 응집도는 높은 구조로, 기능 변경 시 다른 코드에 영향을 최소화할 수 있습니다.   
2. **이해하기 쉬운 구조**: 가독성이 높은 코드 구조로 협업과 유지보수가 쉬워집니다.
3. **재사용 가능한 컴포넌트**: 명확히 분리된 책임 덕분에 코드 재사용성이 높아집니다.
4. **확장 가능한 설계**: 새로운 기능 추가 시 기존 코드를 수정하지 않고도 확장 가능합니다.
5. **테스트가 쉬운 구조**: 단일 책임을 가진 모듈은 테스트가 간단하고 명확합니다.
6. **기술 부채 예방**: 책임과 관심사를 분리하면 기능 확장이 반복되더라도 코드의 복잡도가 증가하지 않습니다.   

이처럼 SOLID 원칙은 결국, "변화에 강한 코드"를 만드는 데 목적이 있습니다.

---

## 1. 단일 책임 원칙 (SRP)
> 클래스는 하나의 책임만 가져야 하며, 변경의 이유도 하나뿐이어야 합니다.

### 핵심
- 하나의 모듈은 오직 하나의 변경 이유만 가져야 합니다.
- 여기서 '책임'은 단순히 '동작'이 아니라 **이해관계자(Actor)의 책무 단위**를 의미합니다. 예: 기획, 디자인, 개발 등
- 각자의 '책임' 정의는 주관적일 수 있으므로, "변경의 이유가 하나인가?"를 기준으로 판단합니다.

### 면접 예시 대화 (출처: 카카오뱅크 기술블로그)
```text
👨‍🦳 면접관: SRP에 대해서 설명 부탁드립니다.
👨🏻 지원자: SRP는 단일 책임 원칙으로서, 클래스를 만들 때 책임을 하나만 가져야 한다는 원칙입니다.
👨‍🦳 면접관: 좋은 답변 감사합니다. 만약 개발자 A, B가 생각하는 책임의 크기가 각각 다를 때는 어떻게 해야 할까요?
👨🏻 지원자: 서로 논의를 통해서 책임을 하나로 맞추면 될 것 같습니다.
👨‍🦳 면접관: 그럼 팀원이 10명이라고 했을 때, 10명의 논의를 통해 나온 결론이 명확하게 1가지의 책임을 가지고 있다고 할 수 있을까요?
👨🏻 지원자: 논의를 통해 결정했기 때문에 지켜야 하지만, 책임은 상황에 따라 다를 것 같습니다.
👨‍🦳 면접관: ...
```
→ 이 대화를 통해 '책임'이 매우 주관적일 수 있다는 점을 알 수 있으며, SRP의 판단 기준은 **책임의 수**가 아니라 **변경의 이유**가 하나인지로 파악하는 것이 더 정확합니다.

### 안드로이드 위반 사례
```kotlin
class MainActivity : AppCompatActivity() {
    fun onCreate() {
        super.onCreate()
        fetchData()
    }

    fun fetchData() {
        val data = apiService.getData() // 네트워크 요청
        val processedData = processData(data) // 데이터 처리
        updateUI(processedData) // UI 업데이트
    }
}
```
→ UI, 네트워크, 데이터 가공까지 하나의 클래스에 몰려 있어 SRP를 위반함

### 안드로이드 해결 사례
```kotlin
class DataRepository(private val apiService: ApiService) {
    fun fetchData(): Data {
        return apiService.getData()
    }
}

class MainViewModel(private val dataRepository: DataRepository) : ViewModel() {
    val data = MutableLiveData<Data>()

    fun loadData() {
        data.value = dataRepository.fetchData()
    }
}

class MainActivity : AppCompatActivity() {
    private val viewModel: MainViewModel by viewModels()

    fun onCreate() {
        super.onCreate()
        viewModel.data.observe(this) { data ->
            updateUI(data)
        }
        viewModel.loadData()
    }
}
```
→ Activity는 UI에만 집중하고, ViewModel은 데이터 처리, Repository는 네트워크를 담당하여 SRP를 만족시킴

### SRP 요약 - 클린 아키텍처 관점
- 각 계층 (데이터, 도메인, 프레젠테이션)은 각자의 책임만 가짐
- 계층 간 결합도를 낮추고 유지보수성을 높임
- 변경이 다른 계층에 영향을 주지 않음 → 단일 책임 원칙의 실현

---

## 2. 개방-폐쇄 원칙 (OCP)
> 확장에는 열려 있고, 수정에는 닫혀 있어야 합니다.

### 핵심
- 새로운 기능이 추가될 때, **기존 코드를 변경하지 않고 확장**할 수 있어야 합니다.
- 코드의 유연성과 확장성을 보장하여, 변경이나 새로운 요구사항이 생겨도 기존 코드에 영향을 주지 않음
- 대표적인 구현 방식: 전략 패턴, 템플릿 메서드 패턴, 의존성 주입(DI)

### 안드로이드 위반 사례
```kotlin
class ItemAdapter : RecyclerView.Adapter<RecyclerView.ViewHolder>() {
    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        when (holder) {
            is TextViewHolder -> holder.bind(textData[position])
            is ImageViewHolder -> holder.bind(imageData[position])
        }
    }
}
```
→ 새로운 타입이 생길 때마다 `when` 분기를 계속 추가해야 하므로 OCP를 위반함

### 안드로이드 해결 사례
```kotlin
abstract class BaseViewHolder<T>(itemView: View) : RecyclerView.ViewHolder(itemView) {
    abstract fun bind(data: T)
}

class TextViewHolder(itemView: View) : BaseViewHolder<String>(itemView) {
    override fun bind(data: String) {
        // 텍스트 데이터 바인딩
    }
}

class ImageViewHolder(itemView: View) : BaseViewHolder<Image>(itemView) {
    override fun bind(data: Image) {
        // 이미지 데이터 바인딩
    }
}

class ItemAdapter<T>(
    private val items: List<T>,
    private val viewTypeToViewHolder: Map<Int, BaseViewHolder<T>>
) : RecyclerView.Adapter<BaseViewHolder<T>>() {
    override fun onBindViewHolder(holder: BaseViewHolder<T>, position: Int) {
        holder.bind(items[position])
    }
}
```
→ 새로운 ViewHolder 타입을 추가해도 기존 `Adapter` 로직을 수정하지 않음

### OCP 요약 - 클린 아키텍처 관점
- 유스케이스(Use Case) 중심 설계 → 기능이 바뀌지 않아도 요구사항 대응 가능
- 새로운 기능은 기존 코드를 확장하는 방식으로 추가
- 새로운 데이터 소스가 생겨도 기존 코드를 수정하지 않고 Repository 인터페이스 확장 또는 새로운 구현체만 추가

### 면접 상황 예시 (출처: 카카오뱅크 기술블로그)
👨‍🦳 면접관: OCP에 대해 설명 부탁드립니다.

👩🏼‍🦰 지원자: OCP는 개방 폐쇄 원칙으로 기존 수정에는 닫혀 있고, 확장에는 열려 있어야 한다는 원칙입니다.

👨‍🦳 면접관: 답변 감사합니다. 어떻게 하면 기존의 코드를 수정하지 않고 기능을 추가할 수 있을까요?

👩🏼‍🦰 지원자: ...

→ 기존 코드 변경 없이 확장을 위한 구조 설계가 되어 있어야 함 (전략 패턴, DI, 템플릿 메서드 패턴 등)

---

## 3. 리스코프 치환 원칙 (LSP)
> 하위 타입은 상위 타입을 대체할 수 있어야 한다.

### 핵심
- LSP는 객체지향 설계의 **다형성(polymorphism)** 을 보장하는 원칙입니다.
- 상위 클래스의 객체가 사용되는 모든 곳에서, 하위 클래스의 객체도 **문제 없이 동작**해야 함을 의미합니다.
- 상위 타입의 행위를 변경하거나 제한하지 않고, **일관된 동작을 보장**해야 합니다.
- 클린 아키텍처에서는 계층 간 안정적인 인터페이스를 통해 이 원칙을 지킴

### 안드로이드 위반 사례
```kotlin
open class BaseViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
    open fun bind(data: Image) {
        // 기본 데이터 바인딩
    }
}

class TextViewHolder(itemView: View) : BaseViewHolder(itemView) {
    override fun bind(data: Image) {
        throw UnsupportedOperationException("TextViewHolder는 이미지 바인딩을 할 수 없습니다.")
    }
}
```
→ 부모 클래스의 동작을 변경하거나 예외를 던지는 구현은 LSP를 위반하게 됨

### 안드로이드 해결 사례
```kotlin
abstract class BaseViewHolder<T>(itemView: View) : RecyclerView.ViewHolder(itemView) {
    abstract fun bind(data: T)
}

class TextViewHolder(itemView: View) : BaseViewHolder<String>(itemView) {
    override fun bind(data: String) {
        // 텍스트 데이터 바인딩
    }
}

class ImageViewHolder(itemView: View) : BaseViewHolder<Image>(itemView) {
    override fun bind(data: Image) {
        // 이미지 데이터 바인딩
    }
}
```
→ 제네릭을 활용해 타입에 맞게 분리함으로써 부모의 기능을 해치지 않고 일관성 있게 동작

### LSP 요약 - 클린 아키텍처 관점
- Domain Layer에서 정의한 인터페이스는 Data Layer에서 다양한 방식으로 구현 가능해야 함
- 인터페이스와 구현체는 **동일한 동작 보장** 필요
- 예: Repository 구현체(Remote/Local)가 각각 다르더라도 Domain/Presentation 계층은 **동일한 방식으로 사용할 수 있어야 함**

### 보완된 면접 상황 예시 (출처: 카카오뱅크 기술블로그)
👨‍🦳 면접관: LSP에 대해 설명해 주시겠어요?

👨🏻‍💻 지원자: 리스코프 치환 원칙은 하위 클래스가 상위 클래스를 대체할 수 있어야 한다는 원칙입니다.

👨‍🦳 면접관: 그럼 어떤 상황에서 이 원칙이 깨질 수 있을까요?

👨🏻‍💻 지원자: 상위 클래스에서 정의된 메서드를 하위 클래스가 예외를 던지거나, 기대와 다른 방식으로 구현하면 LSP를 위반하게 됩니다.

👨‍🦳 면접관: 예시를 하나 들어주실 수 있나요?

👨🏻‍💻 지원자: RecyclerView에서 ViewHolder를 상속할 때, 특정 ViewHolder에서 부모의 메서드를 override 하면서 다른 타입에 대해 예외를 던지면 문제가 됩니다. 이런 경우엔 제네릭 타입을 활용하거나 추상 클래스를 분리해서 해결할 수 있습니다.

---

## 4. 인터페이스 분리 원칙 (ISP)
> 클라이언트는 자신이 사용하지 않는 인터페이스에 의존하지 않아야 한다.

### 핵심
- ISP는 인터페이스를 **작고 명확하게 분리**하여, 불필요한 메서드 의존을 방지하는 원칙입니다.
- 클라이언트는 자신에게 **필요한 기능만 제공하는 인터페이스에 의존**해야 하며,
  그렇지 않으면 변경의 영향 범위가 커지고 유지보수가 어려워집니다.

### 안드로이드 위반 사례
```kotlin
interface ItemTouchListener {
    fun onClick()
    fun onLongClick()
    fun onSwipe()
}

class SimpleItem : ItemTouchListener {
    override fun onClick() { /* 클릭 처리 */ }
    override fun onLongClick() { /* 사용하지 않음 */ }
    override fun onSwipe() { /* 사용하지 않음 */ }
}
```
→ 사용하지 않는 기능까지 강제로 구현해야 하므로 ISP 위반

### 안드로이드 해결 사례
```kotlin
interface Clickable {
    fun onClick()
}

interface Swipeable {
    fun onSwipe()
}

class SimpleItem : Clickable {
    override fun onClick() { /* 클릭 처리 */ }
}
```
→ 필요한 인터페이스만 구현하여 관심사의 분리 및 책임 최소화

### ISP 요약 - 클린 아키텍처 관점
- 각 layer는 자신이 **필요한 기능만 제공하는 인터페이스**에 의존해야 함
- 변경의 영향 범위를 최소화하고, 시스템의 유연성과 유지보수를 향상
- 구체적인 기능별로 인터페이스를 나누어, 불필요한 메서드 구현을 방지

### 보완된 면접 상황 예시 (출처: 카카오뱅크 기술블로그)
👨‍🦳 면접관: ISP에 대해 설명 부탁드립니다.

👨🏽‍🦱 지원자: ISP는 인터페이스 분리 원칙으로, 클라이언트가 사용하지 않는 메서드에 의존하지 않아야 한다는 원칙입니다.

👨‍🦳 면접관: 네, 인터페이스를 어떻게 잘 분리할 수 있나요?

👨🏽‍🦱 지원자: 기능 및 책임에 따라 구체적으로 작게 나누고, 필요한 기능만 제공하는 인터페이스에 의존하게 해야 합니다. 예를 들어, 안드로이드에서 클릭과 스와이프 기능이 분리된 인터페이스를 제공하면, 사용자는 필요한 기능만 구현할 수 있습니다.

---

## 5. 의존성 역전 원칙 (DIP)
> 고수준 모듈은 저수준 모듈에 의존해서는 안 되며, **둘 다 추상화에 의존해야 한다.**

### 핵심
- DIP는 **상위 계층(고수준 모듈)**이 **하위 계층(저수준 모듈)**의 구현에 의존하지 않도록 하여, **계층 간 의존성을 역전시키는 원칙**입니다.
- 의존성을 **구현체가 아닌 추상화(인터페이스 등)에 두어**, 변경에 유연하고 테스트 가능한 구조를 만듭니다.

### 안드로이드 위반 사례
```kotlin
class UserRepository {
    private val localDataSource = LocalDataSource()
    private val remoteDataSource = RemoteDataSource()

    fun getUser(): User {
        return if (localDataSource.hasUser()) {
            localDataSource.getUser()
        } else {
            remoteDataSource.getUser()
        }
    }
}
```
→ Repository가 직접 구현체(LocalDataSource, RemoteDataSource)에 의존하고 있어 DIP 위반

### 안드로이드 해결 사례
```kotlin
interface UserDataSource {
    fun getUser(): User
}

class LocalDataSource : UserDataSource {
    override fun getUser(): User = User("Local User", 25)
}

class RemoteDataSource : UserDataSource {
    override fun getUser(): User = User("Remote User", 30)
}

class UserRepository(
    private val localDataSource: UserDataSource,
    private val remoteDataSource: UserDataSource
) {
    fun getUser(): User {
        return if ((localDataSource as LocalDataSource).hasUser()) {
            localDataSource.getUser()
        } else {
            remoteDataSource.getUser()
        }
    }
}
```
→ 인터페이스를 통해 추상화에 의존하여 DIP 준수

### DIP 요약 - 클린 아키텍처 관점
- **상위 계층이 하위 계층의 구현에 의존하지 않도록**, 계층 간 의존성을 **인터페이스로 추상화**
- 이를 통해 **테스트 용이성** 증가 및 **구현체 교체가 유연해짐**
- DIP는 Repository나 DataSource와 같은 하위 계층 클래스를 추상화하고, **DI 프레임워크(Dagger, Hilt 등)**를 통해 주입하는 구조에서 자주 적용됨

### 보완된 면접 상황 예시 (출처: 카카오뱅크 기술블로그)
👨‍🦳 면접관: DIP에 대해서 설명 부탁드립니다.

👱🏻‍♀️ 지원자: DIP는 의존성 역전 원칙으로, 고수준 모듈이 저수준 모듈의 구현에 의존하지 않고, **둘 다 추상화에 의존해야 한다**는 원칙입니다.

👨‍🦳 면접관: 좋은 답변 감사합니다. 왜 의존성을 역전시켜야 할까요?

👱🏻‍♀️ 지원자: 구현에 대한 의존을 제거하면 변경에 더 유연해지고, 테스트가 쉬워지며, 유지보수성이 높아집니다. 안드로이드에서는 인터페이스 추상화와 DI 프레임워크를 함께 활용해 DIP를 잘 실현할 수 있습니다.

---

## SOLID 원칙, 꼭 지켜야 할까?

### 언제 SOLID를 지켜야 할까요?

**코드가 자주 변경되거나 확장되어야 할 때**  
→ 유지보수성과 유연성을 위해 원칙이 큰 도움이 됩니다.

**협업 중이거나 팀이 크다면**  
→ 명확하게 책임이 분리된 구조는 팀 커뮤니케이션 비용을 줄여줍니다.

**비즈니스 로직이 복잡해질 때**  
→ SRP, OCP 등은 로직을 분리하고 테스트하기 쉽게 만들어 줍니다.

---

### 언제 SOLID가 오히려 불편할 수 있을까요?

**너무 작은 프로젝트일 때**  
→ 과한 추상화와 인터페이스 분리는 오히려 복잡도를 높일 수 있습니다.

**실용보다 형식을 우선할 때**  
→ DIP를 무조건 적용하려다 불필요한 DI 구조나 추상화로 인해 가독성이 떨어질 수 있습니다.

**개발 속도가 가장 중요한 상황**  
→ MVP나 프로토타이핑 단계에서는 속도와 단순함이 더 중요할 수 있습니다.

---

### 어떻게 접근하면 좋을까요?

> SOLID는 언제든 깨도 된다. 다만 왜 깨는지, 왜 지키는지를 스스로 설명할 수 있어야 한다.

## 참고 자료
- [패스트캠퍼스 클린 아키텍처 강의](https://fastcampus.co.kr/dev_online_clean)
- [카카오 FE 기술블로그](https://fe-developers.kakaoent.com/2023/230330-frontend-solid/)
- [카카오뱅크 기술블로그](https://tech.kakaobank.com/posts/2411-solid-truth-or-myths-for-developers/)
- [F-Lab 기술블로그](https://f-lab.kr/insight/understanding-solid-principles)
