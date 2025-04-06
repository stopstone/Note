# remember와 rememberSaveable, 알고 쓰자

Jetpack Compose를 다루면서 상태(state)를 관리하는 것은 매우 중요합니다.  
Compose에서 필수적으로 알아야 하는 `remember`와 `rememberSaveable`에 대해 이야기해 보겠습니다.  
둘의 개념을 정리하고, 언제 무엇을 사용해야하는지, 무작정 써도 되는지에 대해 알아보는 것을 목표로 합니다.

## 1. 둘의 개념과 차이 정리

| 항목 | remember | rememberSaveable |
|:---|:---|:---|
| 정의 | 컴포저블 함수 내에서 값을 기억 | 컴포저블 함수 내에서 값을 기억 + 구성 변경(Configuration Change)/프로세스 재생성(Activity recreation) 후 복원 |
| 저장 방식 | 메모리에 저장 | Bundle을 활용해 저장 |
| 복원 가능성 | 구성 변경(Configuration Change) 시 초기화 | 구성 변경(Configuration Change) 후에도 복원 |
| 저장 가능한 데이터 | 기본 타입, MutableState | 기본 타입, Parcelable, Saver 지원 |
| 생명주기 | 컴포저블이 활성화되어 있을 때만 | 구성 변경 이후에도 유지 |
| 유지 범위 | Recomposition 동안 | 구성 변경(Configuration Change), 프로세스 재생성(Activity recreation)까지 (완전한 프로세스 죽음 복구는 불가) |
| 사용 예 | 버튼 클릭 수, 임시 UI 상태 | 사용자 입력값, 스크롤 위치, 탭 상태 |
| 주의사항 | 구성 변경 시 초기화 | 데이터 크기 제한 주의 (약 1MB 초과 시 주의) |

> 참고: "구성 변경"은 화면 회전이나 언어 변경처럼 사용자가 트리거하는 변화이고,
> "프로세스 재생성"은 메모리 부족 등으로 시스템이 프로세스를 강제 종료한 뒤 복구하는 상황입니다.

- **remember**: 컴포저블이 살아있는 동안 상태를 기억합니다.
- **rememberSaveable**: 구성 변경(Configuration Change)이나 시스템에 의한 프로세스 재생성(Activity recreation) 후에도 상태를 유지할 수 있습니다.  
  다만, 앱이 완전히 종료되어 프로세스가 제거된 경우에는 ViewModel과 SavedStateHandle 같은 추가 기법을 사용해야 합니다.

**remember 예시 코드**
```kotlin
@Composable
fun Counter() {
    var count by remember { mutableStateOf(0) }
    Column {
        Text("Count: $count")
        Button(onClick = { count++ }) {
            Text("Increment")
        }
    }
}
```

**rememberSaveable 예시 코드**
```kotlin
@Composable
fun SaveableCounter() {
    var count by rememberSaveable { mutableStateOf(0) }
    Column {
        Text("Count: $count")
        Button(onClick = { count++ }) {
            Text("Increment")
        }
    }
}
```

## 2. 왜 둘 다 필요한가?

| 항목 | remember | rememberSaveable |
|:---|:---|:---|
| 목적 | Recomposition 동안 값 유지 | 구성 변경(Configuration Change), 프로세스 재생성(Activity recreation) 후 값 복원 |
| 필요 이유 | Recomposition 시 초기화를 막기 위해 | 구성 변경 등에도 상태를 유지하기 위해 |

### "rememberSaveable더 확실한 방법 같은데, rememberSaveable만 쓰면 되는 것 아닌가?"

- `rememberSaveable`은 내부적으로 Bundle을 이용해 상태를 저장합니다.
- 하지만 Bundle에는 저장 크기 제한이 있습니다. **Android에서는 일반적으로 하나의 트랜잭션으로 1MB를 초과하면 `TransactionTooLargeException`이 발생할 수 있습니다.**
- 복잡하거나 대용량 데이터를 저장하면 오류가 발생할 수 있습니다.

**복구 가능/불가능 구분**
| 상황 | 복구 여부 |
|:---|:---|
| 구성 변경(Configuration Change) | 복구 가능 |
| 시스템이 프로세스를 종료한 후 Activity를 재생성하는 경우 | 복구 가능 |
| 앱 완전 종료(사용자가 명시적으로 앱을 종료하거나 시스템이 프로세스를 완전히 제거한 경우) | 복구 불가 (ViewModel + SavedStateHandle 필요) |

