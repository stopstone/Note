## View의 visibility 속성

> 안드로이드 개발에서 View의 표시 여부를 제어할 때 사용하는 가장 기본적인 속성이 바로 `visibility`입니다.  
> 이 속성은 단순히 보이고 안 보이는 것 이상의 의미를 가지며, **레이아웃 구성, 뷰 재사용, 성능 최적화** 등과도 밀접한 관련이 있습니다.  

> 이 글에서는 `VISIBLE`, `INVISIBLE`, `GONE` 각각이 어떤 의미인지, 실제로 언제 어떤 상황에서 사용해야 하는지를 중심으로 정리해보겠습니다.  

---

### 1. visibility란?

`View` 클래스가 기본으로 갖고 있는 속성으로,  
해당 뷰가 **현재 레이아웃 상에서 어떤 상태로 존재할 것인지**를 정의합니다.  

```xml
android:visibility="visible"   // 기본값
android:visibility="invisible"
android:visibility="gone"
```

또는 코드로 설정할 경우:

```kotlin
view.visibility = View.VISIBLE
view.visibility = View.INVISIBLE
view.visibility = View.GONE
```

---

### 2. VISIBLE

* 뷰가 **화면에 보이며**, **공간을 차지**합니다.
* 기본값이며, 보이도록 만들고 싶을 때 사용합니다.

```kotlin
textView.visibility = View.VISIBLE
```

**사용 예시**

* 버튼을 눌렀을 때 텍스트가 나타나야 하는 경우
* 로딩이 완료되었을 때 실제 데이터를 보여주는 View

---

### 3. INVISIBLE

* 뷰가 **화면에는 보이지 않지만**, **레이아웃 상 공간은 그대로 차지**합니다.
* 보이지는 않지만 나중에 다시 보일 것을 고려해 레이아웃 자리를 유지할 필요가 있을 때 사용합니다.

```kotlin
textView.visibility = View.INVISIBLE
```

**사용 예시**

* 사용자에게는 안 보이지만 레이아웃 정렬을 깨뜨리지 않아야 할 때
* 애니메이션 시작 전/후 잠시 숨길 때 (자리 유지 필수)

---

### 4. GONE

* 뷰가 **화면에도 보이지 않고**, **레이아웃 상에서도 존재하지 않는 것처럼 공간을 차지하지 않습니다.**
* 완전히 사라진 것처럼 취급되며, 동적으로 레이아웃을 구성할 때 유용합니다.

```kotlin
textView.visibility = View.GONE
```

**사용 예시**

* 조건에 따라 뷰 자체가 없어져야 하는 상황 (예: 로그인 상태에 따라 버튼 제거)
* 리스트 중 특정 항목이 필요 없을 때 레이아웃 재정렬을 유도할 경우

---

### 5. 언제 어떤 속성을 써야 할까?

| 상황                | 추천 visibility |
| ----------------- | ------------- |
| 뷰를 보여줘야 함         | `VISIBLE`     |
| 뷰를 숨기되 공간은 유지해야 함 | `INVISIBLE`   |
| 뷰를 완전히 제거하고 싶음    | `GONE`        |

**Tip**: 단순히 안 보이는 게 목적이라면 `GONE`을 쓰는 게 일반적이에요.  
하지만 자리를 유지하며 교체되거나 애니메이션될 뷰는 `INVISIBLE`이 더 적합해요.  

---

### 참고자료

* [Android Developers - View#setVisibility](https://developer.android.com/reference/android/view/View#setVisibility%28int%29)
* [Android Layouts - Official Guide](https://developer.android.com/guide/topics/ui/declaring-layout)
