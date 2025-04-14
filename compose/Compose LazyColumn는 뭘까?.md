# Jetpack Compose LazyColumn
> Compose로 개발을 진행하던 중, 문득 for문과 Column 조합으로 리스트를 만들 수 있지 않나? 라는 고민을 하게 되었습니다.   
> 그렇다면 왜 LazyColumn이 있고, 어떤 방식으로 동작할까?라는 궁금증을 가지고 공부하게 되었습니다.  
> 
> 그래서 LazyColumn과의 차이를 명확히 이해하고, 동작방식을 정리하고자 이 글을 작성합니다.  
> 추가로 View의 RecyclerView와도 비교해보며 어떤 이점을 가지는지 알아보겠습니다.

## 1. Column + for문 vs LazyColumn 비교

| 항목 | Column + for문 | LazyColumn |
|:---|:---|:---|
| 메모리 사용 | 전체 생성 | 필요한 만큼 생성 |
| 성능 | 느림 | 최적화됨 |
| 상태 관리 | 필요 없음 | key 설정 필요 |

`Column + for문`은 작은 데이터 리스트에는 문제가 없어보일 수 있지만,  
많은 데이터를 다루거나 성능 최적화가 필요한 경우 `LazyColumn`을 사용하는 것이 필수입니다.

## 2. LazyColumn은 어떻게 동작할까요?

- **LazyColumn**은 화면에 필요한 아이템만 컴포지션하고, 보이지 않는 아이템은 캐시에 저장해두었다가 필요할 때 재사용합니다.
- 내부적으로 **LayoutNode**를 캐시하여 불필요한 리컴포지션과 측정을 줄이며, 성능을 최적화합니다.
- 스크롤 시 새로운 아이템은 미리 컴포지션(prefetch)하고, 화면 밖으로 사라진 아이템은 즉시 제거하지 않고 캐시합니다.
- **contentType**을 설정하면 동일한 유형의 항목끼리 컴포지션을 재사용할 수 있어 성능이 더욱 향상됩니다.
- Compose의 선언형 UI 철학에 따라, 데이터가 변경되면 필요한 부분만 리컴포지션됩니다.

## 3. 기본 사용법

```kotlin
LazyColumn {
    // 단일 아이템 추가
    item { Text("Header") }

    // 여러 개의 아이템 추가 (개수 기반)
    items(5) { index -> Text("Item: $index") }

    // 단일 아이템 추가
    item { Text("Footer") }
}
```

`item()`은 단일 항목을 추가할 때 사용하고, `items()`는 여러 항목(리스트, 배열, 개수 등)을 추가할 때 사용합니다.

## 4. Key 설정하기

- 리스트 항목이 변동될 경우를 대비해 안정적인 key를 제공해야 합니다.
- key를 지정하지 않으면 불필요한 Recomposition이 발생할 수 있습니다.
- 안정적인 key를 제공하면 **데이터 변경 시 화면 갱신 범위**를 최소화할 수 있어 성능 최적화에 반드시 필요합니다.

```kotlin
LazyColumn {
    items(messages, key = { it.id }) { message ->
        MessageRow(message)
    }
}
```

## 6. RecyclerView vs LazyColumn 비교 요약

그렇다면 View 시스템의 대표적인 리스트 컴포넌트인 **RecyclerView**와 Compose의 **LazyColumn**은 어떤 차이가 있을까요?

- **RecyclerView**는 View 시스템 기반으로, ViewHolder 패턴을 이용해 뷰를 재사용합니다. 이미 생성된 View를 풀(pool)에 저장해두었다가 스크롤할 때 재활용하는 구조입니다.
- 반면, **LazyColumn**은 Compose의 리컴포지션 메커니즘을 활용합니다. LayoutNode를 캐시하여, 필요에 따라 컴포지션을 최소화하고 재사용합니다.
- 데이터 바인딩 관점에서는 RecyclerView는 Adapter를 통해 명시적으로 바인딩하는 반면, LazyColumn은 선언형 방식으로 데이터가 변경되면 자동으로 화면을 갱신합니다.
- 성능 최적화 방식에서도 차이가 있습니다. RecyclerView는 View 재사용 및 DiffUtil 등을 통해 최적화를 하고, LazyColumn은 리컴포지션 최적화와 캐시를 통해 부드러운 스크롤을 지원합니다.

| 항목 | RecyclerView | LazyColumn |
|:---|:---|:---|
| 기반 기술 | View 시스템 | Compose (리컴포지션 기반) |
| 뷰 관리 | ViewHolder 패턴 | LayoutNode 캐시 재사용 |
| 데이터 바인딩 | 수동 Adapter 바인딩 | 선언형 자동 바인딩 |
| 성능 최적화 | View 재사용, DiffUtil | 리컴포지션 최적화, 캐시 재사용 |
| 스크롤 제어 | addOnScrollListener | LazyListState + 코루틴 |

## 참고자료

- [Android Developers 공식 문서 - Lists and Grids in Compose](https://developer.android.com/develop/ui/compose/lists)
- [Pluu Blog - Lazy layouts in Compose 요약](https://pluu.github.io/blog/android/androidx/2025/01/10/LazyList-1/)
- [Suhyeon Kim Blog - LazyColumn 작동 방식 이해하기](https://medium.com/@wisemuji/lazycolumn-%EC%9E%91%EB%8F%99-%EB%B0%A9%EC%8B%9D-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0-0a5433f31306)

