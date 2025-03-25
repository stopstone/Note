# 안드로이드 액티비티 라이프사이클

안드로이드 앱에서 액티비티는 4대 구성요소 중 하나로 UI와 직접 연결되는 기본 단위입니다.  
라이프사이클을 이해하면 앱의 상태를 안전하게 관리하고, 예기치 않은 종료나 버그를 방지할 수 있습니다.   
이 글은 액티비티 라이프사이클이 어떻게 구성되었는지, 각 단계가 어떤 경우에 호출되는지를 알아봅니다.  
또한, 액티비티 라이프사이클에서 질문할 수 있는 내용을 정리합니다.

---

## 액티비티 라이프사이클이란?

액티비티는 생성부터 소멸까지 일련의 상태 전이를 거치며, 각 상태에 진입할 때 특정 콜백 메서드가 호출됩니다.  
대표적으로 다음과 같은 7가지 콜백 메서드가 존재합니다.

호출되는 콜백 메서드는 아래와 같습니다.

- onCreate()
- onStart()
- onResume()
- onPause()
- onStop()
- onRestart()
- onDestroy()

<p align="center">
  <img src="https://github.com/user-attachments/assets/36358955-228e-4202-8709-03c7c80c1e04" width="360" />
</p>

---

### 라이프사이클을 알아야 하는 이유
- 전화 수신, 홈 버튼, 화면 회전 등 다양한 상황에 안전하게 대응할 수 있음
- 메모리 누수 방지 및 리소스 해제를 위한 적절한 위치를 파악할 수 있음
- 사용자 상태 저장 및 복구에 필수적인 개념

## 라이프사이클 콜백 메서드

### onCreate()
- 액티비티가 최초 생성될 때 호출
- 레이아웃 설정, 의존성 초기화, savedInstanceState 복원 등의 작업 수행

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    Log.d("LifeCycle", "onCreate")
}
```

### onStart()
- 액티비티가 사용자에게 보이기 시작할 때 호출

```kotlin
override fun onStart() {
    super.onStart()
    Log.d("LifeCycle", "onStart")
}
```

### onResume()
- 액티비티가 포그라운드 상태로 진입해 사용자와 상호작용 가능해짐

```kotlin
override fun onResume() {
    super.onResume()
    Log.d("LifeCycle", "onResume")
}
```

### onPause()
- 액티비티가 포커스를 잃었을 때 호출 (예: 전화 수신, 반투명 액티비티 등장)

```kotlin
override fun onPause() {
    super.onPause()
    Log.d("LifeCycle", "onPause")
}
```

### onStop()
- 액티비티가 사용자에게 완전히 보이지 않을 때 호출됨

```kotlin
override fun onStop() {
    super.onStop()
    Log.d("LifeCycle", "onStop")
}
```

### onRestart()
- 정지된 액티비티가 다시 시작될 때 호출됨

```kotlin
override fun onRestart() {
    super.onRestart()
    Log.d("LifeCycle", "onRestart")
}
```

### onDestroy()
- 액티비티가 종료되기 직전에 호출됨

```kotlin
override fun onDestroy() {
    super.onDestroy()
    Log.d("LifeCycle", "onDestroy")
}
```

---

## 상태 저장과 복원

### onSaveInstanceState()
- 일시적으로 소멸될 가능성이 있는 상태를 저장함

```kotlin
override fun onSaveInstanceState(outState: Bundle) {
    outState.putString("text", textView.text.toString())
    super.onSaveInstanceState(outState)
}
```

### onRestoreInstanceState()
- 저장된 상태를 복원함

```kotlin
override fun onRestoreInstanceState(savedInstanceState: Bundle) {
    super.onRestoreInstanceState(savedInstanceState)
    val savedText = savedInstanceState.getString("text")
    textView.text = savedText
}
```

---

## 라이프사이클 흐름 예시

### 최초 실행
- onCreate → onStart → onResume

### 새 액티비티 실행 시
- 기존 액티비티: onPause → onStop
- 새 액티비티: onCreate → onStart → onResume

### 액티비티 종료 후 복귀
- 종료: onPause → onStop → onDestroy
- 복귀: onRestart → onStart → onResume

### 화면 회전
- onPause → onStop → onDestroy → onCreate → onStart → onResume

---

## 프로세스 상태와 종료 가능성 (출처: Android 공식 문서)

| 상태 | 액티비티 상태 | 종료 가능성 |
|------|----------------|--------------|
| 포그라운드 | onResume | 가장 낮음 |
| 표시됨 | onStart / onPause | 낮음 |
| 백그라운드 | onStop | 높음 |
| 소멸됨 | onDestroy | 가장 높음 |

---

## 질문

<details>
<summary>액티비티 A에서 B를 실행할 때 호출되는 라이프사이클 순서는?</summary>

A: onPause →  
B: onCreate → onStart → onResume →  
A: onStop

</details>

<details>
<summary>B에서 A로 돌아올 때 라이프사이클 순서는?</summary>

B: onPause →  
A: onRestart → onStart → onResume →  
B: onStop → onDestroy

</details>

<details>
<summary>구성 변경(화면 회전) 시 어떤 콜백들이 호출되는가?</summary>

onPause → onStop → onDestroy →  
onCreate → onStart → onResume

</details>

<details>
<summary>Dialog를 띄웠을 때 기존 액티비티 라이프사이클은 어떤가?</summary>

Dialog는 액티비티 내부 UI로 간주되므로 라이프사이클 변화가 없음.  
일반적으로 onPause도 호출되지 않음.

</details>

<details>
<summary>투명한 액티비티와 불투명한 액티비티의 차이는?</summary>

- **투명한 액티비티**: 기존 액티비티는 onPause까지만 호출되고 onStop은 호출되지 않음  
- **불투명한 액티비티**: 기존 액티비티는 onPause → onStop까지 호출됨

</details>


---

## 참고 자료
- [Android 공식 문서](https://developer.android.com/guide/components/activities/activity-lifecycle)
- [안드로이드 액티비티 라이프사이클 (Life Cycle)](https://brunch.co.kr/@mystoryg/80)
- [박상권의 삽질 블로그 - 지원자 95%가 틀리는 startActivity() 라이프사이클, 당신도 예외는 아닙니다](https://medium.com/%EB%B0%95%EC%83%81%EA%B6%8C%EC%9D%98-%EC%82%BD%EC%A7%88%EB%B8%94%EB%A1%9C%EA%B7%B8/%EC%A7%80%EC%9B%90%EC%9E%90-95-%EA%B0%80-%ED%8B%80%EB%A6%AC%EB%8A%94-startactivity-%EB%9D%BC%EC%9D%B4%ED%94%84%EC%82%AC%EC%9D%B4%ED%81%B4-%EB%8B%B9%EC%8B%A0%EB%8F%84-%EC%98%88%EC%99%B8%EB%8A%94-%EC%95%84%EB%8B%99%EB%8B%88%EB%8B%A4-ed0947a48d6)

