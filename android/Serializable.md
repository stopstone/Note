# Serializable

> 안드로이드에서 컴포넌트 간 객체를 전달하거나 저장할 때 직렬화가 필요합니다.  
> Serializable은 Java에서 제공하는 가장 기본적인 직렬화 방식입니다.  

## 1. Serializable이란?

`Serializable`은 Java의 **Marker Interface**입니다.  
직렬화 대상 객체에 이 인터페이스를 구현하면, Java는 해당 객체를 바이트 스트림으로 변환할 수 있게 됩니다.  
Kotlin에서도 `Serializable`은 그대로 사용 가능합니다.

```kotlin
import java.io.Serializable

data class User(
    val id: Int,
    val name: String
) : Serializable
```

* 인터페이스이지만 구현할 메서드는 없습니다.
* `serialVersionUID`를 명시하지 않으면 클래스 변경 시 역직렬화 문제가 발생할 수 있습니다.

```kotlin
companion object {
    private const val serialVersionUID = 1L
}
```

## 2. 사용 예시

안드로이드에서는 `Intent`나 `Bundle`을 통해 Activity 또는 Fragment 간 객체를 전달할 때 사용합니다.

```kotlin
val intent = Intent(this, DetailActivity::class.java)
intent.putExtra("user", user) // user는 Serializable 객체
startActivity(intent)
```

전달받는 쪽에서는 다음과 같이 사용합니다:

```kotlin
val user = intent.getSerializableExtra("user") as? User
```

## 3. 장단점 비교

### 3-1. 장점

* 구현이 간단합니다.
* Java 표준이므로 다양한 환경에서 사용 가능합니다.

### 3-2. 단점

* 성능이 느립니다 (리플렉션 기반).
* 클래스 구조 변경 시 역직렬화 에러 가능성이 있습니다.
* Android 환경에서는 `Parcelable` 대비 비효율적입니다.

## 4. Parcelable과의 비교

| 항목     | Serializable | Parcelable            |
| ------ | ------------ | --------------------- |
| 정의     | Java 표준 직렬화  | Android 전용 직렬화        |
| 성능     | 느림 (리플렉션 기반) | 빠름 (직접 구현)            |
| 구현 난이도 | 매우 쉬움        | 비교적 복잡                |
| 사용처    | 범용 직렬화       | Android 컴포넌트 간 데이터 전달 |

안드로이드에서는 대부분의 경우 `Parcelable` 사용을 권장합니다.  
`Parcelable`은 Android 플랫폼에 최적화되어 있으며, 더 빠른 속도와 낮은 메모리 사용량을 제공합니다.

## 참고 자료

* [Android Developers - Parcelable](https://developer.android.com/reference/android/os/Parcelable)
* [Java SE Documentation - Serializable](https://docs.oracle.com/javase/8/docs/api/java/io/Serializable.html)
