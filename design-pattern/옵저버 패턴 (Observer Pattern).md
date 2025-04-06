# 옵저버 패턴 (Observer Pattern)

> 옵저버 패턴은 객체 간의 일대다(1:N) 관계를 정의하여, 어떤 객체의 상태가 변경되었을 때
> 그 객체에 의존하는 모든 객체들에게 자동으로 알림이 전달되고, 필요한 동작을 수행하게 만드는 디자인 패턴입니다.
> 다양한 플랫폼에서 광범위하게 사용되며, 특히 비동기 이벤트 처리에 적합합니다.

---

## 1. 옵저버 패턴, 무엇일까요?

커피숍에서 바리스타가 "아메리카노 나왔습니다!"라고 외칠 때, 자신의 이름이 불렸을 때만 반응하는 상황을 생각해볼 수 있습니다.
이처럼 특정 이벤트가 발생했을 때 관련된 객체만 반응하는 구조가 옵저버 패턴입니다.

안드로이드에서는 키보드 입력, 버튼 클릭, API 응답 수신 등 다양한 이벤트를 감지할 때 옵저버 패턴이 사용됩니다.

---

## 2. 옵저버 패턴의 구현 원리

- 이벤트를 발생하는 클래스(Subject)와 이를 감시하는 클래스(Observer)를 직접 연결하지 않고, 인터페이스를 통해 느슨하게 연결합니다.
- 이벤트가 발생하면 인터페이스 메서드를 호출하여 수신자에게 알립니다.
- 발행자와 구독자가 서로의 구체적인 존재를 알지 못한 채, Observer 인터페이스를 통해 협력합니다. (Subject는 Observer 인스턴스를 리스트로 관리하지만, 내부 구현은 알지 못합니다.)

---

## 3. 옵저버 패턴의 장단점

### 장점
- 캡슐화: 발행자와 구독자가 서로를 몰라도 협력 가능
- 다형성: 다양한 Observer 추가 가능
- 의존성 역전: 구체 클래스를 몰라도 동작 가능
- 재사용성: 레고처럼 조립 가능
- 확장성: 새로운 Observer 추가 시 기존 코드 수정 없음

### 단점
- 구독자 수가 많으면 성능 저하 가능성
- 알림 순서 보장 어려움
- 메모리 누수 가능성 (구독 해제 누락 시 발생)
- 디버깅이 어려워질 수 있음
- 복잡한 연결로 시스템 이해가 어려워질 수 있음

---

## 4. 옵저버 패턴의 동작 원리

### Subject

Subject(발행자)는 관찰 대상이 되는 데이터나 객체를 의미합니다.
주요 책임은 Observer 목록을 관리하고, 상태 변경 시 모든 Observer에게 알림을 주는 것입니다.

```kotlin
interface Subject {
    fun registerObserver(observer: Observer)
    fun unregisterObserver(observer: Observer)
    fun notifyObservers(videoTitle: String)
}

class YouTubeChannel(private val channelName: String): Subject {
    private val subscribers = mutableListOf<Observer>()

    override fun registerObserver(observer: Observer) {
        subscribers.add(observer)
        println("${observer.name}님이 $channelName 채널을 구독했습니다.")
    }

    override fun unregisterObserver(observer: Observer) {
        subscribers.remove(observer)
        println("${observer.name}님이 $channelName 채널 구독을 취소했습니다.")
    }

    override fun notifyObservers(videoTitle: String) {
        subscribers.forEach { it.update(channelName, videoTitle) }
    }

    fun uploadVideo(videoTitle: String) {
        println("$channelName 채널에 새 영상이 업로드되었습니다: $videoTitle")
        notifyObservers(videoTitle)
    }
}
```

### Observer

Observer(구독자)는 Subject의 변경 사항에 반응하는 객체입니다.
주요 책임은 Subject로부터 업데이트를 받아 처리하는 것입니다.

```kotlin
interface Observer {
    val name: String
    fun update(channelName: String, videoTitle: String)
}

class Subscriber(override val name: String) : Observer {
    override fun update(channelName: String, videoTitle: String) {
        println("$name님, $channelName 채널에 새 영상이 업로드되었습니다: $videoTitle")
    }
}
```

### 동작 과정

```kotlin
fun main() {
    val techChannel = YouTubeChannel("테크 리뷰 채널")
    val cookingChannel = YouTubeChannel("요리 레시피 채널")

    val alice = Subscriber("Alice")
    val bob = Subscriber("Bob")
    val charlie = Subscriber("Charlie")

    techChannel.registerObserver(alice)
    techChannel.registerObserver(bob)
    cookingChannel.registerObserver(bob)
    cookingChannel.registerObserver(charlie)

    techChannel.uploadVideo("최신 스마트폰 리뷰")
    cookingChannel.uploadVideo("간단한 파스타 레시피")

    techChannel.unregisterObserver(alice)
    techChannel.uploadVideo("새로운 노트북 비교")
}
```

옵저버 패턴의 핵심은 Subject가 Observer를 구체적으로 알지는 않지만, Observer 인터페이스를 통해 update() 메서드를 호출하는 것입니다.

