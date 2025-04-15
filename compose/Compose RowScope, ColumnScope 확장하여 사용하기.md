# Compose RowScope, ColumnScope 확장하여 사용하기

> Compose로 UI를 구성하다 보면 반복적인 레이아웃 구조나 특정한 정렬 패턴을 반복해서 사용하는 경우가 많습니다.  
> 이럴 때 RowScope나 ColumnScope를 확장해 컴포저블을 커스텀하면 UI를 더 간결하고 재사용성 있게 구성할 수 있어요.  
> 이러한 스코프 확장을 중심으로 예시와 함께 정리해보겠습니다.

## 1. 왜 알아야 할까요?

반복되는 UI, 특히 다이얼로그 버튼이나 섹션 헤더처럼 일정한 구조를 가진 컴포넌트를 만들 때,  
매번 Column이나 Row에 정렬과 스타일을 지정하는 건 번거롭고 비효율적일 수 있습니다.  
스코프 확장 함수로 이런 반복을 추상화하면 유지보수성도 좋아지고 코드도 깔끔해집니다.  

- **커스텀 UI 추상화**: ColumnScope나 RowScope를 확장하면 반복되는 UI 구조를 컴포저블 함수로 깔끔하게 추상화 가능
- **UI 일관성 유지**: 같은 스타일과 정렬을 공유하는 컴포넌트를 반복 구성할 때 일관성을 유지
- **유지보수 용이성**: 명확한 역할 분리는 코드의 가독성을 높이고, 유지보수 향상
- **재사용성 향상**: 스코프 기반 확장은 다양한 화면에서 일관되게 재사용 가능한 컴포넌트화 가능

## 2. 스코프 확장 함수 예시
### 2-1. ColumnScope 스코프 확장 함수 예시

```kotlin
@Composable
fun ColumnScope.SampleTitle() {
    Text("제목입니다", modifier = Modifier.align(Alignment.CenterHorizontally))
    Text("본문 내용", modifier = Modifier.align(Alignment.Start))
}

Column {
    SampleTitle()
}
```
내부의 각 `Text` 컴포저블에서 `Modifier.align()`을 자연스럽게 사용할 수 있어요.  
별도의 람다 인자를 받지 않아도 ColumnScope 문맥을 공유할 수 있습니다.

이런 확장은 **설정 화면에서 섹션 제목과 본문을 반복해서 표현하거나**, **피드 목록에서 콘텐츠 블록을 통일된 구조로 배치할 때** 활용할 수 있습니다.

### 2-2. RowScope 스코프 확장 함수 예시

```kotlin
@Composable
fun RowScope.SampleButtons() {
    Button(onClick = { /* 취소 */ }, modifier = Modifier.weight(1f)) {
        Text("취소")
    }
    Spacer(modifier = Modifier.width(8.dp))
    Button(onClick = { /* 확인 */ }, modifier = Modifier.weight(1f)) {
        Text("확인")
    }
}

Row(modifier = Modifier.fillMaxWidth()) {
    SampleButtons()
}
```

이처럼 RowScope를 확장하면 버튼 그룹 같은 컴포넌트를 **한 번에 배치하고 스타일까지 통일**시킬 수 있습니다.  

## 참고 자료

- [Compose Row, Column and Scoped Modifiers](https://www.valueof.io/blog/compose-row-column-rowscope-columnscope-modifier)
- [Understand Arrangement and Alignment in Jetpack Compose](https://vitor-ramos.medium.com/understand-arrangement-and-alignment-in-jetpack-compose-7633f2ed5b39)
- [Composing Alignment & Arrangement](https://zoewave.medium.com/composing-alignment-arrangement-4917d41640e9)

