# Compose Stateless vs Stateful

> Jetpack Compose에서는 UI를 설계할 때 **Stateless**와 **Stateful** 컴포저블을 명확하게 나누는 것이 구조적이고 유지보수에 강한 코드를 작성하는 데 중요합니다.  
> 상태(state)를 어디서 소유하느냐에 따라 구성 요소의 재사용성, 테스트 용이성, 확장성이 완전히 달라지기 때문이에요.  

## 1. Compose에서의 Stateless란?

Stateless란 말 그대로 **상태를 갖지 않는 컴포저블**을 의미합니다.  
다시 말해, UI의 입력 값(state)을 외부로부터 받아서 **단순히 그려주는 역할만 하는** 구조입니다..  

```kotlin
@Composable
fun CounterText(count: Int) {
    Text(text = "현재 값은 $count")
}
```

* `count`를 외부에서 받아 그려만 줌
* 재사용성과 테스트 용이성이 뛰어남
* 상태 변경 책임이 없음 → Side-effect 없음

**이유**: UI 상태를 직접 다루지 않으니 부작용이 없고, 다양한 곳에서 재사용 가능해져요.

## 2. Compose에서의 Stateful이란?

Stateful 컴포저블은 내부에 상태를 소유하고, 그 상태에 따라 UI를 변화시키는 역할까지 수행합니다.

```kotlin
@Composable
fun CounterWithButton() {
    var count by remember { mutableStateOf(0) }

    Column {
        Text("현재 값은 $count")
        Button(onClick = { count++ }) {
            Text("증가")
        }
    }
}
```

* `count` 상태를 직접 소유
* 자체적으로 UI 상태 변경 처리
* 외부에서 제어하기 어려움 → 재사용성 낮음

**주의점**: 상태가 많아지거나 복잡해지면 테스트나 코드 흐름 추적이 어려워져요.

## 3. 왜 Stateless & Stateful을 분리해야 할까?

> Jetpack Compose는 **단방향 데이터 흐름(One-Way Data Flow)** 를 지향합니다.  
> 상태는 외부에서 관리하고, UI는 그 상태를 받아 표현만 하도록 나누는 것이 이상적일 수도 있습니다.   

### 장점

| 구분   | 설명                                   |
| ---- | ------------------------------------ |
| 유지보수 | 상태 관리가 한곳에 집중되기 때문에 버그 추적이 쉬움        |
| 테스트  | Stateless 컴포저블은 단순 렌더링만 하므로 테스트하기 쉬움 |
| 재사용성 | 다양한 상태와 함께 재활용 가능                    |
| 컴포지션 | 컴포저블 간의 의존성이 줄어들어 구조가 유연해짐           |

### 리컴포지션과의 관계

Stateless한 컴포저블은 외부에서 상태를 주입받기 때문에, **상태가 변경될 때만 리컴포지션**이 발생해요.
이는 Compose의 리컴포지션 성능 최적화와도 밀접하게 연결돼 있어요.

* 불필요한 상태 소유를 줄이면 리컴포지션 범위를 좁힐 수 있음
* Compose는 파라미터가 변경되지 않는 한 재컴포지션을 하지 않음
* 따라서 Stateless하게 작성하면 성능상 이점도 생김

## 4. 실전 예시 - 리팩토링 구조

### 안 좋은 구조

```kotlin
@Composable
fun CounterBox() {
    var count by remember { mutableStateOf(0) }

    Column {
        Text("Count: $count")
        Button(onClick = { count++ }) {
            Text("증가")
        }
    }
}
```

모든 게 하나의 컴포저블에 몰려 있어 테스트도, 재사용도 어려워요.

### 리팩토링된 구조

```kotlin
@Composable
fun CounterScreen() {
    var count by remember { mutableStateOf(0) }
    CounterContent(count = count, onIncrement = { count++ })
}

@Composable
fun CounterContent(count: Int, onIncrement: () -> Unit) {
    Column {
        Text("Count: $count")
        Button(onClick = onIncrement) {
            Text("증가")
        }
    }
}
```

* `CounterScreen`: 상태를 소유 (Stateful)
* `CounterContent`: 상태를 표현 (Stateless)

이렇게 나누면 `CounterContent`는 다른 ViewModel이나 상태와 쉽게 조합할 수 있어요.

## 5. Compose 설계 시 권장 패턴

> 항상 기본은 Stateless로 만들고, 꼭 필요한 경우에만 Stateful하게 확장하세요.

### 공식 권장 방식

* `State hoisting` (상태 끌어올리기)
  → 상태를 외부로 분리하고, 이벤트 콜백만 내부에 둬요.
* ViewModel → UI State → Stateless UI 구조로 설계

## 참고자료

* [State and State Hoisting - Android Developers 공식 문서](https://developer.android.com/jetpack/compose/state)
* [Thinking in Compose - 공식 아키텍처 가이드](https://developer.android.com/jetpack/compose/architecture)
