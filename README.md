# Note
### 안드로이드, 코틀린 이론 정리 노트
안드로이드, 코틀린을 공부하며 키워드들을 정리하는 저장소
키워드와 설명을 간결하게 정리
하루 5개씩 정리해보기


## Kotlin

- 널 안정성
- 데이터 클래스
- 확장 함수
- 람다 표현식
- 코루틴
- 스마트 캐스트
- 컴패니언 객체
- 고차 함수
- 프로퍼티
- Mutable, Immutable
- 지연 초기화
- 봉인 클래스
- 객체 표현식과 선언
- 위임
- 인라인 함수
- 타입 별칭
- 범위 지정 함수
- 중위 함수
- 연산자 오버로딩
- 제네릭
- 리시버와 함께 쓰는 함수 타입

## Android

- 액티비티 (Activity):
사용자 인터페이스를 가진 단일 화면, 앱의 기본 구성 요소
- 프래그먼트 (Fragment):
액티비티 내에서 재사용 가능한 UI 컴포넌트
- 인텐트 (Intent):
액티비티, 서비스, 브로드캐스트 리시버 등 안드로이드의 주요 컴포넌트들 사이에서 정보를 전달
- 서비스 (Service):
백그라운드에서 장기 실행 작업을 수행하는 컴포넌트
- 브로드캐스트 리시버 (Broadcast Receiver):
시스템 전체 이벤트를 수신하고 반응하는 컴포넌트
- 컨텐트 프로바이더 (Content Provider):
앱 간 데이터 공유를 관리하는 컴포넌트
- 뷰 (View):
UI를 구성하는 기본 빌딩 블록
- 레이아웃 (Layout):
뷰 그룹을 사용하여 UI 구조를 정의
- 리사이클러뷰 (RecyclerView):
대량의 데이터 세트를 효율적으로 표시하는 뷰
- DiffUtil
androidx 패키지에 포함되어 두 리스트 간의 차이를 계산하고, 새로운 리스트로 변경하기 위한 작업목록을 반영하는 것에 도움을 주는 유틸리티 클래스
- 뷰모델 (ViewModel):
UI 관련 데이터를 저장하고 관리
- 라이브데이터 (LiveData):
데이터의 변경을 관찰할 수 있는 데이터 홀더 클래스
- 데이터 바인딩 (Data Binding):
레이아웃의 UI 컴포넌트를 데이터 소스와 연결
- 코루틴 (Coroutines):
비동기 프로그래밍을 위한 경량 스레드
- 의존성 주입 (Dependency Injection):
클래스 간의 의존성을 외부에서 주입하는 디자인 패턴
- Jetpack Compose:
선언적 UI
- Room:
SQLite 추상화 레이어를 제공하는 데이터베이스 라이브러리
- Retrofit:
REST API 통신을 위한 HTTP 클라이언트 라이브러리
- Gson/Moshi:
JSON 파싱 라이브러리
- Context:
앱의 현재 상태에 대한 인터페이스
- SharedPreferences:
간단한 키-값 쌍의 데이터 저장소
