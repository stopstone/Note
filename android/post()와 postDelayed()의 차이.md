# post()와 postDelayed()의 차이

> 안드로이드에서 UI 작업을 예약할 때 `post()`나 `postDelayed()`를 사용하는 경우가 많습니다.  
> 특히 `View`나 `Handler`를 통해 메시지 큐에 작업을 등록할 수 있는데요, 두 메서드의 차이를 명확히 이해하고 있으면 UI 타이밍 버그를 예방하고 더 안정적인 코드를 작성할 수 있습니다.  
> 이 글에서는 두 메서드의 동작 방식과 차이, 그리고 각각을 언제 쓰면 좋은지 정리해보겠습니다.  

## 1. post()란?

`post()`는 View가 attach된 후 메시지 큐에 Runnable을 등록해, 해당 View가 그려진 다음 실행되도록 예약하는 메서드입니다.

```kotlin
view.post {
    // View가 layout된 이후 실행됨
}
```

### 특징

* View가 아직 attach되지 않았다면 attach된 이후 실행됨
* UI 스레드에서 실행됨 (Main Thread)
* View의 크기나 위치 정보가 필요할 때 자주 사용됨 (예: `view.width`, `view.height`)

## 2. postDelayed()란?

`postDelayed()`는 지정한 시간만큼 딜레이를 준 후, 메시지 큐에 Runnable을 등록하여 실행되도록 합니다.

```kotlin
view.postDelayed({
    // 지정된 시간 이후 실행됨
}, 1000) // 1초 후
```

### 특징

* 지정한 시간만큼 딜레이된 후 실행됨
* 메시지 큐 기준으로 지연되며, 정확한 시점은 시스템 상황에 따라 달라질 수 있음
* UI 애니메이션 등 시간 제어가 필요한 곳에서 자주 사용됨

## 3. 공통점과 차이점

| 항목         | post()                | postDelayed()       |
| ---------- | --------------------- | ------------------- |
| 실행 시점      | 가능한 빨리                | 지정된 시간 이후           |
| 메시지 큐 등록   | O                     | O                   |
| View 크기 접근 | 보장됨 (layout 이후 실행)    | 보장되지 않음 (타이밍 주의 필요) |
| 주요 사용 목적   | View 크기/위치 접근, 초기화 작업 | 애니메이션, 지연 동작 처리     |

## 4. 언제 어떤 걸 써야 할까?

### post()를 사용하는 경우

* `View`의 크기나 위치가 계산된 이후에 무언가 작업을 해야 할 때
* 예: `view.width`나 `view.height`를 읽어서 애니메이션 시작점 계산 등

```kotlin
view.post {
    startAnimation(view.width)
}
```

### postDelayed()를 사용하는 경우

* 일정 시간 이후 동작을 실행하고 싶을 때
* 예: Splash 화면 종료, 일정 시간 후 UI 전환 등

```kotlin
view.postDelayed({
    navigateToNext()
}, 2000)
```

## 참고자료

* [Handler.post() 공식 문서](https://developer.android.com/reference/android/os/Handler#post%28java.lang.Runnable%29)
* [View.post() 공식 문서](https://developer.android.com/reference/android/view/View#post%28java.lang.Runnable%29)
* [post vs postDelayed StackOverflow 토론](https://stackoverflow.com/questions/3130654/difference-between-post-and-postdelayed)
