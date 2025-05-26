# SharedPreferences와 DataStore 차이점 정리

> Android에서 Key-Value 형태의 데이터를 저장할 때 많이 사용하는 SharedPreferences와 DataStore 가 있습니다.
> 두 방식의 주요 차이점을 비교합니다.

## 1. 비교

| 항목              | SharedPreferences        | DataStore (Preferences)        |
| --------------- | ------------------------ | ------------------------------ |
| **동기/비동기**      | 동기 (apply도 내부적으로 디스크 접근) | 비동기 (Coroutine 기반 처리)          |
| **I/O 처리 위치**   | UI 스레드에서 동작 가능 (ANR 위험)  | 항상 백그라운드에서 안전하게 처리             |
| **트랜잭션 안전성**    | 낮음 (동시 접근 시 오류 가능)       | 높음 (원자성 보장)                    |
| **데이터 관찰**      | 불가능                      | 가능 (Flow로 실시간 데이터 관찰 가능)       |
| **타입 안정성**      | 없음 (모든 값이 String 기반)     | 낮음 (Preferences 기준, Proto는 높음) |
| **구조화된 데이터 저장** | 불가능                      | 가능 (Proto DataStore 사용 시)      |
| **마이그레이션 용이성**  | 직접 수동 처리 필요              | 유연한 마이그레이션 로직 구현 가능            |
| **공식 권장 여부**    | 권장하지 않음                  | Android 공식 권장 방식               |

## 2. 결론

이제는 SharedPreferences 대신 DataStore를 사용하는 게 자연스러운 흐름이에요.  
SharedPreferences는 간단하지만, UI 스레드에서 작동하는 구조나 동시 접근 이슈 같은 단점이 명확하죠.  
반면 DataStore는 비동기 처리와 Flow 기반 관찰 기능, 트랜잭션 안전성까지 모두 갖춘 현대적인 저장소입니다.  
