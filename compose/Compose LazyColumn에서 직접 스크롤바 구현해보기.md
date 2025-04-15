# Jetpack Compose에서 스크롤바 직접 만들어쓰기

> 컴포즈 스크롤을 구현하면서 문득, 스크롤바는 어떻게 추가하지? 라는 고민을 하게 되었습니다.
> 찾아보니 공식문서나 Material3에서 지원하고 있는 사항이 아닌 것 같아 직접 한 번 구현해보게 되었습니다.
> 이번 글에서는 **코드 구조나 디자인보다는 동작하는 커스텀 스크롤바 구현**을 목표로 한 첫 시도 과정을 기록하려고 합니다.
>
> 전체 코드는 해당 저장소에서 확인하실 수 있습니다. [커스텀 스크롤바](https://github.com/stopstone/MyCustomScrollbar)

---

## 1. 기본 레이아웃 구성

```kotlin
Box(modifier = Modifier.fillMaxSize()) {
    LazyColumn(...) { ... } // 보여줄 리스트
    CustomScrollbarForLazyList(...) { ... } // 커스텀 스크롤바
}
```

리스트(`LazyColumn`)와 스크롤바를 하나의 `Box` 안에 쌓아두는 구조로 구성했습니다.  
`CustomScrollbarForLazyList`라는 컴포저블을 따로 만들어,  
스크롤바 위치를 `Box`의 `align(Alignment.CenterEnd)`로 오른쪽에 배치했습니다.

---

## 2. 리스트 구성 (`LazyColumn`)

```kotlin
LazyColumn(
    modifier = Modifier.fillMaxSize(),
    state = listState,
) {
    itemsIndexed(items) { _, item -> 
        Text(...) 
        VerticalDivider(...) 
    }
}
```

- 30개의 간단한 텍스트 아이템으로 리스트를 구성했습니다.
- `itemsIndexed`를 사용해 각 아이템을 순서대로 렌더링하고, `Text` 아래엔 `VerticalDivider`로 줄을 하나씩 그려주었습니다.
- 이때 스크롤 상태를 추적하기 위해 `LazyListState`를 사용했습니다.

> LazyListState는 Jetpack Compose의 LazyColumn, LazyRow와 같은 Lazy 계열 컴포넌트에서  
> 스크롤 위치, 첫 번째 보이는 아이템의 인덱스, 오프셋 등을 추적하기 위해 사용하는 상태 객체입니다.  
---

## 3. 스크롤바 구현 시작 (`CustomScrollbarForLazyList`)

```kotlin
Box(modifier = modifier.fillMaxHeight().onGloballyPositioned { ... }) { ... }
```

- 스크롤바를 담는 박스를 따로 만들고, 뷰포트의 높이(`viewportHeightPx`)를 추적하기 위해 `onGloballyPositioned`를 사용했습니다.  
  
> onGloballyPositioned는 컴포저블이 레이아웃에 배치된 후의 위치나 크기 정보를 얻을 수 있게 해주는 콜백입니다.

```kotlin
val layoutInfo = listState.layoutInfo
val firstVisible = layoutInfo.visibleItemsInfo.firstOrNull()
val totalItems = layoutInfo.totalItemsCount
```

- `LazyListState`에서 `layoutInfo`를 꺼내 현재 보이는 아이템들과 전체 아이템 수를 파악했습니다.

---

## 4. 스크롤 위치 계산 로직

```kotlin
if (estimatedItemHeightPx == null) {
    estimatedItemHeightPx = layoutInfo.visibleItemsInfo.map { it.size }.average().toFloat()
}
```

- 전체 높이를 계산하려면 아이템 높이가 필요합니다.  
  그래서 보이는 아이템들의 평균 높이를 한 번 계산해서 추정값으로 사용했습니다.

```kotlin
val totalContentHeight = itemHeight * totalItems
```

- 전체 콘텐츠 높이는 "아이템 높이 × 아이템 수"로 계산했습니다.

```kotlin
val thumbHeightPx = (viewportHeightPx * viewportHeightPx) / totalContentHeight
```

- 스크롤바 높이는 보이는 영역과 전체 콘텐츠의 비율을 기반으로 설정했습니다.
- 너무 작으면 조작이 힘들기 때문에 `30dp` 이상으로 제한했습니다.

---

## 5. 스크롤바 위치 계산

```kotlin
val scrollOffsetY = firstVisible.index * itemHeight - firstVisible.offset
val maxOffset = (totalContentHeight - viewportHeightPx).coerceAtLeast(1f)
val thumbOffsetPx = (scrollOffsetY / maxOffset) * (viewportHeightPx - thumbHeightPxClamped)
```

- 현재 리스트의 스크롤 위치에 따라 스크롤바의 Y 오프셋을 계산했습니다.
- 전체 높이 대비 현재 위치를 비율로 환산하여 thumb의 Y위치를 결정했습니다.

---

## 6. 밀도 변환 및 Thumb 그리기

```kotlin
with(LocalDensity.current) {
    val thumbOffsetDp = thumbOffsetPx.toDp()
    val thumbHeightDp = thumbHeightPxClamped.toDp()
    thumbContent(thumbOffsetDp, thumbHeightDp)
}
```

- 픽셀 단위로 계산한 값을 Compose에서 쓸 수 있는 `Dp` 단위로 변환했습니다.
- `thumbContent()`를 호출해 실제 스크롤바 모양을 그리게 했습니다.

---

## 7. 스크롤바 모양 (`ScrollbarThumb`)

```kotlin
Box(
    modifier = Modifier
        .offset(y = offsetY)
        .height(height)
        .width(width)
        .background(color)
        .clip(shape)
)
```

- 이 부분에서 스크롤바의 모양을 직접 다룰 수 있게 해두었습니다.
- offset, height, width는 호출부로부터 받아와 위치를 정의하는 것이기 때문에 건들면 안됩니다.
- background나 shape을 통해 모양을 변경할 수 있습니다.

---

## 8. 라이브러리 활용

사실 직접 구현한 방식이다 보니, 내부적으로 문제도 많고 최적화가 완벽하지 않을 수 있습니다.  
그렇기 때문에 많은 분들이 이미 잘 만들어둔 서드파티 라이브러리를 활용하시는 것이 훨씬 편할 수도 있다고 생각합니다.

그중 하나는 [LazyColumnScrollbar](https://github.com/nanihadesuka/LazyColumnScrollbar) 라이브러리입니다. 

---

## 마무리

이번 구현은 완성형은 아닙니다.  
터치로 드래그도 안 되고, 정확한 높이 추정도 상황에 따라 달라질 수 있습니다.  
하지만 공식 컴포넌트가 없는 상황에서 **스크롤 상태와 스크롤바 위치를 수식으로 직접 매핑**해보는 연습을 해보았고,  
어떤 구조로 구성해야 할지 방향을 잡는 데 의미가 있었다고 생각합니다.

다음 단계에서는  
- **드래그 가능한 Thumb**
- **애니메이션 효과**

같은 고도화를 시도해볼 수 있을 것 같습니다.
