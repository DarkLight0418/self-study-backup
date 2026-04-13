# 네트워크 손필기 전사용 노트 ver5
## Chapter 2(Application Layer) + TCP 심화 포함 통합본
부제: **개념별 정리(ver1) + 손필기 표현(ver2) + 와이어프레임 보강 + TCP 심화 유지**

---

## 이 버전의 원칙
- **개념별로 묶되**, 제목 옆에 **관련 페이지 범위**를 표시
- **손으로 옮겨 적기 쉬운 문장** 위주로 구성
- `|||` 대신 **바로 따라 그릴 수 있는 와이어프레임/ASCII 도식** 제공
- **TCP 심화는 절대 빼지 않고** 별도 큰 단원으로 유지
- 시험 문제 풀이 때 바로 써먹을 수 있도록 **비교축 / 체크 질문 / 핵심 식** 포함

---

# 0. 먼저 외울 핵심 문장 20개

1. **응용 계층 프로토콜은 메시지 타입, 문법, 의미, 송수신 규칙을 정의한다.**
2. **서로 다른 호스트의 프로세스는 메시지를 주고받으며 통신한다.**
3. **소켓은 응용 계층과 전송 계층 사이의 문이다.**
4. **프로세스 식별은 IP만으로 부족하고 IP + Port가 필요하다.**
5. **앱이 전송 계층에 요구하는 핵심은 integrity, security, timing, throughput이다.**
6. **TCP는 신뢰적·연결형이고 flow control, congestion control을 제공한다.**
7. **UDP는 비연결형이고 단순하며 빠르지만 신뢰성을 보장하지 않는다.**
8. **기본 TCP/UDP 소켓은 암호화가 없어서 TLS가 필요하다.**
9. **HTTP는 TCP를 사용하지만 기본적으로 stateless다.**
10. **Non-persistent HTTP는 객체마다 TCP 연결을 새로 만든다.**
11. **Persistent HTTP는 한 연결로 여러 객체를 보내 RTT와 오버헤드를 줄인다.**
12. **비지속 HTTP에서 객체 1개 응답 시간 감각은 2RTT + 파일 전송 시간이다.**
13. **쿠키는 stateless한 HTTP 위에서 상태를 유지하는 장치다.**
14. **웹 캐시는 가까운 곳에서 응답해 지연과 트래픽을 줄인다.**
15. **Conditional GET은 바뀌지 않은 객체를 다시 보내지 않게 한다.**
16. **SMTP는 보냄, IMAP은 가져옴이다.**
17. **DNS는 이름을 IP로 바꾸는 분산 계층형 데이터베이스다.**
18. **P2P의 핵심 장점은 참여자가 늘면 자원도 같이 늘어난다는 점이다.**
19. **DASH는 현재 대역폭에 맞춰 화질을 고르는 적응형 스트리밍이다.**
20. **Flow control은 수신자 보호, congestion control은 네트워크 보호다.**

---

# 1. 응용 계층 큰 그림 (Chapter 2 p8~p16)

## 1-1. 네트워크 앱의 본질 (p8)
- 네트워크 앱 = 서로 다른 호스트에서 실행되는 프로그램들이 **메시지** 를 주고받는 것
- 같은 호스트 내부 통신은 OS가 처리
- 다른 호스트 간 통신은 네트워크가 전달

### 손필기식 표현
- **프로그램은 말을 하고, 네트워크는 그 말을 다른 호스트로 보낸다.**
- **네트워크 코어 장비가 앱을 돌리는 게 아니라, end system에서 앱이 돈다.**

---

## 1-2. 소켓의 개념 (p9)
- 소켓 = 응용 프로세스가 메시지를 내보내고 받는 **출입문**
- 응용 개발자는 소켓을 통해 전송 계층 서비스를 사용

### 손필기식 표현
- **소켓은 문, 메시지는 문을 통과하는 짐**
- **프로세스가 말하고, 소켓이 내보내고, 전송 계층이 옮긴다**

### 와이어프레임
```text
[소켓 통신 구조]

클라이언트 프로세스
      |
    socket
      |
전송/네트워크
      |
    socket
      |
서버 프로세스
```

---

## 1-3. 프로세스 식별: IP + Port (p10)
- 메시지를 받았을 때 누구 프로세스에게 줄지 분류해야 함
- IP만으로는 부족
- 같은 호스트 안에 여러 프로세스가 동시에 존재 가능
- 식별자 = **IP 주소 + Port 번호**

### 손필기식 표현
- **집 주소(IP)만으로는 부족, 누구 방(Port)인지까지 알아야 함**
- **메시지를 받은 뒤 분류하려면 포트가 필요함**

