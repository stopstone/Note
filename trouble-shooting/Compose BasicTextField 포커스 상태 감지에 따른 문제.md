# Compose BasicTextField 포커스 상태 감지에 따른 문제 해결

> Compose에서 커스텀 텍스트필드 컴포넌트를 사용할 때 발생하는 포커스 상태 감지 문제에 대해 알아보겠습니다.  
> Material3의 OutlinedTextField와 BasicTextField의 차이점에 대해 공부하였습니다.  

---

## 1. 문제 상황

```kotlin
// 문제가 발생한 코드
NearTextField(
    value = uiState.otherReasonText,
    onValueChange = onUpdateOtherReasonText,
    placeHolderText = "탈퇴 사유를 자세히 알려주세요",
    singleLine = false
)
```

**예상 동작**: 포커스 시 파란색 테두리, 포커스 해제 시 회색 테두리  
**실제 동작**: 포커스 상태와 관계없이 항상 회색 테두리 유지

---

## 2. 문제 원인 분석

### 2.1 NearTextField 컴포넌트 구조

```kotlin
// NearTextField.kt
@Composable
fun NearTextField(
    // ... 파라미터들
) {
    val colors = NearTextFieldColors()
    
    BasicTextField(  // 문제의 원인
        value = value,
        // ... 기타 설정들
        decorationBox = { innerTextField ->
            NearOutlinedTextFieldDecorationBox(
                // ... 설정들
            )
        }
    )
}
```

### 2.2 근본 원인

**BasicTextField 사용의 한계:**

1. **수동 상태 관리 필요**: `BasicTextField`는 `InteractionSource`를 통해 포커스 상태를 감지할 수 있지만, 자동으로 색상을 변경하지는 않음
2. **색상 시스템 불일치**: `BasicTextField`는 `TextFieldDefaults.colors()`를 사용해야 하는데, `OutlinedTextFieldDefaults.colors()`를 사용하고 있음
3. **Material3 호환성**: Material3의 디자인 시스템과 완전히 통합되지 않음

### 2.3 색상 설정 확인

```kotlin
// NearTextFieldColors.kt
@Composable
fun NearTextFieldColors() =
    OutlinedTextFieldDefaults.colors(
        focusedBorderColor = NearTheme.colors.BLUE02_8ACCFF,
        unfocusedBorderColor = NearTheme.colors.GRAY03_EBEBEB,
        // ... 기타 색상 설정들
    )
```

**분석 결과**: `BasicTextField`에서 `OutlinedTextFieldDefaults.colors()`를 사용하고 있어 색상이 제대로 적용되지 않음.   
`TextFieldDefaults.colors()`를 사용해야 함

---

## 3. 해결 방법

### 3.1 즉시 해결 방법: OutlinedTextField 직접 사용

```kotlin
// Before: NearTextField 사용 (문제 있음)
NearTextField(
    value = uiState.otherReasonText,
    onValueChange = onUpdateOtherReasonText,
    placeHolderText = "탈퇴 사유를 자세히 알려주세요",
    singleLine = false
)

// After: OutlinedTextField 직접 사용 (해결됨)
OutlinedTextField(
    value = uiState.otherReasonText,
    onValueChange = onUpdateOtherReasonText,
    placeholder = {
        Text(
            text = "탈퇴 사유를 자세히 알려주세요",
            style = NearTheme.typography.B2_14_MEDIUM,
            color = NearTheme.colors.GRAY02_B7B7B7,
        )
    },
    modifier = Modifier.fillMaxWidth(),
    maxLines = 4,
    colors = OutlinedTextFieldDefaults.colors(
        focusedBorderColor = NearTheme.colors.BLUE02_8ACCFF,
        focusedTextColor = NearTheme.colors.BLACK_1A1A1A,
        focusedContainerColor = NearTheme.colors.WHITE_FFFFFF,
        unfocusedTextColor = NearTheme.colors.BLACK_1A1A1A,
        unfocusedBorderColor = NearTheme.colors.GRAY03_EBEBEB,
        unfocusedContainerColor = NearTheme.colors.WHITE_FFFFFF,
        cursorColor = NearTheme.colors.BLACK_1A1A1A,
    ),
    textStyle = NearTheme.typography.B2_14_MEDIUM,
)
```

