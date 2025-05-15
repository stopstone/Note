# Parcelable

> Parcelable은 안드로이드에서 액티비티나 프래그먼트 간 데이터를 전달할 때 사용하는 직렬화 방식 중 하나로,  
> 성능과 효율성 측면에서 가장 많이 사용됩니다.

## 1. Parcelable이란?

안드로이드에서 객체를 다른 컴포넌트(Activity, Fragment 등)로 전달하려면 객체를 **직렬화(Serialization)** 해야 합니다.  
`Parcelable`은 안드로이드에서 제공하는 고성능 직렬화 인터페이스입니다.

### 1-1. 왜 필요한가요?

* 액티비티 간 화면 전환 시 객체 전달
* Bundle에 객체 담기
* 프로세스 간 통신(IPC) 등에 사용

### 1-2. 특징

* Android에 최적화되어 매우 빠름
* 직접 구현해야 하므로 코드가 복잡할 수 있음
* 기본적으로는 메모리 내에서 객체를 byte stream으로 바꾸고, 다시 복원함

## 2. Parcelable 구현 방법

```kotlin
import android.os.Parcel
import android.os.Parcelable

data class User(
    val name: String,
    val age: Int
) : Parcelable {
    constructor(parcel: Parcel) : this(
        parcel.readString() ?: "",
        parcel.readInt()
    )

    override fun writeToParcel(parcel: Parcel, flags: Int) {
        parcel.writeString(name)
        parcel.writeInt(age)
    }

    override fun describeContents(): Int = 0

    companion object CREATOR : Parcelable.Creator<User> {
        override fun createFromParcel(parcel: Parcel): User {
            return User(parcel)
        }

        override fun newArray(size: Int): Array<User?> {
            return arrayOfNulls(size)
        }
    }
}
```

## 3. @Parcelize를 활용한 간편한 구현

`@Parcelize` 애노테이션을 사용하면 번거로운 메서드 구현 없이 간단하게 `Parcelable`을 사용할 수 있습니다.

### 3-1. Gradle 설정

```kotlin
plugins {
    id("kotlin-parcelize")
}
```

* `build.gradle.kts` 또는 `build.gradle`의 `plugins` 블록에 추가합니다.

### 3-2. 사용 예시

```kotlin
import android.os.Parcelable
import kotlinx.parcelize.Parcelize

@Parcelize
data class User(
    val name: String,
    val age: Int
) : Parcelable
```

* `writeToParcel`, `describeContents`, `CREATOR` 모두 생략 가능

### 3-3. 주의사항

* 모든 프로퍼티가 `Parcelable` 가능해야 함
* Kotlin의 `data class`에만 적용 가능
* Java 클래스에는 사용할 수 없음
* `import kotlinx.parcelize.Parcelize`로 정확히 불러와야 함 (구버전 `kotlinx.android.parcel`은 deprecated)

## 4. Parcelable vs Serializable

| 항목     | Parcelable | Serializable          |
| ------ | ---------- | --------------------- |
| 성능     | 매우 빠름      | 상대적으로 느림              |
| 구현 난이도 | 복잡함        | 간단함                   |
| 사용성    | 안드로이드 전용   | 자바 표준                 |
| GC 영향  | 적음         | 비교적 큼 (reflection 사용) |

## 5. Parcelable 사용 예시

```kotlin
// Intent로 데이터 전달
val intent = Intent(this, DetailActivity::class.java)
intent.putExtra("user", user)
startActivity(intent)

// DetailActivity에서 받기
val user = intent.getParcelableExtra<User>("user")
```

## 6. 주의할 점

* `null` 처리를 반드시 고려해야 합니다.
* `writeToParcel`, `createFromParcel`은 순서를 정확히 맞춰야 함
* 모든 필드가 `Parcelable` 가능해야 함 (예: List 안의 요소도 모두 `Parcelable`이어야 함)

### 참고자료

* [Android Developers - Parcelable](https://developer.android.com/reference/android/os/Parcelable)