### 예시
- HTTP 서버: 80
- 메일 서버: 25
- 형태: `10.10.16.68:80`

---

## 1-4. 응용 계층 프로토콜이 정의하는 것 (p11)
응용 계층 프로토콜은 다음 4가지를 정함.

1. 메시지 타입
2. 메시지 문법
3. 메시지 의미
4. 송수신 규칙

### 손필기식 표현
- **프로토콜은 그냥 규칙이 아니라, 메시지를 어떻게 생기게 하고 어떻게 주고받을지까지 정하는 것**

### Open vs Proprietary
- Open protocol: RFC 공개
- Proprietary protocol: 특정 기업/서비스 전용

---

## 1-5. 앱이 전송 계층에 요구하는 것 (p12~p13)
핵심 4요소:
1. **integrity**
2. **security**
3. **timing**
4. **throughput**

### 손필기식 표현
- **앱이 보는 건 네 가지: 안 깨지나 / 안전하나 / 늦지 않나 / 충분히 많이 가나**

### 감각적 분류
- 파일 전송 / 이메일 / 웹 문서
  - 유실되면 안 됨
  - TCP 쪽
- 실시간 음성 / 게임
  - 약간 손실 허용 가능
  - 지연이 더 중요
  - UDP 쪽

---

## 1-6. TCP vs UDP vs TLS (p14~p16)

### TCP
- 신뢰적 전송
- 연결형
- flow control
- congestion control
- timing guarantee 없음
- minimum throughput guarantee 없음
- 보안 자체 제공 X

### UDP
- 비연결형
- 신뢰성 X
- 순서 보장 X
- flow control X
- congestion control X
- 단순하고 빠름

### 왜 UDP가 있나?
- TCP는 정확하지만 절차가 무겁다
- 지연을 줄여야 하는 앱은 손실을 조금 감수하고 UDP를 쓸 수 있다

### TLS
- 기본 TCP/UDP 소켓은 암호화 없음
- 비밀번호도 평문으로 오갈 수 있음
- TLS는 TCP 위에 보안을 씌움
  - 암호화
  - 무결성
  - 종단 인증

### 손필기식 표현
- **TCP = 정확하게**
- **UDP = 빨리, 단순하게**
- **HTTPS = HTTP + TLS**

### 비교표
| 항목 | TCP | UDP |
|---|---|---|
| 연결 설정 | O | X |
| 신뢰성 | O | X |
| 순서 보장 | O | X |
| flow control | O | X |
| congestion control | O | X |
| 사용 감각 | 정확성 우선 | 지연/단순성 우선 |

---

# 2. HTTP (Chapter 2 p18~p47)

## 2-1. 웹 페이지와 객체 (p18~p19)
- 웹 페이지는 여러 **객체(object)** 로 구성
- 객체는 HTML, 이미지, 오디오 등
- 하나의 페이지 = base HTML + 여러 referenced objects

### 손필기식 표현
- **웹 페이지 1개를 여는 건, 실제로 여러 파일을 가져오는 것**
- HTTP = Web의 응용 계층 프로토콜
- 클라이언트 = 브라우저, 서버 = 웹 서버

---

## 2-2. HTTP의 성격: TCP 사용 + Stateless (p20)
- HTTP는 TCP 사용
- 기본 포트: 80
- 중요한 특징: **stateless**

### stateless 뜻
- 서버가 이전 요청을 기억하지 않음
- 요청이 서로 독립적

### 손필기식 표현
- **서버가 이전 요청 기억 X, 요청 독립적**
- **뒤로 가기 눌러도 재요청**
- **프로토콜은 상태 유지가 매우 어려움**

### 중요한 이유
- 장점: 단순, 확장 유리
- 단점: 로그인 / 장바구니 같은 상태 유지가 따로 필요

---

## 2-3. Non-persistent HTTP vs Persistent HTTP (p21~p25)

### Non-persistent HTTP
- 요청 시 TCP 연결 1번 열기
- 객체 1개 전송 후 연결 종료
- 객체가 많을수록 연결도 많아짐

### Persistent HTTP (HTTP/1.1)
- TCP 연결 1번 열기
- 같은 연결로 여러 객체 전송
- RTT 절약
- 오버헤드 감소

### 손필기식 표현
- **비지속 HTTP = 객체마다 새 연결**
- **지속 HTTP = 한번 열고 쫙 보냄**

