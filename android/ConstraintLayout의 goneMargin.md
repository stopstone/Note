# ConstraintLayout의 goneMargin

> ConstraintLayout을 사용하다 보면 `margin`과 비슷한 `goneMargin`이라는 속성이 있습니다.  
> 그런데 이 속성은 단순히 마진처럼 작동하지 않으며, 뷰가 `gone` 상태일 때만 적용되는 특수한 마진입니다.  
> 이 글에서는 `goneMargin`이 무엇인지, 어떤 상황에서 유용하게 사용되는지를 정리해보겠습니다.  

## 1. goneMargin이란?

* `goneMargin`은 **뷰가 `visibility="gone"`일 때만 적용되는 마진**입니다.
* 일반적인 `layout_marginStart`, `layout_marginTop` 등의 속성은 뷰가 보여질 때 적용되지만,  
  `goneMargin`은 뷰가 **완전히 사라졌을 때 기준선이 되는 마진**을 따로 설정할 수 있도록 도와줍니다.

## 2. 왜 필요한가요?

* ConstraintLayout은 뷰들 간의 관계를 기반으로 배치됩니다.
* 중간에 위치한 뷰가 `gone` 상태가 되면, 그 뷰를 기준으로 배치되던 다른 뷰의 위치가 예상치 못하게 변경될 수 있습니다.
* 이때 `goneMargin`을 사용하면 **사라진 뷰가 존재했던 자리에 간격을 유지할 수 있어** 전체 레이아웃의 균형을 맞출 수 있습니다.

## 3. 예시 코드로 살펴보기

```xml
<androidx.constraintlayout.widget.ConstraintLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <TextView
        android:id="@+id/title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="제목"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_marginStart="16dp" />

    <TextView
        android:id="@+id/subtitle"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="서브제목"
        android:visibility="gone"
        app:layout_constraintStart_toEndOf="@+id/title"
        app:layout_constraintTop_toTopOf="@+id/title"
        app:layout_marginStart="8dp"
        app:layout_goneMarginStart="32dp" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

* 위 예시에서 `subtitle`이 gone 상태일 경우에도 `title`의 오른쪽에는 32dp만큼 여유가 생깁니다.
* `goneMarginStart` 덕분에 `subtitle`이 사라져도 레이아웃 흐름이 깨지지 않고 정렬이 유지됩니다.

## 4. margin과 goneMargin 비교 정리

| 속성 이름            | 적용 시점             | 설명                  |
| ---------------- | ----------------- | ------------------- |
| `layout_margin*` | 뷰가 **visible**할 때 | 일반적인 마진, 항상 적용됨     |
| `goneMargin*`    | 뷰가 **gone**일 때    | 사라진 뷰의 위치 기준 마진 유지용 |

## 5. 언제 사용하면 좋을까?

* **조건부 UI 요소가 있는 경우**

  * 예: 사용자 상태 메시지가 있을 때만 보여주는 뷰
* **간격이 중요한 레이아웃 구성일 경우**

  * 예: 버튼들이 일렬로 정렬되어야 할 때, 특정 버튼이 `gone` 상태여도 균형 유지
* **ViewStub을 사용할 때와 유사한 상황**

  * 나중에 동적으로 붙을 뷰의 자리를 미리 고려한 마진 설정

## 참고자료

* [Android Developers - ConstraintLayout](https://developer.android.com/reference/androidx/constraintlayout/widget/ConstraintLayout)
