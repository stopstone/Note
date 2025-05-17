# Kotlin typealias

> Kotlin의 `typealias`는 타입에 별칭을 붙여주는 기능으로, 복잡하거나 반복적인 타입 선언을 간결하게 만들 수 있습니다.  
> 이 글에서는 `typealias`의 정의, 사용 예시, 주요 특징, 그리고 실무에서의 활용 시점을 중심으로 정리합니다.

## 1. typealias란?

`typealias`는 Kotlin에서 기존 타입에 *별칭(Alias)* 을 붙여주는 기능입니다.
이름 그대로 타입을 ‘다른 이름으로 부르는’ 것뿐이며, **기존 타입과 완전히 동일하게 작동**합니다.

```kotlin
typealias UserMap = Map<Int, String>
```

위 선언 이후부터는 `Map<Int, String>`을 `UserMap`으로 대신 쓸 수 있게 됩니다.


## 2. 사용 예시

### 2-1. 제네릭 타입 단축

```kotlin
typealias StringList = List<String>

fun printNames(names: StringList) {
    names.forEach { println(it) }
}
```

### 2-2. 람다 타입 간결화

```kotlin
typealias OnClick = (Int) -> Unit

fun setClickListener(callback: OnClick) {
    callback(123)
}
```

### 2-3. 중첩 타입 정리

```kotlin
class Repository {
    sealed class Result<T> {
        data class Success<T>(val data: T) : Result<T>()
        data class Error<T>(val message: String) : Result<T>()
    }
}

typealias RepoResult<T> = Repository.Result<T>
```

## 3. typealias 특징

* **런타임에는 영향 없음**: `typealias`는 컴파일 타임에만 존재하고, **런타임에서는 실제 타입으로 치환**됨.
* **확장 함수, 제네릭 타입, 람다식 모두에 사용 가능**.
* **새로운 타입이 아님**: Swift의 `typealias`, TypeScript의 `type`처럼 실제 타입을 대체하진 않고, 참조하는 역할임.

## 4. 실무에서의 활용 포인트

* **코드 가독성 향상**: 긴 타입을 줄여서 코드의 목적을 더 명확하게 표현할 수 있습니다.
* **도메인 의미 부여**: `typealias UserId = Int`처럼 의미 있는 이름으로 타입에 도메인 문맥을 부여할 수 있습니다.
* **중복 제거**: 자주 쓰이는 람다나 제네릭 타입을 재사용성 있게 선언할 수 있습니다.
* 

## 참고 자료

* [Android Developers - Kotlin typealias](https://developer.android.com/kotlin/learn#type-aliases)
* [Kotlin 공식 문서 - typealias](https://kotlinlang.org/docs/type-aliases.html)