### 와이어프레임
```text
[비지속 HTTP]

URL 입력
  -> TCP 연결
  -> HTML 요청/응답
  -> 연결 종료
  -> 이미지마다 다시 반복

[지속 HTTP]

TCP 연결 1번
  -> HTML
  -> image1
  -> image2
  -> image3
  -> 종료
```

### 비지속 응답 시간 감각 (p24)
- 객체 1개당 대략:
- **2RTT + 파일 전송 시간**

이유:
- 1 RTT = TCP 연결 성립
- 1 RTT = HTTP 요청 + 첫 응답 바이트
- 이후 = 파일 전송 시간

### 시험 포인트
- “왜 2RTT인가?” 설명 가능해야 함

---

## 2-4. HTTP 메시지 구조와 메서드 (p26~p29)

### 요청 메시지
- request line
- header lines
- blank line
- 필요 시 body

### 대표 메서드
- GET = 이거 줘
- POST = 데이터 담아 보낼게
- HEAD = 헤더만 줘
- PUT = 자원 갱신/업로드할게

### 응답 메시지
- status line
- header lines
- blank line
- data(body)

### 손필기식 표현
- **HTTP 요청/응답은 헤더가 중요**
- **Date 헤더는 시간 기준 맞추는 데도 의미**

### 와이어프레임
```text
[HTTP Request]

Request Line
Header 1
Header 2
Header 3

Body(필요 시)

[HTTP Response]

Status Line
Header 1
Header 2
Header 3

Data
```

---

## 2-5. HTTP 상태 코드 (p30)
- 200 OK
- 301 Moved Permanently
- 400 Bad Request
- 404 Not Found
- 505 HTTP Version Not Supported

### 손필기식 표현
- **301 = 영구 이동**
- **404 = 서버에 문서 없음**

---

## 2-6. 쿠키(Cookie) (p31~p35)
HTTP는 stateless이므로 상태 유지를 위해 쿠키를 사용

쿠키 4요소:
1. 응답 메시지의 set-cookie
2. 다음 요청 메시지의 cookie
3. 브라우저의 cookie 파일
4. 서버의 백엔드 DB

### 쿠키 용도
- 권한 유지
- 장바구니
- 유저 세션
- 추천

### 손필기식 표현
- **쿠키는 재로그인 자체가 아니라, 이미 인증된 상태를 식별하는 것**
- **쿠키는 stateless HTTP 위에 상태를 얹는 장치**

### 와이어프레임
```text
[쿠키 흐름]

첫 방문
  -> 서버가 ID 생성
  -> set-cookie 전송
  -> 브라우저 저장

다음 요청
  -> cookie: ID 포함
  -> 서버 DB 대조
  -> 사용자 식별
```

### 주의
- 편리하지만 프라이버시 문제 존재

---

## 2-7. 웹 캐시(Web Cache)와 Conditional GET (p36~p42)

### 웹 캐시
- 캐시에 객체가 있으면 바로 반환
- 없으면 origin 서버에서 받아와 저장 후 반환

### 손필기식 흐름
```text
if 캐시에 객체 존재
-> 캐시가 고객에게 객체 반환

else
-> 캐시가 origin 서버에서 받아와 저장 후 반환
```

### 왜 쓰나?
- 응답 시간 단축
- 액세스 링크 트래픽 감소
- 원 서버 부담 감소

### 손필기식 표현
- **캐시는 클라이언트 입장에선 서버, origin 입장에선 클라이언트**
- **학교/회사/ISP 쪽에 많이 둠**

### Conditional GET
- 요청: `If-Modified-Since`
- 응답:
  - 변경 없음 -> `304 Not Modified`
  - 변경됨 -> `200 OK + 새 데이터`

### 손필기식 표현
- **안 바뀐 파일은 안 보내서 시간과 트래픽 절약**

---

## 2-8. HTTP/2, HTTP/3 (p43~p47)

### HTTP/1.1 문제
- 큰 객체가 앞에 있으면 뒤의 작은 객체가 기다림
- HOL blocking

### HTTP/2
- 객체를 프레임 단위로 나눠 전송
- 전송 순서를 더 유연하게 조절
- HOL 완화

### HTTP/3
- UDP 기반 방향으로 발전
- 보안과 지연 측면 개선
- 이후 QUIC과 연결 가능

### 손필기식 표현
- **왜 하지? -> 유저 편의성**
- **프레임별로 잘라 보냄**

### 와이어프레임
```text
[HOL blocking]

HTTP/1.1
O1(큼) -> O2 -> O3 -> O4
작은 객체도 O1 뒤에서 기다림

HTTP/2
O1-frame1 / O2 / O1-frame2 / O3 / O4
작은 객체를 더 빨리 체감 가능
```

