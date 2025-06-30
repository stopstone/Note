# Activity와 Fragment의 차이

> 안드로이드 앱을 구성할 때 가장 기본이 되는 단위는 `Activity`와 `Fragment`입니다.  
> 하지만 둘은 단순히 UI를 보여주는 컴포넌트가 아니라, 책임과 사용 맥락이 다르기 때문에 적절하게 구분해서 사용하는 것이 중요합니다.  

## 1. Activity란?

`Activity`는 안드로이드 애플리케이션의 **UI를 담당하는 기본 단위**입니다.   
앱을 실행했을 때 최초로 사용자에게 보여지는 화면이 보통 하나의 `Activity`입니다.  

### 주요 특징

* 하나의 앱은 여러 개의 `Activity`를 가질 수 있음
* 각 `Activity`는 독립적인 생명주기를 가짐
* 운영체제가 관리하는 최상위 단위 (OS의 앱 스택에 쌓임)
* 인텐트를 통해 다른 `Activity`로 전환 가능

### 언제 사용하는가?

* 사용자 흐름상 **전혀 다른 역할을 수행하는 화면**이 필요한 경우
* 외부 앱과의 명확한 경계를 두고 화면을 분리할 때 (`ACTION_VIEW`, `ACTION_SEND` 등)
* 다중 task를 명확히 분리해야 할 경우

## 2. Fragment란?

`Fragment`는 `Activity` 내부에서 UI를 구성하기 위한 **하위 컴포넌트**입니다.   
`Activity` 안에 포함되어야만 화면에 표시할 수 있습니다.  

### 주요 특징

* `Activity`에 종속적임
* 재사용 가능한 UI 단위로 설계할 수 있음
* 화면 회전에 강한 구성 가능 (ViewModel + Fragment)
* 하나의 `Activity` 안에 여러 `Fragment`를 동적으로 교체 가능

### 언제 사용하는가?

* 하나의 화면에서 **여러 개의 UI 조각**을 필요로 할 때
* 태블릿과 같은 큰 화면에서 **멀티패널 UI**를 구성할 때
* `Navigation Component`를 활용한 **화면 간 전환**을 단순하게 관리할 때
* 다중 화면 구조를 하나의 `Activity` 안에서 구현하고 싶을 때

## 3. Activity vs Fragment 요약 비교

| 항목    | Activity        | Fragment              |
| ----- | --------------- | --------------------- |
| 종속성   | 독립적             | Activity에 종속적         |
| 생명주기  | 운영체제가 직접 관리     | Activity의 생명주기에 따라 변화 |
| 재사용성  | 낮음              | 높음 (동적 교체 가능)         |
| UI 구성 | 전체 화면 단위        | 부분 UI 단위              |
| 화면 전환 | 인텐트 기반 전환       | 트랜잭션 기반 전환            |
| 사용 예  | 로그인 화면, 설정 화면 등 | 리스트/디테일 구조, 탭 화면 등    |

## 참고자료

* [Android 공식 문서 - Activity](https://developer.android.com/guide/components/activities/intro)
* [Android 공식 문서 - Fragment](https://developer.android.com/guide/fragments)
