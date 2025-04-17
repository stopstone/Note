# Compose Alignment, Arrangement

> Jetpack Compose에서 UI를 구성할 때 레이아웃 정렬은 아주 많이 활용되는 개념입니다.
>
> 이 글에서는 다음과 같은 내용을 정리합니다:
> - Modifier.align()과 Arrangement/Alignment의 차이점
> - Row와 Column에서 Alignment와 Arrangement의 적용 방식 비교
> - 각 정렬 방식의 종류 정리
> 

## Modifier.align() vs Arrangement / Alignment

| 구분 | 대상 | 역할 | 사용 위치 |
|------|------|------|------------|
| `Modifier.align()` | 개별 컴포저블 | 개별 자식의 정렬 | `RowScope`, `ColumnScope`, `BoxScope` 등 |
| `Arrangement` | 전체 레이아웃 | 주 축(main axis)의 자식 정렬 | `horizontalArrangement`, `verticalArrangement` |
| `Alignment` | 전체 레이아웃 | 교차 축(cross axis)의 자식 정렬 | `horizontalAlignment`, `verticalAlignment` |

### 예시: Modifier.align()
```kotlin
Row {
    Text("Hello", modifier = Modifier.align(Alignment.CenterVertically))
}
```
→ Text는 Row 내부에서 수직 중앙에 정렬됩니다.  

### 예시: Arrangement & Alignment
```kotlin
Column(
    verticalArrangement = Arrangement.SpaceBetween,
    horizontalAlignment = Alignment.CenterHorizontally
) {
    Text("Item 1")
    Text("Item 2")
}
```
→ Text들은 수직 방향으로 균등하게 배치되고, 수평 방향으로 중앙 정렬됩니다.  

```kotlin
Row(
    horizontalArrangement = Arrangement.SpaceAround,
    verticalAlignment = Alignment.CenterVertically
) {
    Text("Item A")
    Text("Item B")
}
```
→ Text들은 수평 방향으로 동일한 간격을 두고 배치되며, 수직 방향으로는 중앙 정렬됩니다.  
→ Text들은 수직 방향으로 균등하게 배치되고, 수평 방향으로 중앙 정렬됩니다.  

---

## Arrangement의 종류

**Row에서의 Arrangement**  
![image](https://github.com/user-attachments/assets/c80962f3-f07d-4a21-a18c-40eac6688074)  

**Column에서의 Arrangement**  
![image](https://github.com/user-attachments/assets/5c7f5db3-ee06-4438-a320-330281c3c2ed)  

| Arrangement | 설명 |
|-------------|------|
| `Arrangement.Start` | 자식들을 시작 지점에 정렬 |
| `Arrangement.End` | 자식들을 끝 지점에 정렬 |
| `Arrangement.Center` | 자식들을 중앙에 정렬 |
| `Arrangement.SpaceBetween` | 첫 번째 자식은 시작, 마지막 자식은 끝에 배치, 나머지는 사이 간격 동일하게 |
| `Arrangement.SpaceAround` | 자식 사이 간격 동일, 시작과 끝은 절반 크기의 간격을 둠 |
| `Arrangement.SpaceEvenly` | 모든 자식과 시작/끝 사이 간격을 동일하게 둠 |

※ `Row`에선 `horizontalArrangement`, `Column`에선 `verticalArrangement`로 사용됩니다.

---

## Alignment의 종류

| Alignment | 사용 위치 | 설명 |
|-----------|-----------|------|
| `Alignment.Top` | Row의 `verticalAlignment` | 자식들을 위쪽에 정렬 |
| `Alignment.CenterVertically` | Row의 `verticalAlignment` | 수직 중앙에 정렬 |
| `Alignment.Bottom` | Row의 `verticalAlignment` | 아래쪽에 정렬 |
| `Alignment.Start` | Column의 `horizontalAlignment` | 자식들을 왼쪽에 정렬 |
| `Alignment.CenterHorizontally` | Column의 `horizontalAlignment` | 수평 중앙에 정렬 |
| `Alignment.End` | Column의 `horizontalAlignment` | 오른쪽에 정렬 |

---

## 참고 자료
- [Jetpack Compose 공식 문서 - Layouts](https://developer.android.com/develop/ui/compose/layouts/basics)
- [Jetpack Compose 공식 문서 - Alignment lines](https://developer.android.com/develop/ui/compose/layouts/alignment-lines)
- [Medium - Understand Arrangement and Alignment in Jetpack Compose](https://vitor-ramos.medium.com/understand-arrangement-and-alignment-in-jetpack-compose-7633f2ed5b39)