---

# 3. E-mail / SMTP / IMAP (Chapter 2 p49~p57)

## 3-1. 이메일의 3대 구성 (p49~p50)
1. User Agent
2. Mail Server
3. SMTP

### 손필기식 표현
- **사람은 UA를 쓰고, 실제 전달은 서버끼리 처리**

---

## 3-2. SMTP (p51~p56)
- 이메일은 무결성이 중요해서 TCP 사용
- 포트 25
- 3단계:
  1. handshaking
  2. message transfer
  3. closure

### 특징
- command/response
- ASCII text
- status code + phrase
- 7-bit ASCII
- persistent connection
- push 방식

### 손필기식 표현
- **SMTP = 밀어 넣는 방식(push)**

### Alice -> Bob 흐름
```text
보내는 사람
  -> 내 메일 서버
  -> 상대 메일 서버
  -> 받는 사람
```

### 메일 형식
- header
  - To
  - From
  - Subject
- blank line
- body

### 주의
- `From:` 과 `MAIL FROM:` 은 다름

---

## 3-3. IMAP / HTTP (p57)
- SMTP = 보냄
- IMAP = 서버에 저장된 메일 가져오기 / 관리
- HTTP = 웹메일 인터페이스

### 손필기식 표현
- **SMTP = 송신**
- **IMAP = 수신/조회/관리**

### 비교표
| 항목 | SMTP | IMAP |
|---|---|---|
| 역할 | 메일 보냄 | 메일 가져옴/관리 |
| 방향 | 송신 중심 | 수신/접근 중심 |
| 핵심 감각 | push | 서버 저장 메일 접근 |

---

# 4. DNS (Chapter 2 p59~p72)

## 4-1. DNS란? (p59)
- 사람은 이름 사용
- 네트워크는 IP 주소 사용
- 이름 <-> IP 변환 필요

### 정의
- **분산된 계층형 데이터베이스**
- 동시에 **응용 계층 프로토콜**

### 손필기식 표현
- **DNS는 단순 전화번호부가 아니라, 계층형 분산 전화번호 시스템**

---

## 4-2. DNS를 중앙집중형으로 못 하는 이유 (p60)
- SPOF
- 트래픽 폭증
- 멀리 있는 중앙 DB
- 유지보수 어려움
- 확장성 부족

### DNS 서비스
- hostname -> IP
- host aliasing
- mail server aliasing
- load distribution

### 손필기식 표현
- **질의가 너무 많아서 중앙집중형은 스케일이 안 됨**

---

## 4-3. DNS 계층 구조 (p61~p64)
1. Root DNS
2. TLD DNS
3. Authoritative DNS
4. Local DNS

### Local DNS
- 학교 / 회사 / ISP의 기본 DNS
- 실제 사용 시작점

### 와이어프레임
```text
[DNS 계층 구조]

Local DNS
   |
 Root
   |
 TLD(.com, .org, .edu, .kr ...)
   |
 Authoritative DNS
   |
 실제 host 정보
```

---

## 4-4. Iterative vs Recursive Query (p65~p66)

### Iterative
- “나는 모르니 저 서버에 물어봐”

### Recursive
- “내가 대신 끝까지 찾아줄게”

### 손필기식 표현
- **Iterative = 쟤한테 물어봐**
- **Recursive = 내가 대신 찾아줄게**

### 비교표
| 항목 | Iterative | Recursive |
|---|---|---|
| 느낌 | 저 서버에 물어봐 | 내가 대신 찾아줄게 |
| 부담 | 요청자 쪽 | 중간 서버 쪽 |

---

## 4-5. 캐싱, TTL, 레코드 (p67~p68)
- DNS 결과는 캐시됨
- TTL 지나면 만료
- IP 변경되어도 TTL 전까지 구정보가 남을 수 있음

### 손필기식 표현
- **TTL = 살아 있는 시간, 끝나면 다시 질의**

### 레코드
- A = hostname -> IP
- NS = authoritative name server
- CNAME = alias -> canonical name
- MX = mail server

### 손필기식 표현
- **A는 주소, MX는 메일, CNAME은 별칭, NS는 권한 서버**

---

## 4-6. 메시지 / 등록 / 보안 (p69~p72)
- query / reply 구조
- question / answer / authority / additional
- registrar가 도메인 등록 처리
- 보안 이슈:
  - DDoS
  - redirect attack
  - DNS poisoning
- 대응:
  - DNSSEC

---

# 5. P2P / BitTorrent (Chapter 2 p74~p82)

