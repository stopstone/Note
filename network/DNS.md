# DNS

> DNS(Domain Name System)는 인터넷의 전화번호부와 같은 역할을 하는 시스템입니다.  
> 사람이 읽을 수 있는 도메인 이름(www.google.com)을 컴퓨터가 이해할 수 있는 IP 주소(172.217.175.78)로 변환해줍니다.  
> DNS의 개념과 동작 원리에 대해 정리하였습니다.

---

## 1. DNS란?

DNS(Domain Name System)는 도메인 이름과 IP 주소를 매핑해주는 분산형 데이터베이스 시스템입니다.

인터넷에서 통신하려면 IP 주소가 필요한데, 사람이 숫자로 된 IP 주소를 외우기는 어렵습니다.  
DNS는 이런 문제를 해결하기 위해 `www.google.com`과 같은 사람이 기억하기 쉬운 도메인 이름을 사용할 수 있게 해줍니다.

### DNS가 없다면?

```
사용자: 구글에 접속하고 싶은데...
- DNS 없이: 172.217.175.78을 외워서 입력해야 함
- DNS 있을 때: www.google.com 만 입력하면 됨
```

---

## 2. DNS의 구성 요소

| 구성 요소            | 설명                                                                 |
|---------------------|----------------------------------------------------------------------|
| 도메인 이름(Domain Name) | 사람이 읽을 수 있는 웹사이트 주소 (예: www.naver.com)                |
| IP 주소(IP Address)  | 컴퓨터가 이해하는 실제 서버 주소 (예: 223.130.195.95)                |
| DNS 서버(Name Server) | 도메인과 IP 주소를 저장하고 제공하는 서버                            |
| DNS 레코드(DNS Record) | 도메인에 대한 정보를 담고 있는 데이터 (A, AAAA, CNAME 등)           |

---

## 3. DNS 계층 구조

DNS는 계층적 구조로 되어 있어 효율적인 조회가 가능합니다.

```
                    Root DNS Server (.)
                           |
        +-----------------+-----------------+
        |                 |                 |
    .com DNS          .net DNS          .kr DNS      (TLD: Top-Level Domain)
        |                 |                 |
    google.com        naver.com         co.kr        (Second-Level Domain)
        |                 |                 |
  www.google.com    mail.naver.com    www.naver.co.kr  (Sub Domain)
```

### 도메인 계층

| 계층                | 설명                              | 예시                  |
|--------------------|-----------------------------------|-----------------------|
| Root Domain         | DNS 계층의 최상위 (보이지 않음)    | .                     |
| Top-Level Domain    | 최상위 도메인 (TLD)                | .com, .net, .kr, .org |
| Second-Level Domain | 조직이나 서비스를 나타내는 도메인  | google, naver         |
| Sub Domain          | 세부 서비스를 구분하는 도메인      | www, mail, blog       |

---

## 4. DNS 동작 원리 (DNS 조회 과정)

사용자가 브라우저에 `www.google.com`을 입력했을 때 일어나는 일:

```
1. 사용자가 www.google.com 입력
2. 브라우저 캐시 확인 → 없으면 다음 단계
3. OS 캐시 확인 → 없으면 다음 단계
4. 로컬 DNS 서버(ISP DNS)에 질의
5. Root DNS 서버에 질의 → ".com" 담당 서버 주소 반환
6. TLD DNS 서버(.com)에 질의 → "google.com" 담당 서버 주소 반환
7. Authoritative DNS 서버(google.com)에 질의 → IP 주소 반환
8. 로컬 DNS 서버가 IP 주소를 사용자에게 반환
9. 브라우저가 해당 IP 주소로 HTTP 요청
```

### Recursive Query vs Iterative Query

| 구분              | Recursive Query (재귀적 질의)          | Iterative Query (반복적 질의)     |
|------------------|---------------------------------------|----------------------------------|
| 동작 방식         | DNS 서버가 최종 답을 찾을 때까지 대신 조회 | 각 단계마다 다음 서버 주소만 알려줌 |
| 클라이언트 부담   | 낮음 (DNS 서버가 모든 작업 수행)        | 높음 (클라이언트가 여러 번 질의)  |
| 일반적 사용       | 로컬 DNS 서버 ↔ 클라이언트             | 로컬 DNS 서버 ↔ 다른 DNS 서버들   |

---

## 5. DNS 레코드 타입

DNS 서버는 다양한 타입의 레코드를 저장합니다.

| 레코드 타입 | 설명                                      | 예시                                    |
|------------|-------------------------------------------|-----------------------------------------|
| A          | 도메인 → IPv4 주소 매핑                    | google.com → 172.217.175.78             |
| AAAA       | 도메인 → IPv6 주소 매핑                    | google.com → 2404:6800:4004:825::200e   |
| CNAME      | 도메인 → 다른 도메인 매핑 (별칭)           | www.example.com → example.com           |
| MX         | 메일 서버 지정                            | example.com → mail.example.com          |
| NS         | 네임 서버 지정                            | example.com → ns1.example.com           |
| TXT        | 텍스트 정보 저장 (SPF, DKIM 등)           | example.com → "v=spf1 ..."              |
| PTR        | IP 주소 → 도메인 매핑 (역방향 조회)        | 172.217.175.78 → google.com             |

### A 레코드와 CNAME 레코드의 차이

```
A 레코드 (직접 IP 매핑):
www.example.com → 192.168.1.100

CNAME 레코드 (도메인 별칭):
blog.example.com → www.example.com → 192.168.1.100
shop.example.com → www.example.com → 192.168.1.100

장점: 여러 서브도메인이 하나의 IP를 가리킬 때,
      A 레코드 하나만 변경하면 모든 CNAME이 자동으로 새 IP를 가리킴
```

---

## 6. DNS 캐싱

DNS 조회는 시간이 걸리기 때문에 여러 단계에서 캐싱이 이루어집니다.

| 캐시 위치          | 설명                                    | TTL (유효 시간)       |
|-------------------|----------------------------------------|----------------------|
| 브라우저 캐시      | 브라우저가 최근 조회한 DNS 정보 저장     | 수 분 ~ 수 시간       |
| OS 캐시           | 운영체제 수준에서 DNS 정보 저장          | 수 시간               |
| 로컬 DNS 서버 캐시 | ISP의 DNS 서버가 최근 조회 정보 저장     | DNS 레코드의 TTL 값   |

### TTL(Time To Live)

DNS 레코드마다 TTL 값이 설정되어 있어, 이 시간이 지나면 캐시가 만료되고 새로 조회합니다.

```
example.com.  3600  IN  A  192.168.1.100
                ↑
              TTL (초 단위)
              3600초 = 1시간 동안 캐시 유지
```

---
