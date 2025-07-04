# PNG와 SVG의 차이

> PNG와 SVG는 많이 사용되는 이미지 형식입니다.  
> 하지만 구조와 표현 방식이 다르기 때문에 각각의 특성을 이해하고 상황에 맞게 활용하는 것이 중요합니다.  
> 이 글에서는 PNG와 SVG의 기본 개념, 차이점, 그리고 언제 어떤 포맷을 선택하면 좋은지 알아보겠습니다.  

## 1. PNG란?

**PNG(Portable Network Graphics)** 는 픽셀 기반의 **비손실 압축**을 지원하는 래스터 이미지 포맷입니다.

### 특징

* 비손실 압축 포맷
* 투명도(알파 채널) 지원
* 픽셀 기반 이미지라 확대 시 깨짐 발생
* 배경 이미지, 사진, 섬세한 그래픽에 적합
* 파일 크기가 상대적으로 큼

### 사용 예시

* 앱 아이콘, 스크린샷, 정적인 배너 이미지 등
* 고정된 해상도에 맞춘 UI 배경 요소

## 2. SVG란?

**SVG(Scalable Vector Graphics)** 는 XML 기반의 **벡터 이미지 포맷**으로, 도형, 선, 텍스트 등을 수학적으로 표현합니다.

### 특징

* 벡터 기반이라 **확대/축소 시에도 깨짐 없음**
* 코드 기반이라 스타일 및 애니메이션 처리 가능
* 일반적으로 PNG보다 파일 크기가 작음
* 복잡한 그래픽에는 부적합할 수 있음

### 사용 예시

* 아이콘, 단순한 UI 그래픽 요소
* 다양한 해상도 지원이 필요한 화면 구성
* 테마에 따라 색상을 쉽게 변경해야 하는 경우

## 3. PNG vs SVG 비교

| 구분      | PNG         | SVG                     |
| ------- | ----------- | ----------------------- |
| 타입      | 래스터(비트맵)    | 벡터                      |
| 압축      | 비손실 압축      | XML 텍스트 기반              |
| 해상도 의존성 | 해상도 고정      | 해상도 독립적                 |
| 확대 시 품질 | 깨짐 발생       | 품질 유지                   |
| 투명도 지원  | O           | X (알파 채널 없음, 투명 배경은 가능) |
| 애니메이션   | 불가능         | 가능                      |
| 렌더링 속도  | 빠름          | 복잡한 경우 느릴 수 있음          |
| 사용 용도   | 사진, 복잡한 이미지 | 아이콘, UI 요소, 반응형 그래픽     |

## 4. 언제 어떤 포맷을 선택할까?

* **아이콘, 단순한 로고, 벡터 UI 요소**: SVG 추천

  * 확대 시 깨짐 없이 다양한 해상도 대응
  * 코드에서 색상 및 크기 변경 가능

* **사진, 섬세한 이미지, 배경 그래픽**: PNG 추천

  * 픽셀 단위로 세밀한 표현 가능
  * 디자인된 고정 이미지에 적합

## 5. Android에서의 사용 팁

* Android에서는 벡터 이미지 리소스로 `VectorDrawable`을 주로 사용하며, SVG는 `VectorDrawable`로 변환해서 사용합니다.
* SVG → VectorDrawable 변환 도구: Android Studio의 `New > Vector Asset` 기능 이용
* 복잡한 SVG는 변환 시 오류가 발생할 수 있으니 단순한 구조의 SVG만 사용 권장
* 너무 복잡한 이미지는 PNG로 처리하는 것이 오히려 성능에 유리할 수 있음

## 참고자료

* [Android Developers - Vector drawables](https://developer.android.com/guide/topics/graphics/vector-drawable-resources)
* [MDN - SVG 소개](https://developer.mozilla.org/ko/docs/Web/SVG)
* [W3C - PNG Specification](https://www.w3.org/TR/PNG/)