## 5-1. P2P 구조 (p74)
- 항상 켜져 있는 중앙 서버 없음
- peer들이 직접 통신
- 요청도 하고 서비스도 제공

### 장점
- 참여자 증가 = 자원 증가
- self-scalability

### 단점
- IP 변경
- churn
- 관리 복잡

### 손필기식 표현
- **요청만 하는 게 아니라, 나도 남에게 제공한다**

---

## 5-2. 파일 배포 시간 관점 (p75~p78)
### Client-Server
- 서버 혼자 N명에게 보내야 함
- N이 커질수록 부담 증가

### P2P
- 각 peer도 업로드 참여
- 사용자 증가가 자원 증가로도 이어짐

### 손필기식 표현
- **사람이 늘면 일도 늘지만, 도와줄 손도 늘어난다**

---

## 5-3. BitTorrent (p79~p82)
- 파일을 chunk로 분할
- tracker가 peer 목록 관리
- peer끼리 chunk 교환

### 요청 전략
- rarest first

### 전송 전략
- tit-for-tat
- optimistic unchoke

### 손필기식 표현
- **희귀한 조각부터 받아야 전체가 골고루 퍼짐**
- **서로 잘 주는 상대와 우선 교환**

### 와이어프레임
```text
[BitTorrent]

파일 -> chunk1, chunk2, chunk3, chunk4

tracker
  -> peer 목록 제공

peer A <-> peer B
peer A <-> peer C
peer B <-> peer C
```

---

# 6. Streaming / DASH / CDN (Chapter 2 p84~p98)

## 6-1. 스트리밍이 어려운 이유 (p84~p90)
- 비디오 트래픽 큼
- 사용자 많음
- 대역폭 변화
- jitter
- 손실
- 재생은 끊기면 안 됨

### 해결
- client-side buffering

### 손필기식 표현
- **미리 좀 받아놔야 네트워크가 흔들려도 영상이 안 끊김**

---

## 6-2. 비디오 압축 개념 (p85~p86)
- spatial coding
- temporal coding
- CBR / VBR

### 손필기식 표현
- **매번 전체를 다 보내지 않고, 겹치는 부분은 줄여서 보냄**

---

## 6-3. DASH (p91~p92)
- 서버:
  - 비디오를 chunk로 나눔
  - 여러 bitrate 버전 저장
  - manifest 제공
- 클라이언트:
  - 현재 대역폭 측정
  - 적절한 화질 chunk 선택

### 손필기식 표현
- **좋은 망이면 고화질, 나쁜 망이면 저화질**
- **상황 따라 화질 바꾸는 적응형 스트리밍**

### 와이어프레임
```text
[DASH]

server
  -> 240p chunk
  -> 480p chunk
  -> 720p chunk
  -> manifest

client
  -> 현재 대역폭 측정
  -> 적절한 화질 선택
```

---

## 6-4. CDN (p93~p98)
- 여러 지역에 콘텐츠 복사본 저장
- 사용자 가까운 서버가 전송

### 단일 mega-server 문제
- 병목
- 먼 거리
- 단일 장애점
- 확장성 부족

### 손필기식 표현
- **콘텐츠를 사용자 가까이에 미리 놔두는 구조**

### 와이어프레임
```text
[CDN]

사용자
  -> 가까운 CDN 노드 선택
  -> 해당 노드에서 콘텐츠 수신

CDN 노드 A
CDN 노드 B
CDN 노드 C
```

---

# 7. Socket Programming (Chapter 2 p100~p109)

## 7-1. 두 가지 소켓 (p100~p101)
- UDP socket = unreliable datagram
- TCP socket = reliable byte stream

### 손필기식 표현
- **UDP = 편지 묶음**
- **TCP = 연결된 파이프**

---

## 7-2. UDP 소켓 (p102~p105)
### 특징
- 연결 설정 없음
- handshake 없음
- 매번 목적지 IP/Port 명시
- 순서 보장 X
- 손실 가능

### 기억 포인트
- sendto()
- recvfrom()

### 와이어프레임
```text
[UDP client/server]

client socket 생성
   -> sendto()
server socket
   -> recvfrom()
   -> sendto()
client
   -> recvfrom()
```

---

## 7-3. TCP 소켓 (p106~p109)
### 특징
- 먼저 연결 설정 필요
- 서버는 welcoming socket 준비
- 접속 오면 client 전용 새 socket 생성
- 신뢰적, 순서 보장된 byte stream

### 기억 포인트
- 서버: bind -> listen -> accept
- 클라이언트: connect
- 이후 send / recv / close

