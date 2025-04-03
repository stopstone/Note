# DataBinding과 ViewBinding

> 안드로이드 개발에서 UI 요소를 코드와 연결하기 위해 과거에는 `findViewById()`를 사용했습니다.  
> `findViewById()`는 뷰 ID를 기준으로 View 객체를 찾아오는 메서드로,  
> 레이아웃 파일에서 정의한 UI 요소에 접근할 수 있게 해줍니다.  
> 하지만 이 방식은 여러 문제점을 초래하였습니다.  
> 이러한 문제를 해결하고자 등장한 것이 **DataBinding**과 **ViewBinding**입니다.  
> 
> **DataBinding**은 더 선언적이고 유연한 UI 구성을 목표로 먼저 등장한 라이브러리로,  
> XML에서 직접 데이터를 바인딩할 수 있게 하여 MVVM 아키텍처를 구현하기에 적합합니다.  
> ViewBinding은 컴파일 시 타입 안전성을 제공하며, XML의 모든 View ID에 대응하는 바인딩 클래스를 생성해줍니다.  
> 이를 통해 findViewById() 없이도 안정적이고 간결하게 뷰를 참조할 수 있습니다.
> 
> 이 문서에서는 DataBinding과 ViewBinding 각각의 특징과 사용법을 정리하고, 두 개념이 같은 것인지에 대한 의문을 포함해,
> 둘 사이의 차이점 및 어떤 상황에서 어떤 방식을 선택하는 것이 더 적절한지에 대해 살펴보겠습니다.

**뷰 바인딩과 데이터 바인딩을 한 문서에서 함께 다루는 이유는,  
둘 다 `findViewById()`를 대체하고 데이터를 뷰에 연결하는 공통된 목적이 있기 때문입니다.  
하지만 이 둘이 동일한 기술이라는 의미는 아니며, 목적과 사용 방식에 있어 분명한 차이가 존재합니다.**

---

## 1. 바인딩 기술을 사용하는 이유

우선 `findViewById()`를 살펴보겠습니다.  
안드로이드 초기에는 `findViewById()` 메서드를 사용해 XML에 선언된 뷰를 코드에서 참조했습니다.  

### 예시 코드
```kotlin
val textView = findViewById<TextView>(R.id.my_text_view)
textView.text = "Hello, world!"
```

위와 같이 뷰를 찾기 위해 뷰 ID를 직접 참조하고, 타입을 명시해 캐스팅해야 했습니다. 이 방식은 다음과 같은 한계를 갖습니다:

- 뷰 ID 오타 → 컴파일 시점이 아닌 런타임에서 오류 발생
- 잘못된 타입 캐스팅 → ClassCastException 발생 가능성
- 뷰 개수가 많아질수록 코드가 장황하고 관리가 어려워짐

이러한 문제를 해결하고자 Android Jetpack에서는 `ViewBinding`과 `DataBinding`이라는 두 가지 바인딩 기능을 제공합니다.  
이들은 **컴파일 시점에 안전한 뷰 참조를 가능하게 하고**, **UI와 코드 간의 결합을 더 직관적으로 관리**할 수 있게 만들어줍니다.

---

## 2. DataBinding

### 정의
- XML과 데이터를 바인딩할 수 있도록 도와주는 Android Jetpack 라이브러리
- 레이아웃과 애플리케이션 로직을 연결하는 글루 코드(glue code)를 줄여줌
- findViewById를 사용하지 않아도 View에 접근 가능
- MVVM 패턴과 잘 어울리며, LiveData와 함께 사용하기 적합

### 장점
- 선언적 방식으로 UI 요소와 데이터를 연결해 코드 간결
- 삼항 연산자 등 표현식 사용 가능 (예: `@{isClicked ? View.GONE : View.VISIBLE}`)
- NullPointerException 방지
- MVVM 아키텍처 구현에 유리
- 레이아웃 변경 감지 및 자동 UI 업데이트 가능

### 단점
- 빌드 속도 및 앱 크기 증가: ViewBinding보다 컴파일 시간이 길고, 바인딩 클래스 생성으로 앱 용량이 증가할 수 있음.
- 디버깅 어려움: XML 표현식 오류가 런타임에 발생하고, 에러 메시지가 직관적이지 않음.
- KAPT 의존 및 유지보수 상태: 현재 구글은 새로운 프로젝트에서는 ViewBinding 사용을 권장하며, DataBinding은 유지보수만 진행되고 있습니다.  
- 높은 결합도: XML에서 ViewModel 속성에 직접 접근해 관심사 분리가 어려울 수 있음.

### 사용법
1. **build.gradle 설정**
```kotlin
android {
    buildFeatures {
        dataBinding = true
    }
    // 또는
    dataBinding {
        enabled = true
    }
}
```

2. **XML 레이아웃 구성**
- 최상단에 `<layout>` 태그 사용
- `<data>` 내부에 바인딩할 변수 선언

