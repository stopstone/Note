# View와 ViewGroup의 차이

> 안드로이드에서 화면을 구성할 때 `View`와 `ViewGroup` 라는 개념이 있습니다.  
> 두 용어는 XML 레이아웃에서 자주 등장하며 비슷해 보이지만, 실제로는 구조와 역할에서 차이를 가집니다.  
> 이번 글에서는 View와 ViewGroup이 무엇인지, 어떤 기준으로 구분되는지 정리해보겠습니다.  

---

### 1. View란?

`View`는 안드로이드 화면에 실제로 표시되는 **단일 UI 요소**를 의미합니다.  
텍스트, 이미지, 버튼처럼 사용자와의 상호작용이 가능한 요소들을 View라고 부릅니다.  

#### 대표적인 View 클래스

* `TextView` : 텍스트 출력
* `ImageView` : 이미지 표시
* `Button` : 클릭 가능한 버튼
* `EditText` : 텍스트 입력창
* `CheckBox`, `Switch`, `ProgressBar` 등

> View는 자체적으로 그려지고 동작하는 하나의 컴포넌트입니다.

---

### 2. ViewGroup이란?

`ViewGroup`은 여러 개의 `View` 또는 다른 `ViewGroup`을 자식으로 포함할 수 있는 **컨테이너 역할**을 합니다.  
자식 뷰들을 어떻게 배치할지를 결정하는 것이 핵심 기능입니다.  

#### 대표적인 ViewGroup 클래스

* `LinearLayout` : 가로나 세로 방향으로 정렬
* `RelativeLayout` : 다른 뷰 기준 상대 위치 지정
* `ConstraintLayout` : 제약 조건 기반 정렬
* `FrameLayout`, `CoordinatorLayout`, `ScrollView`, `RecyclerView` 등

> ViewGroup은 뷰 계층 구조를 만드는 뼈대가 됩니다.

---

### 3. View와 ViewGroup의 구분 기준

| 기준         | View                                | ViewGroup                                |
| ---------- | ----------------------------------- | ---------------------------------------- |
| 역할         | UI 요소 하나                            | 여러 View를 포함하는 컨테이너                       |
| 상속         | `android.view.View`                 | `android.view.ViewGroup` (View 상속)       |
| 자식 뷰 포함 여부 | 불가능                                 | 가능                                       |
| 예시         | `TextView`, `Button`, `ImageView` 등 | `LinearLayout`, `ConstraintLayout` 등     |
| XML 구조     | `<TextView />` 단독 사용                | `<LinearLayout>...</LinearLayout>` 자식 포함 |

---

### 참고자료

* [Android Developers - View](https://developer.android.com/reference/android/view/View)
* [Android Developers - ViewGroup](https://developer.android.com/reference/android/view/ViewGroup)
* [StackOverflow - Difference between View and ViewGroup](https://stackoverflow.com/questions/9445669/what-is-the-difference-between-view-and-viewgroup)