### 손필기식 표현
- **서버는 손님 기다리는 문 하나와, 손님별 대화방 소켓을 따로 둔다**

### 와이어프레임
```text
[TCP client/server]

server:
  socket()
  bind()
  listen()
  accept()

client:
  socket()
  connect()

이후:
  send / recv / close
```

---

# 8. TCP 심화 (Chapter 3 + 직접 손필기 보강)

## 8-1. 왜 TCP 심화가 중요한가?
- Chapter 2에서 TCP를 “쓴다”까지 배웠다면
- 여기서는 TCP가 **어떻게 신뢰성/흐름제어/혼잡제어를 구현하는지** 본다
- 시험에서는 HTTP, SMTP가 TCP를 쓴다는 말만 아는 것보다
- **RTT, ACK, rwnd, cwnd, handshake, AIMD**를 연결해서 말할 수 있어야 함

---

## 8-2. RTT, EstimatedRTT, Timeout (Chap3 p81~p83)
### RTT
- 세그먼트가 갔다가 ACK가 돌아오는 왕복 시간
- 고정값이 아니라 상황에 따라 변함

### 왜 EstimatedRTT가 필요한가?
- SampleRTT는 튈 수 있음
- 그래서 부드러운 추정치 필요

### 식
- `EstimatedRTT = (1 - a) * EstimatedRTT + a * SampleRTT`
- 보통 `a = 0.125`

### DevRTT
- RTT 변동 폭 반영
- `DevRTT = (1 - b) * DevRTT + b * |SampleRTT - EstimatedRTT|`
- 보통 `b = 0.25`

### Timeout
- `TimeoutInterval = EstimatedRTT + 4 * DevRTT`

### 손필기식 표현
- **RTT 평균만 보는 게 아니라, 얼마나 흔들리는지도 본다**
- **변동 폭까지 보고 안전 마진을 둔다**

### 시험 포인트
- timeout이 너무 짧으면 불필요한 재전송
- 너무 길면 손실 대응 느림

---

## 8-3. ACK, delayed ACK, duplicate ACK, fast retransmit (Chap3 p84~p88)
### TCP sender 기본
- 미확인 세그먼트 기준으로 동작
- 타이머는 보통 가장 오래된 unACKed segment 기준

### ACK generation
- in-order 도착 -> ACK
- delayed ACK 가능
- out-of-order 도착 -> duplicate ACK

### fast retransmit
- 같은 ACK가 3번 더 반복되면
- 중간 세그먼트 손실 가능성이 높다고 판단
- timeout까지 안 기다리고 재전송

### 손필기식 표현
- **같은 ACK가 반복되면 중간에 하나 빠졌다고 의심**
- **느리게 timeout 기다리지 말고 빨리 재전송**

### 와이어프레임
```text
[Fast Retransmit 감각]

보낸 순서:
100 / 120 / 140 / 160

수신 측 입장:
100 다음 120은 기대 중인데 140 도착
-> ACK 120
160 도착
-> ACK 120
또 도착
-> ACK 120

송신 측:
중복 ACK 3개 감지
-> 120부터 빨리 재전송
```

---

## 8-4. Flow Control = 수신자 보호 (Chap3 p90~p95)
### 왜 필요한가?
- 네트워크가 너무 빨리 보내면 수신 버퍼가 넘칠 수 있음

### 핵심 변수
- **rwnd (receive window)**

### 의미
- 수신자가 아직 받아들일 수 있는 여유 공간
- 송신자는 이 값을 넘지 않게 전송량 조절

### 손필기식 표현
- **리시버가 감당 가능한 속도로 보내게 하는 것**
- **Flow control = receiver가 sender를 조절**
- **수신 버퍼 절대 overflow 안 나게 하려는 설계**

### 와이어프레임
```text
[Flow Control]

Receiver buffer
+-------------------------+
| 이미 받은 데이터        |
|-------------------------|
| 남은 공간 = rwnd        |
+-------------------------+

sender:
  in-flight 데이터 양 <= rwnd
```

---

## 8-5. Connection Management: 왜 3-way handshake인가? (Chap3 p96~p103)
### 왜 handshake가 필요하나?
- 그냥 보내면 서로 상태 확인이 안 됨
- 오래된 패킷
- 재전송된 연결 요청
- half-open connection
- duplicate data 문제 가능

### 2-way handshake 문제
- 한쪽만 살아 있고 다른 쪽은 이미 종료됐을 수 있음
- half-open connection 발생 가능

### 3-way handshake
1. SYN
2. SYN + ACK
3. ACK

### 의미
- “연결할래?”
- “좋아, 나도 준비됨”
- “확인, 시작하자”

