# Intent와 Bundle의 차이

> 안드로이드에서 Activity, Service 등 컴포넌트 간 데이터를 전달하기 위해 사용하는 대표적인 도구로는 Intent와 Bundle이 있습니다.  
> 본 문서에서는 두 객체의 개념과 차이점, 그리고 예제를 중심으로 실무 기반에서 정리해보겠습니다.  
 
## 1. Intent란?

Intent는 컴포넌트 간에 **동작을 요청**하기 위한 메시지 객체입니다.  
Activity, Service, BroadcastReceiver 등을 실행하거나, 특정 동작을 지시할 때 사용됩니다.  
Intent 객체는 아래와 같은 정보를 담을 수 있습니다.  

* Action: 수행할 작업 (예: `ACTION_VIEW`, `ACTION_SEND`)
* Data: URI 형태의 데이터
* Category: 추가 정보 (예: `CATEGORY_DEFAULT`)
* Extras: 추가 데이터 (내부적으로 Bundle 객체)

### 1-1. Intent 예시 (데이터 전달 포함)

```kotlin
val intent = Intent(this, DetailActivity::class.java)
intent.putExtra("username", "지석오빠")
startActivity(intent)
```

## 2. Bundle이란?

Bundle은 안드로이드에서 사용하는 **키-값 기반의 데이터 묶음 객체**입니다.  
다양한 데이터 타입을 저장할 수 있으며, Intent의 extras로 활용되거나 액티비티 상태 저장 시 사용됩니다.

### 2-1. Bundle 예시 (상태 저장 및 전달)

```kotlin
// 상태 저장
override fun onSaveInstanceState(outState: Bundle) {
    super.onSaveInstanceState(outState)
    outState.putString("username", "지석오빠")
}

// 상태 복원
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    val username = savedInstanceState?.getString("username")
}
```

## 3. Intent vs Bundle 차이점

| 항목     | Intent                                | Bundle                    |
| ------ | ------------------------------------- | ------------------------- |
| 목적     | 컴포넌트 간 동작 요청 및 데이터 전달                 | 데이터 묶음 전달 및 저장            |
| 사용 위치  | Activity, Service, Broadcast 등 컴포넌트 간 | Intent의 extras, 상태 저장 등   |
| 구조     | 내부에 Bundle을 포함하고 있음                   | 키-값 쌍의 단순한 구조             |
| 주요 사용법 | `putExtra(key, value)`                | `putString(key, value)` 등 |

## 4. 함께 사용하는 예제

```kotlin
// MainActivity.kt
val bundle = Bundle().apply {
    putString("username", "지석오빠")
    putInt("userId", 1001)
}

val intent = Intent(this, DetailActivity::class.java).apply {
    putExtras(bundle)
}

startActivity(intent)

// DetailActivity.kt
val bundle = intent.extras
val username = bundle?.getString("username")
val userId = bundle?.getInt("userId")
```

## 참고 자료

* [공식 Android 개발자 문서 - Intent](https://developer.android.com/reference/android/content/Intent)
* [공식 Android 개발자 문서 - Bundle](https://developer.android.com/reference/android/os/Bundle)
