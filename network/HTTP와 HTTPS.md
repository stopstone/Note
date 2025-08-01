# HTTP와 HTTPS

> 인터넷 상에서 클라이언트와 서버가 자원을 주고받을 때 사용하는 통신 규약에 대해 알아보겠습니다.

## 1. HTTP (HyperText Transfer Protocol)

### 정의
HTTP는 **HyperText Transfer Protocol**의 약자로,  
인터넷 상에서 클라이언트와 서버가 자원을 주고받을 때 사용하는 통신 규약입니다.

### 특징
- **텍스트 기반 프로토콜**: 모든 데이터가 텍스트 형태로 전송됩니다
- **무상태(Stateless)**: 각 요청은 독립적으로 처리되며, 이전 요청과 무관합니다
- **요청-응답 구조**: 클라이언트가 요청을 보내면 서버가 응답하는 구조입니다

### 동작 방식
```
클라이언트 → HTTP 요청 → 서버
클라이언트 ← HTTP 응답 ← 서버
```

### HTTP 메서드
- **GET**: 리소스 조회
- **POST**: 리소스 생성
- **PUT**: 리소스 전체 수정
- **PATCH**: 리소스 부분 수정
- **DELETE**: 리소스 삭제

### 보안 문제점
HTTP는 텍스트 기반이므로 다음과 같은 보안 문제가 있습니다:

1. **평문 전송**: 데이터가 암호화되지 않아 노출될 위험이 있습니다
2. **중간자 공격**: 네트워크 상에서 데이터를 가로채어 내용을 볼 수 있습니다
3. **무결성 보장 없음**: 데이터가 전송 중에 변조될 수 있습니다

## 2. HTTPS (HyperText Transfer Protocol Secure)

### 정의
HTTPS는 **HyperText Transfer Protocol Secure**의 약자로,  
HTTP에 SSL/TLS 프로토콜을 적용하여 보안을 강화한 통신 규약입니다.

### 특징
- **암호화 통신**: 모든 데이터가 암호화되어 전송됩니다
- **인증**: 서버의 신원을 확인할 수 있습니다
- **무결성**: 데이터가 전송 중에 변조되지 않음을 보장합니다

### 동작 방식
```
클라이언트 → SSL/TLS 핸드셰이크 → 서버
클라이언트 ↔ 암호화된 통신 ↔ 서버
```

### SSL/TLS 핸드셰이크 과정
1. **Client Hello**: 클라이언트가 지원하는 암호화 방식과 랜덤 데이터를 서버에 전송
2. **Server Hello**: 서버가 선택한 암호화 방식과 인증서, 랜덤 데이터를 클라이언트에 전송
3. **키 교환**: 공개키 암호화를 통해 세션 키를 안전하게 교환
4. **암호화 통신**: 세션 키를 사용하여 데이터를 암호화하여 통신

### 보안 장점
1. **기밀성**: 데이터가 암호화되어 전송됩니다
2. **무결성**: 데이터가 전송 중에 변조되지 않음을 보장합니다
3. **인증**: 서버의 신원을 확인할 수 있습니다

## 3. HTTP vs HTTPS 비교

| 구분 | HTTP | HTTPS |
|------|------|-------|
| **프로토콜** | 텍스트 기반 | 암호화 기반 |
| **포트** | 80 | 443 |
| **보안** | 취약함 | 안전함 |
| **속도** | 빠름 | 상대적으로 느림 |
| **인증서** | 불필요 | SSL 인증서 필요 |
| **SEO** | 불리함 | 유리함 |

## 5. 개발 시 고려사항

### HTTP에서 HTTPS로 마이그레이션 시 주의사항
1. **인증서 관리**: SSL 인증서를 적절히 관리해야 합니다
2. **리소스 경로**: 모든 리소스(이미지, CSS, JS)를 HTTPS로 변경해야 합니다
3. **SEO**: 301 리다이렉트를 통해 검색엔진에 변경사항을 알려야 합니다
4. **성능**: HTTP/2나 HTTP/3를 활용하여 성능을 최적화할 수 있습니다

### Android 개발에서의 고려사항
```kotlin
// Network Security Config 설정
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <domain-config cleartextTrafficPermitted="false">
        <domain includeSubdomains="true">example.com</domain>
    </domain-config>
</network-security-config>
```

## 참고 자료
- [Android Network Security Config](https://developer.android.com/training/articles/security-config)
- [HTTP/2](https://http2.github.io/)
- [HTTP/3](https://quicwg.org/) 