> 참고: "프로세스 재생성"(Activity recreation)은 메모리 부족 등으로 시스템이 프로세스를 종료했다가 다시 복구하는 경우를 의미합니다. 이 과정에서는 SavedInstanceState를 활용해 rememberSaveable 상태가 복원될 수 있습니다.

따라서 모든 경우에 rememberSaveable을 사용하는 것은 적절하지 않습니다. 데이터의 크기와 특성, 생명주기를 고려하여 신중하게 선택해야 합니다.

## 3. 어떤 상황에 무엇을 쓸까?

| 상황 | remember 사용 | rememberSaveable 사용 |
|:---|:---|:---|
| 버튼 클릭 수 표시 | O |  |
| 임시 UI 상태 유지 | O |  |
| 사용자 입력 텍스트 유지 |  | O |
| 스크롤 위치 유지 |  | O |
| 탭 선택 상태 유지 |  | O |

**간단한 추천 기준**
- UI 반응 상태(버튼 클릭 수 등)처럼 일시적인 데이터는 `remember` 사용
- 사용자 입력, 스크롤 위치처럼 구성 변경 후에도 유지해야 하는 데이터는 `rememberSaveable` 사용
- 복잡한 화면 상태 관리나 앱 완전 종료 후 복구까지 고려해야 한다면 ViewModel과 SavedStateHandle을 사용하는 것을 고려할 수 있습니다.

**TextField 입력 예시 (rememberSaveable)**
```kotlin
@Composable
fun SaveableTextField() {
    var text by rememberSaveable { mutableStateOf("") }
    Column {
        TextField(
            value = text,
            onValueChange = { text = it },
            label = { Text("Input") }
        )
        Text("You typed: $text")
    }
}
```

## 4. 복잡한 객체를 저장할 때 - Saver 사용하기

`rememberSaveable`은 기본 타입만 자동으로 저장할 수 있습니다. 복잡한 객체를 저장하려면 `Saver`를 활용해야 합니다.

**mapSaver를 이용한 예시**
```kotlin
data class Person(val name: String, val age: Int)

val PersonSaver = mapSaver(
    save = { mapOf("name" to it.name, "age" to it.age) },
    restore = { Person(it["name"] as String, it["age"] as Int) }
)

@Composable
fun PersonScreen() {
    val person = rememberSaveable(saver = PersonSaver) {
        Person("홍길동", 30)
    }
    Text("이름: ${person.name}, 나이: ${person.age}")
}
```

**설명**
- `save` 단계에서는 객체를 Map 형태로 단순화하여 저장합니다.
- `restore` 단계에서는 저장된 Map을 다시 원래 객체로 복원합니다.
- 이 과정을 통해 복잡한 객체도 safely 저장할 수 있습니다.

## Tip. by 키워드의 역할
Compose 코드에서 `var count by remember { mutableStateOf(0) }`와 같은 표현을 자주 볼 수 있습니다.

- `by`는 Kotlin의 위임(delegation) 문법을 활용한 것입니다.
- `MutableState` 객체의 `value`를 직접 다루지 않고, 일반 변수처럼 읽고 쓸 수 있게 해줍니다.
- 즉, `count`를 읽으면 내부적으로 `value`를 가져오고, `count`에 값을 대입하면 `value`가 바뀌게 됩니다.

이 덕분에 Compose 코드가 훨씬 간결하고 읽기 쉽게 됩니다.

## 참고 자료

- [Android Developers - Save UI states](https://developer.android.com/develop/ui/compose/state-saving?hl=ko)
- [찰스의 안드로이드 - Jetpack Compose basics – 상태 보존하기](https://charlezz.com/?p=45498)
- [Mobile Innovation Network - Understanding the Difference Between remember and rememberSaveable in Jetpack Compose](http://medium.com/mobile-innovation-network/understanding-the-difference-between-remember-and-remembersaveable-in-jetpack-compose-29d7231053e5)
- [Team QANDA - 요즘 핫🔥-한 Jetpack Compose 로 기능 하나를 통째로 Refactoring 해보기](https://blog.mathpresso.com/jetpack-compose-%EB%A1%9C-%EA%B8%B0%EB%8A%A5%EC%A0%84%EC%B2%B4%EB%A5%BC-refactoring-%ED%95%B4%EB%B3%B4%EC%9E%90-2b921c624e80)