### 3.2 필요한 Import 추가

```kotlin
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.OutlinedTextFieldDefaults
```

---

## 4. 해결 결과

### 4.1 기능 개선사항

- **포커스 시 파란색 테두리**: `focusedBorderColor = NearTheme.colors.BLUE02_8ACCFF`
- **포커스 해제 시 회색 테두리**: `unfocusedBorderColor = NearTheme.colors.GRAY03_EBEBEB`
- **멀티라인 지원**: `maxLines = 4`로 설정하여 긴 텍스트 입력 가능
- **일관된 디자인**: NearTheme의 색상과 타이포그래피 유지

### 4.2 사용자 경험 개선

- **시각적 피드백**: 포커스 상태를 명확하게 표시
- **직관적 인터랙션**: 사용자가 텍스트필드 상태를 쉽게 인식
- **일관된 UI**: 앱 전체의 디자인 시스템과 일치

---

## 5. 대안 해결 방법

### 5.1 NearTextField 컴포넌트 개선 (장기적 해결책)

```kotlin
// NearTextField.kt 개선 방안 - OutlinedTextField 기반으로 재구현
@Composable
fun NearTextField(
    value: String,
    onValueChange: (String) -> Unit,
    placeHolderText: String,
    singleLine: Boolean = true,
    modifier: Modifier = Modifier,
    interactionSource: InteractionSource = remember { MutableInteractionSource() }
) {
    val colors = NearTextFieldColors()
    
    OutlinedTextField(
        value = value,
        onValueChange = onValueChange,
        modifier = modifier,
        interactionSource = interactionSource,
        colors = colors,
        textStyle = NearTheme.typography.B2_14_MEDIUM,
    )
}

@Composable
fun NearTextFieldColors() = OutlinedTextFieldDefaults.colors(
    focusedBorderColor = NearTheme.colors.BLUE02_8ACCFF,
    unfocusedBorderColor = NearTheme.colors.GRAY03_EBEBEB,
    focusedTextColor = NearTheme.colors.BLACK_1A1A1A,
    unfocusedTextColor = NearTheme.colors.BLACK_1A1A1A,
    focusedContainerColor = NearTheme.colors.WHITE_FFFFFF,
    unfocusedContainerColor = NearTheme.colors.WHITE_FFFFFF,
    cursorColor = NearTheme.colors.BLACK_1A1A1A,
)
```

### 5.2 포커스 상태 수동 관리

```kotlin
@Composable
fun WithdrawScreen() {
    val interactionSource = remember { MutableInteractionSource() }
    val isFocused by interactionSource.collectIsFocusedAsState()
    
    OutlinedTextField(
        value = uiState.otherReasonText,
        onValueChange = onUpdateOtherReasonText,
        interactionSource = interactionSource,
        colors = OutlinedTextFieldDefaults.colors(
            focusedBorderColor = NearTheme.colors.BLUE02_8ACCFF,
            unfocusedBorderColor = NearTheme.colors.GRAY03_EBEBEB,
            focusedTextColor = NearTheme.colors.BLACK_1A1A1A,
            unfocusedTextColor = NearTheme.colors.BLACK_1A1A1A,
            focusedContainerColor = NearTheme.colors.WHITE_FFFFFF,
            unfocusedContainerColor = NearTheme.colors.WHITE_FFFFFF,
            cursorColor = NearTheme.colors.BLACK_1A1A1A,
        ),
        placeholder = {
            Text(
                text = "탈퇴 사유를 자세히 알려주세요",
                style = NearTheme.typography.B2_14_MEDIUM,
                color = NearTheme.colors.GRAY02_B7B7B7,
            )
        },
        modifier = Modifier.fillMaxWidth(),
        textStyle = NearTheme.typography.B2_14_MEDIUM,
    )
}
```