### Coroutine + Flow 기반 확장 예시

```kotlin
class FlowBasedChannel(val name: String) {
    private val _videos = MutableSharedFlow<String>()
    val videos = _videos.asSharedFlow()

    suspend fun uploadVideo(videoTitle: String) {
        println("$name 채널에 새 영상이 업로드되었습니다: $videoTitle")
        _videos.emit(videoTitle)
    }
}

fun main() = runBlocking {
    val techChannel = FlowBasedChannel("테크 리뷰 채널")

    launch {
        techChannel.videos.collect { videoTitle ->
            println("Subscriber1: 새 동영상 알림을 받았습니다 - $videoTitle")
        }
    }

    launch {
        techChannel.videos.collect { videoTitle ->
            println("Subscriber2: 새 동영상 리뷰를 작성합니다 - $videoTitle")
        }
    }

    techChannel.uploadVideo("최신 스마트폰 리뷰")
    delay(100)
}
```

> Flow는 cold stream이고, SharedFlow는 hot stream입니다.  
> SharedFlow는 기본적으로 `replay`가 0일 경우 발행한 값을 새 구독자가 받을 수 없습니다.  
> `replay` 값을 1 이상으로 설정하면 과거 이벤트를 버퍼에 저장하여 새 구독자에게 전달할 수 있습니다.

---

## 5. LiveData와 옵저버 패턴

LiveData는 MVVM 아키텍처에서 ViewModel과 View를 연결할 때 사용되며, 내부적으로 옵저버 패턴을 기반으로 동작합니다.

**특징**
- 상태 변화 감지 및 자동 반영
- 1:N 다수 관찰자 지원
- 생명주기 안전성 보장 (구독 해제 자동 관리)
- ViewModel이 View를 몰라도 동작 가능
- 활성 상태(LifecycleOwner가 STARTED 이상)일 때만 Observer에게 알림 전송

---

## 6. 안드로이드에서 볼 수 있는 옵저버 패턴 예시

- `OnClickListener`: 버튼 클릭 감지
- `TextWatcher`: 텍스트 입력 변화 감지
- `LiveData.observe()`: 데이터 변화 감지 후 UI 업데이트

---

## 7. 예상 면접 질문

- 옵저버 패턴이 해결하고자 하는 핵심 문제는 무엇인가요?  
  → 객체 간의 강한 결합 없이 상태 변화를 다수 객체에게 효과적으로 전파하는 문제를 해결합니다.

- 옵저버 패턴을 안드로이드 앱 구조에서 적용할 때 주의해야 할 점은 무엇인가요?  
  → 생명주기에 따라 구독 해제를 철저히 관리해야 하며, 메모리 누수 방지에 신경 써야 합니다.

- 옵저버 패턴을 사용할 때 발생할 수 있는 메모리 누수 문제를 설명하고, 이를 어떻게 해결할 수 있나요?  
  → Observer가 Subject에 등록된 채 해제되지 않으면 GC가 회수하지 못합니다. 생명주기 기반 관리(LifecycleOwner)나 명시적 해제(unregister)를 사용합니다.

- LiveData와 StateFlow의 차이점을 옵저버 패턴 관점에서 설명해보세요.  
  → LiveData는 생명주기를 인식하여 자동으로 Observer를 관리하고, StateFlow는 생명주기 개념 없이 항상 최신 상태를 유지하며 수동으로 collect해야 합니다. LiveData는 UI(View)와, StateFlow는 비즈니스 로직(ViewModel)과 더 친합니다.

- 안드로이드 생명주기와 옵저버 패턴은 어떻게 연결되어 있나요?  
  → LiveData는 Activity, Fragment의 생명주기를 감지해 자동으로 Observer를 등록/해제합니다.

- 옵저버 패턴의 느슨한 결합이 실제 안드로이드 아키텍처에서 어떤 이점을 주나요?  
  → 모듈 간 독립성을 높여 유지보수성과 테스트 용이성을 개선합니다.

- 옵저버 패턴을 적용한 시스템에서 이벤트 전달 순서를 제어할 필요가 있을 때 어떻게 설계할 수 있나요?  
  → Observer 등록 순서를 고려하거나, 이벤트 큐/버퍼를 두어 제어하거나, 우선순위 기반 옵저버 관리 체계를 도입할 수 있습니다.

---

## 참고 자료
- [데보션 기술 블로그 - 옵저버 패턴의 매커니즘으로 하겠습니다. 근데 이제 Coroutine과 Flow를 곁들인](https://devocean.sk.com/blog/techBoardDetail.do?ID=166675&boardType=techBlog)
- [HAERO 블로그 - 옵저버 패턴 개념 떠먹여드립니다](https://velog.io/@haero_kim/%EC%98%B5%EC%A0%80%EB%B2%84-%ED%8C%A8%ED%84%B4-%EA%B0%9C%EB%85%90-%EB%96%A0%EB%A8%B9%EC%97%AC%EB%93%9C%EB%A6%BD%EB%8B%88%EB%8B%A4)