### 손필기식 표현
- **3-way는 메시지 3번이 중요한 게 아니라, 양방향 상태 확인이 중요**
- **요청 - 응답 - 확인**

### 종료
- FIN / ACK 기반
- 양방향 종료는 독립적

### 와이어프레임
```text
[3-way handshake]

Client                Server
  | ---- SYN -------> |
  | <--- SYN+ACK ---- |
  | ---- ACK -------> |

=> 연결 성립
```

---

## 8-6. Congestion Control = 네트워크 보호 (Chap3 p104~p117)
### Flow control과 다름
- Flow control = 수신자가 못 받는 문제
- Congestion control = 네트워크 자체가 감당 못하는 문제

### 혼잡이 생기면
- 라우터 큐 증가
- 지연 증가
- 패킷 손실
- 재전송 증가
- 불필요한 duplicate까지 생김
- 성능 더 악화

### 손필기식 표현
- **느리다 = 그냥 느린 게 아니라, 네트워크가 감당 못하는 상태일 수 있음**
- **라우터 큐 길어짐 -> 지연 증가 -> 드랍 -> 재전송 -> 더 혼잡**

### 중요한 관점
- sender는 라우터 내부 상태를 직접 모름
- 결과(손실, 지연) 보고 **추정** 해야 함

### end-to-end vs network-assisted
- end-to-end: TCP 기본 방식
- network-assisted: ECN 같은 직접 피드백 방식

---

## 8-7. AIMD, cwnd, Slow Start, ssthresh (Chap3 p118~p126)
### cwnd
- congestion window
- 송신자가 동시에 네트워크에 띄워둘 수 있는 양

### AIMD
- Additive Increase
  - 천천히 증가
- Multiplicative Decrease
  - 손실 나면 크게 감소

### 손필기식 표현
- **조금씩 늘리고, 문제 생기면 확 줄인다**
- **네트워크 용량을 조심스럽게 탐색하는 방식**

### Slow Start
- 초기 `cwnd = 1 MSS`
- RTT마다 대략 2배 증가
- 빠르게 사용 가능한 용량 탐색

### ssthresh
- 손실 발생 전 cwnd의 절반 정도를 기준으로 전환
- 그 이후에는 더 천천히 증가

### Reno / Tahoe 감각
- Tahoe: timeout 나면 크게 줄이고 처음부터 다시
- Reno: triple duplicate ACK면 덜 극단적으로 회복

### CUBIC 감각
- 더 나은 throughput을 위해 현대적으로 개선된 방식
- Linux 계열에서 많이 사용

### 와이어프레임
```text
[AIMD 톱니 모양]

cwnd
 ^
 |        / |       /   |      /     |     /       |    /         |   /           |  /             | /              \__
 +----------------------> time
```

```text
[Slow Start]

1 MSS
 -> 2 MSS
 -> 4 MSS
 -> 8 MSS
 -> ...
(loss or ssthresh 도달 전까지 빠르게 증가)
```

---

## 8-8. Flow Control vs Congestion Control (핵심 비교)
| 항목 | Flow Control | Congestion Control |
|---|---|---|
| 보호 대상 | 수신자 | 네트워크 |
| 원인 | receiver buffer 부족 | 라우터/링크 혼잡 |
| 핵심 변수 | rwnd | cwnd |
| 핵심 질문 | 상대가 더 받을 수 있나? | 망이 더 버틸 수 있나? |

### 손필기식 표현
- **Flow control은 receiver 문제**
- **Congestion control은 network 문제**

---

## 8-9. TCP 문제풀이 때 바로 떠올릴 질문
1. timeout 식을 쓸 때 EstimatedRTT와 DevRTT를 구분했는가?
2. duplicate ACK가 왜 발생하는지 설명 가능한가?
3. fast retransmit이 timeout보다 왜 빠른가?
4. rwnd와 cwnd를 헷갈리지 않았는가?
5. 3-way handshake가 왜 2-way보다 안전한가?
6. 혼잡이 왜 재전송 때문에 더 악화될 수 있는가?
7. AIMD와 slow start 차이를 말할 수 있는가?

---

# 9. 헷갈리는 비교 한 번에 정리

## 9-1. TCP vs UDP
| 항목 | TCP | UDP |
|---|---|---|
| 연결 설정 | O | X |
| 신뢰성 | O | X |
| 순서 보장 | O | X |
| flow control | O | X |
| congestion control | O | X |
| 사용 감각 | 정확성 우선 | 지연/단순성 우선 |

