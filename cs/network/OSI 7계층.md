# OSI 7계층

> 네트워크 통신은 우리가 흔히 사용하는 웹 브라우징, 이메일 전송, 파일 다운로드와 같은 작업의 기반이 되는 중요한 개념입니다.  
> 이 문서에서는 OSI 7계층이 무엇인지와 각 계층이 어떤 역할을 수행하는지를 설명합니다. 

![image](https://github.com/user-attachments/assets/25982b0f-d324-4ed3-bd0c-a6cd7a8d47e4)

## 1. OSI 7계층이란?

OSI 7계층(Open Systems Interconnection Model)은 네트워크 통신을 7개의 단계로 나눈 개념 모델입니다.  
각 계층은 독립적인 역할을 가지며, 하위 계층에 의존하고 상위 계층에 기능을 제공합니다.

| 계층 | 이름       | 설명                  | 예시                |
| -- | -------- | ------------------- | ----------------- |
| 7  | 응용 계층    | 사용자와 가장 가까운 계층      | HTTP, FTP, SMTP   |
| 6  | 표현 계층    | 데이터 포맷, 인코딩, 암호화 처리 | JPEG, MPEG, TLS   |
| 5  | 세션 계층    | 세션 제어, 연결 유지        | API 세션, TLS 핸드셰이크 |
| 4  | 전송 계층    | 데이터 신뢰성, 순서 보장      | TCP, UDP          |
| 3  | 네트워크 계층  | 라우팅, IP 주소 할당       | IP, ICMP          |
| 2  | 데이터링크 계층 | MAC 주소, 프레임 전송      | Ethernet, ARP     |
| 1  | 물리 계층    | 전기 신호, 하드웨어 전송      | LAN 케이블, 허브       |

---

## 2. 계층별 상세 설명

### 7. 응용 계층 (Application Layer)

* 최종 사용자에게 가장 가까운 계층입니다.
* 웹, 이메일, 파일 전송 등 사용자 애플리케이션이 위치합니다.
* 예: HTTP, SMTP, FTP

### 6. 표현 계층 (Presentation Layer)

* 데이터를 서로 이해할 수 있는 형태로 변환합니다.
* 암호화/복호화, 인코딩/디코딩을 담당합니다.
* 예: SSL/TLS, Base64

### 5. 세션 계층 (Session Layer)

* 통신을 위한 세션을 생성하고 관리하며, 연결을 유지하거나 종료하는 역할을 합니다.
* 주로 로그인 세션 유지, 인증 처리 등에 활용됩니다.
* 예: TLS 핸드셰이크, API 세션

### 4. 전송 계층 (Transport Layer)

* 데이터 전송의 신뢰성, 순서를 보장하며, 오류 제어와 흐름 제어 기능을 제공합니다.
* TCP는 연결 기반이며 신뢰성을 보장하고, UDP는 빠르지만 비연결성입니다.
* 예: TCP, UDP

### 3. 네트워크 계층 (Network Layer)

* 목적지까지의 최적 경로를 결정합니다. (라우팅)
* IP 주소를 기준으로 패킷을 전송합니다.
* 예: IP, ICMP, 라우터

### 2. 데이터링크 계층 (Data Link Layer)

* 프레임 단위 전송, MAC 주소 기반 통신을 담당합니다.
* 에러 검출, 흐름 제어 기능이 있습니다.
* 예: Ethernet, ARP, 스위치

### 1. 물리 계층 (Physical Layer)

* 전기적 신호 전송 및 물리적 매체를 관리합니다.
* 비트를 실제 신호로 변환하여 전송합니다.
* 예: LAN 케이블, 허브, 리피터

---

## 참고 자료

* [Imperva: OSI Model](https://www.imperva.com/learn/application-security/osi-model/)
* [GeeksforGeeks: OSI Model](https://www.geeksforgeeks.org/open-systems-interconnection-model-osi/)
* [AWS: What is the OSI Model?](https://aws.amazon.com/what-is/osi-model/)
