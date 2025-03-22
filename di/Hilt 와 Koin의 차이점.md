# Hilt vs Koin 차이점 정리

## Hilt와 Koin의 차이 개요
Hilt와 Koin은 안드로이드 개발에서 널리 사용되는 DI(Dependency Injection, 의존성 주입) 프레임워크입니다.   
이 문서에서는 두 기술의 특징과 차이점을 비교하여 정리합니다.  
또한, 두 프레임워크 중 어떤 기준으로 선택하면 좋을지 설명합니다.

---

## Hilt vs Koin 비교

| 항목 | **Hilt** | **Koin** |
|------|-----------|-----------|
| 주입 시점 | 컴파일 타임에 의존성 주입 수행 | 런타임에 의존성 주입 수행 |
| DI 방식 | Dagger2 기반 (컴파일 타임 DI) | 런타임 DI 기반이지만, 내부적으로는 Service Locator 패턴을 함께 사용 |
| 구성 방식 | Annotation 기반 (@Inject, @Module 등) | Kotlin DSL 기반 (module { single { ... } }) |
| ViewModel DI | @HiltViewModel + @Inject로 바로 주입 가능 | by viewModel() 사용 또는 별도 선언 필요 |
| Android 통합 | Android 컴포넌트와의 통합 우수 | Android 생명주기와의 통합 약함 |
| 보일러플레이트 코드 | 최소화 | DSL 기반으로 간결함 |
| 테스트 환경 | Scope 기반 테스트 지원 | Stub/Mock 중심, 구조적 한계 있음 |
| 오류 발생 시점 | 컴파일 타임 오류 확인 가능 | 런타임 오류 발생 가능성 높음 |
| 성능 | 고정 객체 생성, 최적화 용이 | 리플렉션 사용, 대규모 앱에서 성능 저하 우려 |
| Kotlin Multiplatform | 미지원 | KMP에서 사용 가능, iOS/웹 등 공유 가능 |
| 어노테이션 지원 | 기본 제공 | Koin Annotations 사용 가능 (@Single, @Factory 등) |
| 컴파일 타임 DI 체크 | 기본 지원 | Gradle 설정으로 가능 (KOIN_CONFIG_CHECK) |

---

## 사용 추천 기준

| 상황 | 추천 프레임워크 | 설명 |
|------|------------------|------|
| 소규모 프로젝트 | Koin | 빠르게 적용 가능하고 설정이 간단함. Kotlin 환경에 적합. |
| 대규모 / 멀티모듈 프로젝트 | Hilt | 명확한 DI 구조와 높은 안정성. 컴파일 타임 검증 가능. |
| 빠른 빌드 시간이 중요할 때 | Koin | 어노테이션 프로세싱을 사용하지 않아 빌드 시간이 짧음. |
| 보일러플레이트 코드를 줄이고 싶을 때 | Hilt | Dagger 기반이지만 설정이 간단하고 공식 지원. |
| 테스트 코드 작성이 중요한 경우 | Hilt | Scope 기반 DI로 테스트 작성에 유리. |
| DI에 처음 입문하는 경우 | Koin | DSL 기반으로 간단하고 직관적. 러닝커브 낮음. |
| Kotlin Multiplatform을 사용하는 경우 | Koin | 순수 Kotlin 기반으로 KMP 지원. |
| Google 공식 기술을 따르고 싶을 때 | Hilt | Jetpack 및 AndroidX 호환성 우수, 공식 문서에서 지원. |

---

## 종합 정리

| 프레임워크 | 특징 | 장점 | 단점 |
|------------|--------|--------|--------|
| **Hilt** | 컴파일 타임 DI, Dagger2 기반 | 성능 우수, 안정성, Android 공식 지원, 명확한 그래프 구성 | 러닝커브 있음, 빌드 시간 증가 (KAPT 사용), KMP 미지원 |
| **Koin** | 런타임 DI + Service Locator 혼합 구조 | 빠른 적용, 설정 간단, KMP 호환, 어노테이션 기반 코드 가능 | 성능 이슈, 런타임 에러 가능성, Android 생명주기와 통합 미흡 |

---

## 어떤 것을 선택해야 할까?

- **Kotlin Multiplatform 프로젝트인가요?** → **Koin**: 순수 Kotlin 기반이므로 Android가 아니어도 사용 가능하며, iOS, Web 등 재사용 가능
- **런타임 성능이 중요한 대규모 앱인가요?** → **Hilt**: 컴파일 타임 DI로 안정성 및 성능 확보에 유리
- **Android 생명주기와 통합이 중요한가요?** → **Hilt**: Activity, Fragment와의 통합성이 우수함
- **Google 기술 생태계를 따르고 싶은가요?** → **Hilt**: Jetpack과의 통합, 공식 문서 가이드, 장기적인 안정성 기대 가능
- **간단한 프로젝트나 학습 목적인가요?** → **Koin**: 빠르게 적용 가능하고 DSL 기반으로 러닝커브가 낮음
- **동료들이 어떤 걸 익숙하게 쓰고 있나요?** → 협업 환경에 따라 Hilt 또는 Koin 선택

---

## 참고 지료
- [Hilt 공식 문서](https://developer.android.com/training/dependency-injection/hilt-android)
- [Koin 공식 사이트](https://insert-koin.io/)
- [Velog: Koin vs Dagger-Hilt 비교 정리](https://velog.io/@mjini/Koin-Dagger-Hilt-%EB%B9%84%EA%B5%90-%EC%A0%95%EB%A6%AC)
- [뱅크샐러드 블로그: Koin to Hilt 마이그레이션 후기](https://blog.banksalad.com/tech/migrate-from-koin-to-hilt/)
- [Tistory: Hilt vs Koin 비교](https://faith-developer.tistory.com/233)
- [패스트캠퍼스 클린아키텍처 강의](https://fastcampus.co.kr/dev_online_clean)

  ## 관련 자료
- [Hilt 개념](https://github.com/stopstone/Note/blob/main/di/Hilt.md)
- [Koin 개념](https://github.com/stopstone/Note/blob/main/di/Koin.md)