## 9-2. Non-persistent vs Persistent HTTP
| 항목 | Non-persistent | Persistent |
|---|---|---|
| 연결 수 | 객체마다 별도 | 하나로 여러 객체 |
| RTT 부담 | 큼 | 감소 |
| 효율 | 낮음 | 높음 |

## 9-3. HTTP vs SMTP
| 항목 | HTTP | SMTP |
|---|---|---|
| 기본 동작 | pull | push |
| 역할 | 웹 객체 요청/응답 | 메일 전송 |
| 대표 특징 | stateless | 서버 간 전달 |

## 9-4. SMTP vs IMAP
| 항목 | SMTP | IMAP |
|---|---|---|
| 역할 | 메일 보냄 | 메일 가져옴/관리 |
| 방향 | 송신 중심 | 수신/조회 중심 |
| 핵심 감각 | push | 서버 저장 메일 접근 |

## 9-5. Iterative vs Recursive DNS Query
| 항목 | Iterative | Recursive |
|---|---|---|
| 느낌 | 저 서버에 물어봐 | 내가 대신 찾아줄게 |
| 부담 | 요청자 쪽 | 중간 서버 쪽 |

## 9-6. Web Cache vs Cookie
| 항목 | Web Cache | Cookie |
|---|---|---|
| 목적 | 응답 속도/트래픽 최적화 | 상태 유지 |
| 저장 성격 | 자주 쓰는 객체 | 사용자 식별 정보 |
| 핵심 질문 | 더 빨리 줄 수 있나? | 누구인지 기억할 수 있나? |

---

# 10. 손필기 최종 압축본

## 응용 계층 기본
- 프로세스끼리 메시지 교환
- 소켓 = 응용과 전송 계층 사이 문
- 식별 = IP + Port
- 프로토콜 = 타입 / 문법 / 의미 / 규칙

## 전송 서비스
- integrity / security / timing / throughput
- TCP = 신뢰적, 연결형
- UDP = 단순, 빠름, 비연결형
- TLS = TCP 위 보안

## HTTP
- TCP 사용, 포트 80
- stateless
- Non-persistent = 객체마다 연결
- Persistent = 한 연결로 여러 객체
- 쿠키 = 상태 유지
- 캐시 = 가까운 곳에서 응답
- Conditional GET = 안 바뀌면 다시 안 보냄
- HTTP/2 = 프레임으로 HOL 완화
- HTTP/3 = UDP 기반 발전형

## E-mail
- UA / Mail Server / SMTP
- SMTP = 보냄, TCP 25, push
- IMAP = 가져옴

## DNS
- 이름 <-> IP 변환
- 분산 계층형 DB
- Root / TLD / Authoritative / Local
- A / NS / CNAME / MX
- TTL / caching

## P2P
- 중앙 서버 없이 peer끼리 교환
- 참여자 증가 = 자원 증가
- BitTorrent = chunk / rarest first / tit-for-tat

## Streaming
- buffer로 지터 완충
- DASH = 상황 따라 화질 선택
- CDN = 가까운 서버에서 전송

## Socket
- UDP = sendto / recvfrom / datagram
- TCP = connect / accept / byte stream

## TCP 심화
- RTT는 변함 -> EstimatedRTT 사용
- Timeout = EstimatedRTT + 4DevRTT
- duplicate ACK / fast retransmit 중요
- flow control = rwnd, 수신자 보호
- 3-way handshake = 양방향 상태 확인
- congestion control = 네트워크 보호
- AIMD = 조금씩 증가, 크게 감소
- slow start = 처음엔 빠르게 탐색
- cwnd / ssthresh 구분 중요

---

# 11. 마지막 체크리스트
- 왜 IP만으로는 프로세스를 식별할 수 없는가?
- 앱이 요구하는 전송 서비스 4가지는 무엇인가?
- 왜 HTTP는 stateless인데 쿠키가 필요한가?
- Non-persistent HTTP가 왜 비효율적인가?
- Conditional GET의 목적은 무엇인가?
- SMTP와 IMAP의 차이는?
- DNS를 중앙집중형으로 만들면 왜 안 되는가?
- Iterative query와 Recursive query 차이는?
- P2P가 client-server보다 유리한 이유는?
- DASH가 adaptive streaming이라 불리는 이유는?
- UDP 소켓과 TCP 소켓의 차이는?
- EstimatedRTT / DevRTT / Timeout 식을 설명할 수 있는가?
- fast retransmit이 왜 timeout보다 빠른가?
- rwnd와 cwnd를 구분할 수 있는가?
- 3-way handshake가 왜 필요한가?
- AIMD와 slow start 차이는 무엇인가?