3. **바인딩 객체 생성 및 사용**
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
}

```

4. **데이터 바인딩 예제**
```kotlin
binding.activity = this
binding.btnChange.setOnClickListener {
    counter += 1
    // invalidateAll()은 전체 UI를 리바인딩하기 때문에 성능상 비효율적일 수 있음
    // 실제 프로젝트에서는 LiveData, ObservableField 등으로 구체적인 뷰만 갱신하는 것이 권장됨
    binding.invalidateAll()
}
```

**XML 예시:**
```xml
<TextView
    android:text="@{String.valueOf(activity.counter)}"
        <!-- String.valueOf는 null-safe. null일 경우 문자열 'null' 반환. -->
        <!-- toString()은 null일 경우 NPE 발생 --> />
```

---

## 3. ViewBinding

### 정의
- findViewById를 대체하는 Jetpack 기능
- 각 XML 레이아웃에 대해 바인딩 클래스를 자동 생성
- View ID에 직접 접근 가능

### 장점
- 빠른 컴파일 시간
- 사용법이 간단하고 도입이 쉬움
- Null 안전성 보장
- View에 직접 접근 가능 (타입 안전성)

### 단점
- 표현식/변수 미지원: XML에서 조건식, 연산, 바인딩 변수 등을 사용할 수 없음.
- 양방향 바인딩 미지원: View와 데이터 간 실시간 동기화는 직접 구현해야 함.

### ViewBinding과 MVVM
**ViewBinding도 MVVM 아키텍처에서 충분히 사용할 수 있습니다.**  
다만 View와 ViewModel 간의 연결을 직접 XML에서 표현하지 않고, Activity나 Fragment에서 ViewModel의 데이터를 관찰하고 바인딩하는 로직을 명시적으로 구현해야 합니다.

```kotlin
viewModel.text.observe(viewLifecycleOwner) { text ->
    binding.textView.text = text
}
```

이처럼 ViewModel의 데이터를 관찰하고 뷰에 반영하는 방식으로 MVVM 패턴을 구현할 수 있으며, ViewBinding은 뷰 참조를 안전하게 도와주는 역할을 수행합니다.

### viewBindingIgnore 속성 (불필요한 바인딩 방지 팁)
Splash, Dialog처럼 바인딩 클래스 생성을 원하지 않는 경우 XML 루트 뷰에 `tools:viewBindingIgnore="true"` 속성을 추가할 수 있습니다.
```xml
<LinearLayout
    xmlns:tools="http://schemas.android.com/tools"
    tools:viewBindingIgnore="true"
    ... >
</LinearLayout>
```

### 사용법
1. **build.gradle 설정**
```kotlin
android {
    buildFeatures {
        viewBinding = true
    }
    // 또는
    viewBinding {
        enabled = true
    }
}
```

2. **바인딩 객체 생성 및 사용**
```kotlin
private val binding: ActivityMainBinding by lazy {
    ActivityMainBinding.inflate(layoutInflater)
}

setContentView(binding.root)
```

3. **카운터 예제**
```kotlin
var counter = binding.tvCounter.text.toString().toInt()
binding.btnCounterUp.setOnClickListener {
    binding.tvCounter.text = "${++counter}"
}
```

---

## 4. 비교
| 항목 | DataBinding | ViewBinding |
|------|-------------|--------------|
| 목적 | 데이터와 뷰의 바인딩 | findViewById 대체 |
| XML 표현식 | 지원 (@{}) | 미지원 |
| 양방향 바인딩 | 가능 | 불가능 |
| MVVM 연계 | 적합 | 제한적 |
| 빌드 속도 | 느릴 수 있음 | 빠름 |
| 코드 간결성 | 높음 | 중간 |
| 사용 난이도 | 비교적 높음 | 낮음 |

---

## 5. 바인딩 객체 메모리 누수 방지 팁
Fragment의 생명주기는 View보다 길기 때문에, ViewBinding이나 DataBinding 모두 `onDestroyView()`에서 binding 객체를 null로 해제해주는 것이 중요합니다.  
그렇지 않으면 View를 계속 참조하게 되어 메모리 누수가 발생할 수 있습니다.

```kotlin
private var _binding: FragmentSampleBinding? = null
private val binding get() = _binding!!

override fun onCreateView(...): View {
    _binding = FragmentSampleBinding.inflate(inflater, container, false)
    return binding.root
}

override fun onDestroyView() {
    super.onDestroyView()
    _binding = null
}
```


## 7. 참고 자료
- [공식 문서 - ViewBinding](https://developer.android.com/topic/libraries/view-binding)
- [공식 문서 - DataBinding](https://developer.android.com/topic/libraries/data-binding)
- [안드로이드 Databinding](https://brunch.co.kr/@mystoryg/150)
- [Use view binding to replace findViewById](https://medium.com/androiddevelopers/use-view-binding-to-replace-findviewbyid-c83942471fc)

