# LayoutParams의 역할과 measure() / layout()의 차이

> 안드로이드에서 View를 배치할 때 가장 핵심적인 개념 중 하나가 `LayoutParams`, `measure()`, `layout()`입니다.    
> 이 글에서는 LayoutParams가 뭘 하고, measure와 layout이 각각 어떤 역할을 하는지 명확히 구분해서 설명하겠습니다.  

## 1. LayoutParams란?

* View가 **부모에게 요청하는 배치 정보**입니다.
* 대표 속성: `width`, `height` (`match_parent`, `wrap_content`, `0dp` 등)
* View 자신이 아니라 **부모 ViewGroup**이 이를 해석해서 자식 View를 배치합니다.

```kotlin
val params = LinearLayout.LayoutParams(
    ViewGroup.LayoutParams.MATCH_PARENT,
    ViewGroup.LayoutParams.WRAP_CONTENT
)
myView.layoutParams = params
```

* ViewGroup마다 고유한 LayoutParams 하위 클래스를 사용합니다.

  * 예: `LinearLayout.LayoutParams`, `ConstraintLayout.LayoutParams`

## 2. measure() 함수: View의 크기 측정 단계

* `measure()`는 View에게 **너 얼마만큼의 크기로 측정할래?** 라고 묻는 단계입니다.
* 이때 부모는 자식에게 `MeasureSpec`을 통해 제약을 줍니다:

  * `EXACTLY`: 정확히 이 크기로 해라
  * `AT_MOST`: 최대 이 크기까지만 가능하다
  * `UNSPECIFIED`: 얼마든지 원하는 만큼 가져도 된다
* View는 이 값을 바탕으로 실제 측정 크기를 결정하고 `setMeasuredDimension()`을 호출합니다.

```kotlin
override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
    val width = MeasureSpec.getSize(widthMeasureSpec)
    val height = MeasureSpec.getSize(heightMeasureSpec)
    setMeasuredDimension(width, height)
}
```

## 3. layout() 함수: View의 위치 배치 단계

* `layout()`은 측정된 크기를 바탕으로 **View가 어디에 위치할지**를 결정합니다.
* ViewGroup은 자식 View에 대해 layout을 호출하며 위치를 지정합니다.
* 내부적으로는 `onLayout()`을 오버라이드하여 자식들의 좌표를 설정합니다.

```kotlin
override fun onLayout(changed: Boolean, l: Int, t: Int, r: Int, b: Int) {
    for (i in 0 until childCount) {
        val child = getChildAt(i)
        child.layout(left, top, right, bottom)
    }
}
```

## 4. LayoutParams vs measure() vs layout() 비교 정리

| 구분           | 역할             | 호출 주체        | 대상           | 실행 시점        |
| ------------ | -------------- | ------------ | ------------ | ------------ |
| LayoutParams | 부모에게 요청할 배치 정보 | View         | 부모 ViewGroup | 측정 이전        |
| measure()    | View 크기 측정     | 부모 ViewGroup | 자식 View      | onMeasure 시점 |
| layout()     | View 위치 배치     | 부모 ViewGroup | 자식 View      | onLayout 시점  |

## 5. 실제 적용 예: 커스텀 ViewGroup 만들기

* `onMeasure()`에서 자식들을 모두 측정합니다.
* `onLayout()`에서 자식들의 위치를 계산해 layout을 호출합니다.
* 필요한 경우 `generateLayoutParams()`나 `checkLayoutParams()`를 통해 커스텀 속성을 반영합니다.
* 커스텀 ViewGroup을 만들 땐 이 흐름을 정확히 이해하고 있어야 합니다.

## 참고자료

* [Android 공식 문서 - Layout process](https://developer.android.com/guide/topics/ui/how-android-draws)
* [Android Developers - ViewGroup.LayoutParams](https://developer.android.com/reference/android/view/ViewGroup.LayoutParams)
