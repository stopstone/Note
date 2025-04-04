# LiveData

LiveData는 수명 주기를 인식하는 관찰 가능한 데이터 홀더 클래스입니다.

![Untitled (1)](https://github.com/user-attachments/assets/bf9d248a-8344-4772-9524-ec2d415e8f15)

> 실생활에서 예시를 찾아봅시다. 학생들이 선생님 몰래 돈내기를 하고 있습니다.  
> 한 학생은 선생님이 오시는지 살피는 감시자 역할을 하고 있네요.    
> 이처럼 한 학생이 선생님이 오기를 관찰하다가, 선생님이 오는 순간 다른 친구들에게 빠르게 알려줍니다.    
> 알림을 받은 친구들은 즉시 하던 일을 멈추고 정리합니다.
>
> - 관찰자(Observer): 학생 → UI
> - 상태(State): 선생님 등장 여부 → 데이터의 변화
> - 알림(Notification): 변화 감지 후 학생에게 알리는 행동 → LiveData가 UI에 알림을 전달
> 
> 이처럼 특정 대상을 지속적으로 관찰하고, 변화가 감지되었을 때 알림을 통해 대응하는 구조는 소프트웨어에서도 자주 사용됩니다.    
> Android에서는 이러한 '관찰자(Observer)' 패턴을 시스템적으로 지원하기 위해 LiveData라는 컴포넌트를 제공합니다.    
> LiveData는 데이터 변화를 감지하고, 이를 관찰 중인 UI 컴포넌트(Observer)에게 안전하게 알려주는 역할을 합니다.    
> **특히 Lifecycle을 인식**하기 때문에, 화면이 종료되었거나 일시정지된 상태에서는 자동으로 관찰을 중단하여 메모리 누수나 앱 크래시를 방지합니다.  

---

## 1. LiveData의 장점

- **UI와 데이터 상태의 일치 보장**: 데이터가 변경되면 Observer 객체에 알리고, 이를 통해 UI가 자동으로 갱신됩니다.
- **메모리 누수 없음**: 관찰자가 LifecycleOwner에 묶여 수명 주기가 끝나면 자동으로 제거됩니다.
- **중지된 활동으로 인한 비정상 종료 없음**: 비활성 수명 주기 상태에서는 이벤트를 수신하지 않습니다.
- **수명 주기를 더 이상 수동으로 처리하지 않음**: 관찰자가 수명 주기를 자동 인식합니다.
- **최신 데이터 유지**: 비활성화됐다가 활성화되면 최신 데이터를 수신합니다.
- **적절한 구성 변경 지원**: 화면 회전 후에도 최신 데이터 유지.
- **리소스 공유**: 시스템 서비스 등을 싱글톤 패턴으로 LiveData를 통해 공유할 수 있습니다.
- **데이터 바인딩과의 통합**: DataBinding과 함께 사용하면 LiveData를 observe 호출 없이 XML에 바로 연결할 수 있어 코드 양을 줄이고 UI 업데이트를 간편하게 할 수 있습니다.

---

## 2. LiveData 객체 사용 방법

1. ViewModel 클래스 내에서 `LiveData` 인스턴스를 생성합니다.
2. `Observer` 객체를 생성하고 `onChanged()` 메서드를 정의합니다.
3. `observe()` 메서드를 사용해 LifecycleOwner와 Observer를 연결합니다.

---

## 3. LiveData를 활용한 간단한 카운터 예시

**CounterViewModel.kt**
```kotlin
class CounterViewModel : ViewModel() {
    private val _counter: MutableLiveData<Int> = MutableLiveData(0)
    val counter: LiveData<Int> get() = _counter

    fun incrementCounter() {
        _counter.value = (_counter.value ?: 0) + 1
    }
}
```

**MainActivity.kt**
```kotlin
class MainActivity : AppCompatActivity() {
    private val viewModel: CounterViewModel by viewModels()
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)

        viewModel.counter.observe(this) { value ->
            binding.tvCount.text = value.toString()
        }

        binding.btn.setOnClickListener {
            viewModel.incrementCounter()
        }
    }
}
```

**activity_main.xml**
```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>
        <variable
            name="viewModel"
            type="com.stopstone.livedatapractice.CounterViewModel" />
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">

        <TextView
            android:id="@+id/tv_count"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{String.valueOf(viewModel.counter)}"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toEndOf="parent"/>

        <Button
            android:id="@+id/btn"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_margin="24dp"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintBottom_toBottomOf="parent"
            android:text="Increment"/>

    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```

---

## 4. LiveData 세부 특징

### 4.1 MutableLiveData vs LiveData
- `MutableLiveData`는 값의 읽기/수정 모두 가능.
- `LiveData`는 읽기 전용 (ViewModel 외부에 수정 불가).
- **단방향 데이터 흐름(Unidirectional Data Flow)** 구현을 돕는다.

### 4.2 setValue vs postValue
- `setValue()`: MainThread에서 즉시 값 업데이트.
- `postValue()`: 다른 스레드에서 값을 업데이트 요청하고 MainThread에서 반영.
- `postValue()`는 호출이 빠르게 이어질 경우 최신 값만 반영될 수 있다.

### 4.3 항상 최신 데이터만 유지
- LiveData는 마지막으로 설정된 데이터 하나만 보존.
- 연속된 여러 이벤트를 처리해야 할 경우 별도 설계 필요.

### 4.4 LiveData는 UI 업데이트용
- LiveData는 내부적으로 항상 MainThread에서 값 변경 및 전달을 처리한다.
- Repository, UseCase, DataSource 등 백엔드나 IO 처리 레이어에서는 LiveData를 사용하는 것은 권장되지 않는다.
- 이러한 경우에는 Coroutine Flow, RxJava 등 다른 비동기 스트림 라이브러리를 사용하는 것이 적절하다.

### 4.5 LiveData는 단일 값 보존
- LiveData는 이벤트 스트림을 관리하거나 연속된 데이터 처리를 지원하지 않는다.
- 연속된 값 변화를 다루려면 별도의 큐, Channel, StateFlow 등을 사용해야 한다.

### 4.6 LiveData 내부 구조
- `_data`: 현재 값.
- `_pendingData`: postValue 호출 시 임시 저장하는 값.
- `postValue()`는 비동기 처리 후 `setValue()` 호출 구조를 가진다.

### 4.7 Lifecycle 연결 구조
- `LifecycleBoundObserver` 클래스를 통해 LifecycleOwner의 상태를 감시.
- STARTED 또는 RESUMED 상태일 때만 옵저버에게 데이터 알림.
- LifecycleOwner가 DESTROYED 상태가 되면 옵저버가 자동으로 제거되어 추가적인 리소스 정리가 필요 없다.

### 4.8 LiveData의 한계 및 보완 방법
- 이벤트 스트림을 지원하지 않는다.
- 단일 값만 보존 → 여러 이벤트를 처리하려면 `SingleLiveEvent`나 `Event Wrapper` 패턴 필요.
- LiveData는 최신 하나의 상태만 보유하기 때문에, 여러 개의 이벤트가 연속적으로 발생할 경우 중간 이벤트를 잃어버릴 가능성이 높습니다.
- 따라서 이벤트가 연속적으로 중요할 때는 SharedFlow나 Channel과 같은 Flow 계열이 더욱 적합합니다.

### 4.9 이제는 Flow?

최근 Android 개발 트렌드에서는 Coroutine 기반의 `Flow`, 특히 `StateFlow`와 `SharedFlow`가 LiveData의 대안으로 떠오르고 있습니다.  
비동기 데이터 처리와 복잡한 이벤트 흐름을 더 명확하게 처리할 수 있기 때문입니다.

- **StateFlow**: 상태 관리에 특화되어 있으며, 최신 상태를 유지하고 항상 현재 값을 방출합니다.
- **SharedFlow**: 여러 이벤트를 놓치지 않고 처리하는 데 효과적이며, 연속적인 이벤트를 처리할 때 적합합니다.

이처럼, LiveData의 한계를 느끼고 있다면 Flow 기반의 솔루션으로 전환을 고민해볼 필요가 있습니다.

---

## 5. Fragment에서 LiveData를 사용할 때 주의사항

### 5.1 Fragment Lifecycle vs Fragment View Lifecycle
- **Fragment Lifecycle**은 Fragment 객체의 생성 ~ 소멸까지를 관리.
- **Fragment View Lifecycle**은 Fragment의 View가 생성 ~ 소멸될 때까지만 관리.
- LiveData 관찰자는 **Fragment View Lifecycle**에 맞춰야 한다.

### 5.2 LiveData observe 시 ViewLifecycleOwner 사용
- 잘못된 예시 (Fragment 자체를 LifecycleOwner로 사용)

```kotlin
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    viewModel.counter.observe(this) { count ->
        binding.tvCount.text = count.toString()
    }
}
```

- 올바른 예시 (Fragment의 ViewLifecycleOwner를 사용)

```kotlin
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    viewModel.counter.observe(viewLifecycleOwner) { count ->
        binding.tvCount.text = count.toString()
    }
}
```

---

## 참고자료

- [Android Developers - LiveData 공식 문서](https://developer.android.com/topic/libraries/architecture/livedata)
- [H43RO 블로그 - LiveData 알고 사용하기](https://velog.io/@haero_kim/Android-LiveData-%EC%95%8C%EA%B3%A0-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)
- [해리의 유목코딩 블로그 - Jetpack Android LiveData](https://medium.com/@harrythegreat/jetpack-android-livedata-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0-fff87d7e56cb)
- [Pluu Dev - Fragment Lifecycle과 LiveData](https://pluu.github.io/blog/android/2020/01/25/android-fragment-lifecycle/)

