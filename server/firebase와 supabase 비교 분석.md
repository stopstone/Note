# Firebase와 Supabase, 비교 분석

> 혼자 앱을 만들면서도 서버 구축 없이 빠르게 백엔드를 갖추고 싶을 때, Supabase와 Firebase는 가장 많이 고민되는 선택지입니다.    
> 해당 문서에서는 안드로이드 개발자 입장에서 이 두 플랫폼을 실제로 사용할 때 어떤 차이가 있고, 어떤 기준으로 선택하면 좋은지를 정리했습니다.  

## Supabase vs Firebase: 핵심 비교

| 항목 | Firebase | Supabase |
|------|----------|----------|
| 데이터베이스<br>(데이터 구조의 유연성) | Firestore / Realtime DB (NoSQL, 문서 기반) | PostgreSQL (관계형, SQL 기반) |
| 실시간 기능<br>(데이터 동기화 처리 방식) | Firestore의 실시간 동기화 기능 | WebSocket 기반 Realtime 서버 (Logical Replication 활용) |
| 오프라인 지원<br>(네트워크 끊김 대응력) | 강력한 캐싱 및 자동 동기화 지원 | 미지원 또는 실험적 기능 |
| 서버리스 함수<br>(백엔드 로직 처리 방식) | Cloud Functions (Node.js 지원) | Edge Functions (Deno, TypeScript) |
| 인증<br>(로그인/권한 관리 유연성) | Firebase Auth, 다양한 소셜 로그인 지원 (Google, Apple, Facebook 등 OAuth 제공자 폭넓게 지원) | GoTrue 기반, RLS 지원 |
| 호스팅<br>(서비스 배포 방식) | Google Cloud 기반, 자체 호스팅 불가 | 오픈소스, 자체 호스팅 가능 |
| 가격 모델<br>(비용 예측 가능성) | 사용량 기반, 예측 어려움 | 저장 용량 기반, 예측 가능한 요금 |
| 오픈소스 여부 | 아니오 | 예 |
| 안드로이드 SDK 지원 | 공식 SDK 제공 | 커뮤니티 라이브러리(supabase-kt) 활용 가능 |

## 안드로이드 개발에서의 선택 기준

### Firebase를 선택해야 하는 경우
- 빠른 MVP 개발이나 프로토타이핑이 필요할 때<br>예: 해커톤, 빠르게 검증해야 할 스타트업 아이디어
- 실시간 데이터 동기화와 오프라인 지원이 중요한 앱을 개발할 때
- Google 서비스와의 통합이 필요한 경우
- 다양한 언어 지원과 광범위한 커뮤니티를 활용하고 싶을 때

### Supabase를 선택해야 하는 경우
- 복잡한 관계형 데이터를 다뤄야 하는 경우<br>예: 쇼핑몰 앱, 예약 시스템, 주문-배송 추적 등
- SQL 쿼리를 활용한 정교한 데이터 처리가 필요한 경우
- 오픈소스를 선호하고, 자체 호스팅을 고려할 때
- 예측 가능한 비용 구조를 원할 때
- **Kotlin 기반으로 Supabase를 사용하고 싶을 때 (supabase-kt 라이브러리 제공)**

## 성능 비교
- Supabase는 읽기 성능에서 최대 4배, 쓰기 성능에서 최대 3.1배 빠르다는 벤치마크 결과가 있습니다. (https://supabase.com/alternatives/supabase-vs-firebase)
- Firebase는 실시간 데이터 처리와 오프라인 지원 측면에서 강점을 보입니다. (https://www.bytebase.com/blog/supabase-vs-firebase)
