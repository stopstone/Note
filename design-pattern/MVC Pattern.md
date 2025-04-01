# MVC 패턴

> 현대 안드로이드에서는 MVVM, MVI와 같은 아키텍처 패턴이 많이 활용되지만,  
> 그 근간이 되는 MVC 패턴을 이해하는 것은 중요하다고 생각합니다.  
> 안드로이드에서는 Activity가 View와 Controller 역할을 동시에 맡게 되며 구조적 한계가 생깁니다.  
> 코드가 커지고 기능이 많아질수록 Controller가 비대해져 유지보수가 어려워집니다.  
> 그래서 안드로이드에서는 MVC 대신 MVP와 MVVM, 최근에는 MVI를 더 많이 사용합니다.

안드로이드 개발을 하다보면 Activity/Fragment에 모든 코드가 작성되어 있는 경우가 있습니다..
이 경우 본인도 자신의 코드를 알아보지 못하는데요.
이 과정에서 디자인 패턴의 중요성과 MVC 패턴의 단점을 느끼게 됩니다.

이 문서에서는 **왜 안드로이드에서는 사용하지 않는지**, **어떤 구조와 역할을 가지는지**, **그리고 MVC의 장단점은 무엇인지** 에 대해 설명합니다.  
또, 왜 안드로이드 개발에서는 MVC가 아닌 다른 패턴을 활용하는지 설명합니다.

---

## MVC 패턴이란?
![img1 daumcdn](https://github.com/user-attachments/assets/a6bc1b5a-ec4b-4661-bcd8-4c8399241913)

MVC는 **Model, View, Controller**의 약자로, 하나의 애플리케이션을 세 가지 구성 요소로 분리하여  
**역할 분담**과 **관심사의 분리(Separation of Concerns)** 를 명확히 하려는 디자인 패턴입니다.

### Model
**데이터**와 **비즈니스 로직**을 담당한다.
예: 유저 정보, 상품 가격, 계산 로직, 상태 값 등

### View
사용자에게 보이는 **UI를 구성**하며, 데이터를 시각적으로 표현한다.
사용자 입력 이벤트를 수신하지만, 이를 해석하거나 처리하지 않고 Controller에 전달한다.

### Controller
사용자 입력을 해석하고 적절한 Model을 업데이트한 후, View를 갱신한다.
Model과 View 사이의 흐름을 중재하며, 어플리케이션 로직을 관리한다.

---

## MVC를 왜 사용할까요?

- **역할이 분리되어 가독성과 유지보수성이 좋음**
- **Model은 독립적이기 때문에 테스트가 쉬움**
- **동일한 Model을 다른 View에서 재사용 가능**

---

## 그런데 안드로이드에서는 왜 잘 안 쓸까요?

안드로이드에서 MVC 예시 코드를 보겠습니다.  
아래는 간단한 수량 증가/감소 기능을 MVC 패턴 구조로 구현한 예시입니다.

### Model
```kotlin
class CounterModel {
    var count: Int = 0
        private set

    fun increment() {
        count++
    }

    fun decrement() {
        count--
    }
}
```

### View (activity_main.xml)
```xml
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center">

    <TextView
        android:id="@+id/textViewCount"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="0"
        android:textSize="24sp" />

    <Button
        android:id="@+id/buttonIncrement"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="증가" />

    <Button
        android:id="@+id/buttonDecrement"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="감소" />
</LinearLayout>
```

### Controller (MainActivity)
```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var textViewCount: TextView
    private lateinit var buttonIncrement: Button
    private lateinit var buttonDecrement: Button
    private val model = CounterModel()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        textViewCount = findViewById(R.id.textViewCount)
        buttonIncrement = findViewById(R.id.buttonIncrement)
        buttonDecrement = findViewById(R.id.buttonDecrement)

        buttonIncrement.setOnClickListener {
            model.increment()
            updateView()
        }

        buttonDecrement.setOnClickListener {
            model.decrement()
            updateView()
        }
    }

    private fun updateView() {
        textViewCount.text = model.count.toString()
    }
}
```

이처럼 Model은 상태와 로직만 담당하고,  View는 XML로 UI만 정의하며, Controller는 사용자 입력과 UI 갱신을 모두 담당합니다.   
하지만 실제로는 Controller가 View와 Model 모두에 강하게 의존하게 되어 구조적으로 비대해지기 쉽습니다.  

### 1. Activity/Fragment가 너무 많은 역할을 담당함

안드로이드에서 View는 XML, Controller는 Activity라고 하지만,  
실제로는 **Activity가 View와 Controller 역할을 동시에 수행**하는 경우가 많습니다.

→ 결국 Controller가 View에 강하게 의존하게 되어 **비대해지고 스파게티 코드**가 발생합니다.

```kotlin
binding.button.setOnClickListener {
    model.increment()
    binding.textView.text = model.count.toString()
}
```

- 이 구조에서는 View, Controller, Model의 경계가 모호해짐
- 특히 Controller(Activity)는 Android에 강하게 종속 → 유닛 테스트 어려움

---

## MVC의 장단점

| 항목 | 설명 | 구분 |
|------|------|------|
| 역할 분리 | 각각의 책임이 명확 | 장점 |
| Model 독립성 | 안드로이드에 종속되지 않아 단위 테스트 용이 | 장점 |
| 구현 간단 | 작은 프로젝트에서 빠르게 적용 가능 | 장점 |
| 재사용성 | 같은 Model을 여러 View에서 사용할 수 있음 | 장점 |
| View와 Controller 간 강한 결합 | View의 상태 변경, 이벤트 핸들링을 모두 Controller가 처리 | 단점 |
| Controller의 비대화 | 코드가 점점 몰리면서 유지보수가 어려워짐 | 단점 |
| 안드로이드 종속성 | Controller(Activity)는 Android 클래스라 테스트하기 어려움 | 단점 |
| UI 로직 분리 어려움 | View가 Model을 직접 참조해 의존성 생김 | 단점 |

---

## 참고 자료
[안드로이드에는 MVC 아키텍처 패턴이 없다](https://brunch.co.kr/@mystoryg/211)
[안드로이드 MVC](https://brunch.co.kr/@mystoryg/170)
[안드로이드 Architecture 패턴 Part 1: 모델 뷰 컨트롤러 (Model-View-Controller)](http://medium.com/nspoons/%EC%95%88%EB%93%9C%EB%A1%9C%EC%9D%B4%EB%93%9C-architecture-%ED%8C%A8%ED%84%B4-part-1-%EB%AA%A8%EB%8D%B8-%EB%B7%B0-%EC%BB%A8%ED%8A%B8%EB%A1%A4%EB%9F%AC-model-view-controller-881c6fda24d9)
