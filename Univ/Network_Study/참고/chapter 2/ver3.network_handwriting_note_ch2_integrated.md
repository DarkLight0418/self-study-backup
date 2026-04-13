# 네트워크 손필기 전사용 통합 노트
## Chapter 2(Application Layer) 통합판
부제: **페이지별 흐름 + 문장형 정리 + 손필기 표현 + 시험 포인트**를 한 번에 묶은 전사용 버전

---

## 이 노트의 쓰임
- **교안 순서가 끊기지 않게** p8 ~ p109 흐름을 따라감
- 기존 문장형 정리의 **큰 개념 축**을 유지함
- 페이지별 정리본의 **손필기 표현, 수업식 강조 포인트**를 반영함
- 시험에서 바로 헷갈리는 부분은 **비교축**으로 묶음
- `|||` 는 **직접 그려 넣을 그림 자리**
- 문장은 **짧게**, 정의보다 **왜 필요한지 / 무엇과 다른지** 중심으로 적음

---

# 0. 시험 전에 먼저 외울 핵심 18문장

1. **응용 계층 프로토콜은 메시지 타입, 문법, 의미, 송수신 규칙을 정의한다.**
2. **서로 다른 호스트의 프로세스는 메시지를 주고받으며 통신한다.**
3. **소켓은 응용 계층과 전송 계층 사이의 문이다.**
4. **프로세스 식별은 IP만으로 부족하고 IP + Port가 필요하다.**
5. **앱이 전송 계층에 요구하는 핵심은 integrity, security, timing, throughput이다.**
6. **TCP는 신뢰적이고 연결형이며 flow control, congestion control을 제공한다.**
7. **UDP는 비연결형이고 단순하며 빠르지만 신뢰성을 보장하지 않는다.**
8. **기본 TCP/UDP 소켓은 평문이므로 TLS가 필요하다.**
9. **HTTP는 TCP를 사용하지만 기본적으로 stateless다.**
10. **Non-persistent HTTP는 객체마다 TCP 연결을 새로 만든다.**
11. **Persistent HTTP는 한 연결로 여러 객체를 보내 RTT와 오버헤드를 줄인다.**
12. **쿠키는 stateless한 HTTP 위에서 상태를 유지하기 위한 장치다.**
13. **웹 캐시는 가까운 곳에서 응답하여 지연과 트래픽을 줄인다.**
14. **Conditional GET은 바뀌지 않은 객체를 다시 보내지 않게 한다.**
15. **SMTP는 메일을 보내는 프로토콜이고 IMAP은 서버에 저장된 메일을 가져오는 프로토콜이다.**
16. **DNS는 이름을 IP로 바꾸는 분산 계층형 데이터베이스다.**
17. **P2P의 핵심 장점은 참여자가 늘면 자원도 같이 늘어난다는 점이다.**
18. **DASH는 현재 대역폭에 맞춰 화질을 바꾸는 적응형 스트리밍이다.**

---

# 1. Application Layer 큰 그림 (p8 ~ p16)

## p8 프로세스끼리 메시지로 통신
- 네트워크 앱 = 서로 다른 종단 시스템에서 실행되는 프로그램들이 **메시지** 를 주고받는 것
- 같은 호스트 내부 통신은 OS가 처리
- 다른 호스트 간 통신은 네트워크를 통해 메시지를 전달

### 손필기 표현
- **프로그램끼리 직접 말하는 게 아니라, 메시지를 네트워크에 실어 보내는 것**
- **호스트 안에서는 OS가, 호스트 밖에서는 네트워크가 중간 전달자 역할**

### 시험 포인트
- 네트워크 앱은 **네트워크 코어 장비가 아니라 end system에서 실행**
- 빠른 앱 개발/배포가 가능한 이유도 end system에서 돌아가기 때문

---

## p9 소켓의 개념
- 소켓 = 응용 프로세스가 메시지를 내보내고 받는 **출입문**
- 보내는 쪽 프로세스가 소켓에 메시지를 밀어 넣으면
- 전송 계층/OS가 반대편 소켓까지 전달을 맡음

### 손필기 표현
- **소켓은 문, 메시지는 문을 통과하는 짐**
- **프로세스가 말하고, 소켓이 내보내고, 전송 계층이 운반한다**

||| 소켓 통신 그림
`프로세스 -> 소켓 -> 전송/네트워크 -> 소켓 -> 프로세스`

### 시험 포인트
- 응용 개발자가 직접 제어하는 부분과 OS가 제어하는 부분의 경계
- 소켓은 응용 계층과 전송 계층 사이 인터페이스라는 감각

---

## p10 주소와 프로세스 식별
- 메시지를 받아도 **누구 프로세스에게 줄지** 알아야 함
- IP 주소만으로는 부족
- 같은 호스트 안에 여러 프로세스가 동시에 실행 가능하기 때문
- 식별자 = **IP 주소 + Port 번호**

### 손필기 표현
- **집 주소(IP)만으로는 부족, 누구 방(Port)인지까지 알아야 함**
- **메시지를 받은 뒤 분류하려면 포트가 필요함**

### 예시
- HTTP 서버: 80
- Mail 서버: 25
- 형태: `10.10.16.68:80`

### 시험 포인트
- 포트 번호는 **같은 IP 안에서 프로세스를 구분**하기 위한 것
- 포트 번호 자체는 다른 호스트에서 같아도 됨

---

## p11 응용 계층 프로토콜이 정의하는 것
응용 계층 프로토콜은 아래 4가지를 정함.

1. **메시지 타입**
   - request / response
2. **메시지 문법(syntax)**
   - 필드가 어떻게 생겼는가
3. **메시지 의미(semantics)**
   - 필드가 무엇을 뜻하는가
4. **송수신 규칙**
   - 언제 보내고 어떻게 응답하는가

### 손필기 표현
- **프로토콜은 그냥 규칙이 아니라, 메시지를 어떻게 생기게 하고 어떻게 주고받을지까지 정하는 것**

### Open vs Proprietary
- Open protocol: RFC 공개, 누구나 구현 가능
- Proprietary protocol: 특정 기업/서비스 전용

### 시험 포인트
- HTTP, SMTP 같은 대표 프로토콜을 예시로 바로 연결할 수 있어야 함

---

## p12 앱이 필요로 하는 전송 서비스 4요소
1. **data integrity**
2. **security**
3. **timing**
4. **throughput**

### 손필기 표현
- **앱이 보는 건 결국 네 가지: 안 깨지나 / 안전하나 / 늦지 않나 / 충분히 많이 가나**

### 시험 포인트
- 파일 전송: integrity 중요
- 로그인/결제: security 중요
- 통화/게임: timing 중요
- 영상: throughput 중요

---

## p13 일반 앱의 요구사항
### TCP 쪽 감각
- 파일 업/다운로드
- 이메일
- 웹 문서
- 텍스트 메시지

공통점:
- **유실되면 곤란**
- 실시간성은 상대적으로 덜 중요

### UDP 쪽 감각
- 실시간 오디오/비디오
- 인터랙티브 게임

공통점:
- **약간의 손실은 감수 가능**
- 대신 지연이 더 중요

### 시험 포인트
- 문제에서 “조금 손실돼도 괜찮고 지연이 중요”라고 하면 UDP 후보
- “절대 안 깨져야 한다”면 TCP 후보

---

## p14 ~ p16 TCP / UDP / TLS

## TCP
- reliable transport
- connection-oriented
- flow control
- congestion control
- 제공하지 않는 것:
  - timing guarantee
  - minimum throughput guarantee
  - security

## UDP
- connectionless
- unreliable
- 순서 보장 없음
- flow control 없음
- congestion control 없음
- throughput guarantee 없음
- connection setup 없음

### 왜 UDP가 존재하나?
- TCP는 절차가 많고 무거움
- 모든 상황에서 “정확성 우선”이면 비효율적일 수 있음
- 손실을 조금 허용하고 빠르게 보내야 하는 앱도 있음

### 손필기 표현
- **TCP = 정확하게, 절차 있게**
- **UDP = 빨리, 단순하게**

## TLS
- 기본 TCP/UDP 소켓은 암호화 없음
- 비밀번호도 평문으로 갈 수 있음
- TLS는 TCP 위에 보안을 추가
  - 암호화
  - 데이터 무결성
  - 종단 인증

### 손필기 표현
- **HTTPS = HTTP + TLS**
- **TCP 자체가 안전한 게 아니라, 위에서 TLS를 씌우는 것**

### 시험 포인트
- TCP가 reliable하다고 해서 secure한 것은 아님
- security는 TLS 같은 별도 계층적 장치가 필요

---

# 2. Web & HTTP (p18 ~ p47)

## p18 웹 페이지와 객체
- 웹 페이지는 여러 **객체(object)** 로 구성
- 객체는 HTML, JPEG, 오디오 등 가능
- 하나의 페이지 = base HTML + 여러 참조 객체
- 각 객체는 URL로 지정 가능

### 손필기 표현
- **웹 페이지 하나를 여는 건, 실제로는 여러 파일을 가져오는 것**

### 시험 포인트
- base HTML과 referenced objects 구분
- 왜 객체 개수가 많아지면 HTTP 성능이 중요해지는지 연결

---

## p19 HTTP 개요
- HTTP = Web의 응용 계층 프로토콜
- client/server 모델
  - client = browser
  - server = web server
- 브라우저가 요청하고 서버가 객체를 응답

### 시험 포인트
- HTTP는 application layer protocol
- transport는 TCP 사용

---

## p20 HTTP의 성격: TCP + Stateless
- HTTP는 TCP 사용
- 기본 포트: 80
- HTTP 메시지 교환 후 연결 종료 가능
- 중요한 특징: **stateless**

### stateless 뜻
- 서버가 이전 요청을 기억하지 않음
- 모든 요청이 독립적임

### 손필기 표현
- **뒤로 가기 눌렀다고 예전 상태를 기억하는 게 아니라, 다시 요청해서 가져오는 구조**
- **프로토콜이 상태를 유지하는 건 생각보다 매우 어렵다**

### 시험 포인트
- stateless의 장점: 단순함, 확장성
- stateless의 단점: 로그인/장바구니 같은 상태 유지가 어려움

---

## p21 비연결형 vs 연결형 HTTP

## Non-persistent HTTP
- TCP 연결 열기
- 객체 1개 보내기
- TCP 연결 닫기

## Persistent HTTP
- TCP 연결 1번 열기
- 같은 연결로 여러 객체 전송
- RTT와 오버헤드 감소

### 손필기 표현
- **비연결형 = 파일 하나 받을 때마다 다시 연결**
- **연결형 = 한번 열고 쭉 보냄**

### 시험 포인트
- 둘의 차이를 “연결 수”와 “RTT 낭비”로 설명할 수 있어야 함

---

## p22 ~ p24 Non-persistent HTTP 예시와 응답시간
- HTML 1개 받기 위해 1번 연결
- 이미지 10개면 추가로 10번 연결
- 연결 생성/종료가 반복됨

### 손필기 표현
- **HTML 1개 + 이미지 10개면 연결이 11번까지 늘어날 수 있음**
- **객체마다 새 연결이라 비효율**

### 응답 시간
- 객체 1개당 대략
- **2 RTT + 파일 전송 시간**

이유:
- 1 RTT: TCP 연결 설정
- 1 RTT: HTTP 요청 + 첫 응답 바이트
- 이후: 파일 전송

||| Non-persistent HTTP 흐름 그림
- URL 입력
- TCP 연결
- HTML 요청/응답
- 종료
- 이미지마다 반복

### 시험 포인트
- “왜 2RTT인가?”를 말로 설명할 수 있어야 함

---

## p25 Persistent HTTP (HTTP/1.1)
- 서버가 연결을 열어 둠
- 같은 클라이언트/서버 사이에서 여러 객체 전송 가능
- 객체마다 다시 연결하지 않으므로 효율 상승

### 손필기 표현
- **한번 열고 필요한 것들을 이어서 받는 형태**

### 시험 포인트
- Non-persistent의 문제점: 2RTT/객체 + OS 오버헤드
- Persistent의 장점: 같은 연결 재사용

---

## p26 ~ p29 HTTP 메시지

## 요청 메시지
- request line
- header lines
- blank line
- 필요 시 body

대표 method:
- **GET** = 이거 줘
- **POST** = 이거 담아서 보낼게
- **HEAD** = 헤더만 보여줘
- **PUT** = 이 자원 갱신/업로드할게

## 응답 메시지
- status line
- header lines
- blank line
- data(body)

### 손필기 표현
- **HTTP 요청/응답은 헤더가 매우 중요**
- **Date 헤더는 시간 기준을 맞추는 데도 의미**

||| HTTP general format 그림
- request: request line / header / blank line / body
- response: status line / header / blank line / data

### 시험 포인트
- request와 response의 큰 구조를 구분할 수 있어야 함
- POST는 body, GET은 URL 뒤에 데이터가 붙을 수 있음

---

## p30 HTTP 상태 코드
- **200 OK**
- **301 Moved Permanently**
- **400 Bad Request**
- **404 Not Found**
- **505 HTTP Version Not Supported**

### 손필기 표현
- 301 = 영구 이동, 리다이렉트
- 404 = 서버에 문서 없음

### 시험 포인트
- 200 / 301 / 404 정도는 의미를 바로 떠올릴 수 있어야 함

---

## p31 ~ p35 쿠키(Cookies)
### 왜 필요한가?
- HTTP는 stateless
- 그런데 로그인, 장바구니, 세션 유지에는 **상태 정보** 가 필요

### 쿠키 구성 4요소
1. 응답 메시지의 set-cookie
2. 다음 요청 메시지의 cookie
3. 브라우저의 cookie 파일
4. 서버의 백엔드 DB

### 저장 위치
- 사용자의 브라우저 쪽

### 용도
- 권한 유지
- 장바구니
- 추천
- 세션 유지

### 손필기 표현
- **쿠키는 재로그인 자체가 아니라, 이미 인증된 상태를 식별하게 해주는 것**
- **stateless HTTP 위에 상태를 얹는 장치**

### 프라이버시 메모
- 추적 쿠키 문제 존재
- 편리하지만 사생활 측면 주의

||| 쿠키 흐름 그림
- 첫 방문 -> 서버가 ID 생성 -> 브라우저 저장
- 다음 요청 -> cookie ID 포함 -> 서버 DB와 대조

### 시험 포인트
- 왜 쿠키가 필요한지: HTTP가 stateless이기 때문
- 쿠키를 상태 유지 장치로 설명 가능해야 함

---

## p36 ~ p42 웹 캐시와 Conditional GET

## p36 웹 캐시(Web Cache, Proxy)
- 브라우저가 캐시에게 요청
- 캐시에 객체가 있으면 바로 반환
- 없으면 캐시가 origin 서버에서 받아와 저장 후 반환

### 손필기식 흐름
```text
if 캐시에 객체 존재
-> 캐시가 고객에게 객체 반환
else
-> 캐시가 origin 서버에서 받아와 저장 후 반환
```

### 왜 쓰는가?
- 응답 시간 단축
- 액세스 링크 트래픽 감소
- 원 서버 부담 감소

### 손필기 표현
- **캐시는 클라이언트 입장에선 서버, origin 입장에선 클라이언트**
- **학교/회사/ISP 쪽에 설치되는 경우 많음**

### 시험 포인트
- proxy cache가 양쪽 역할을 동시에 한다는 점
- 가까운 곳에서 응답하므로 지연 감소

---

## p42 Conditional GET
- 목표: 캐시에 최신 버전이 있으면 **객체를 다시 안 보내기**
- 요청 헤더: `If-Modified-Since`
- 응답:
  - 안 바뀜 -> `304 Not Modified`
  - 바뀜 -> `200 OK` + 데이터

### 손필기 표현
- **안 바뀐 파일은 다시 안 보내서 시간과 트래픽을 아낌**

### 시험 포인트
- 304 Not Modified 의미
- 캐시 효율을 높이는 방법으로 Conditional GET 설명 가능해야 함

---

## p43 ~ p47 HTTP/2, HTTP/3

## HTTP/1.1 문제
- 요청 순서대로 처리하는 성격(FCFS)
- 앞의 큰 객체 때문에 뒤의 작은 객체가 기다릴 수 있음
- 이것이 **HOL blocking**

## HTTP/2
- 객체를 **프레임(frame)** 단위로 나눔
- 전송 순서를 더 유연하게 조절
- 작은 객체가 큰 객체 뒤에 오래 묶이지 않도록 HOL 완화

### 손필기 표현
- **왜 하지? -> 유저 편의성**
- **프레임별로 잘라서 보냄**

## HTTP/3
- UDP 기반 방향으로 발전
- 보안과 지연 측면 개선
- 이후 QUIC과 연결되는 흐름

||| HOL blocking 완화 그림
- HTTP/1.1: O1 큰 객체 뒤에 O2/O3/O4 대기
- HTTP/2: 프레임을 섞어 O2/O3/O4 체감 전달 빨라짐

### 시험 포인트
- HTTP/2 = frame, HOL 완화
- HTTP/3 = UDP 기반 발전형

---

# 3. E-mail / SMTP / IMAP (p49 ~ p57)

## p49 ~ p50 이메일의 3대 구성
1. **User Agent**
   - 메일 작성/읽기 도구
2. **Mail Server**
   - 메일함, 송신 대기열 보관
3. **SMTP**
   - 서버 간 메일 전송

### 손필기 표현
- **사람은 UA를 쓰고, 실제 전달은 서버끼리 처리**

### 시험 포인트
- 메일 시스템을 UA / mail server / SMTP 세 층으로 설명 가능해야 함

---

## p51 SMTP
- 메일은 무결성이 중요해서 **TCP 사용**
- 포트 번호: **25**
- 전송 서버가 client처럼 행동해 수신 서버로 보냄

### 3단계
1. handshaking
2. message transfer
3. closure

### 특징
- command / response
- ASCII text
- 상태 코드 + 구절
- 7-bit ASCII
- persistent connection 사용

### 손필기 표현
- **SMTP는 push 방식**

### 시험 포인트
- HTTP와 비교 시 SMTP는 push, HTTP는 pull

---

## p52 Alice -> Bob 시나리오
흐름:
`보내는 사람 -> 내 메일 서버 -> 상대 메일 서버 -> 받는 사람`

### 손필기 표현
- **메일은 사람끼리 바로 가는 게 아니라 각자 서버를 거친다**

### 시험 포인트
- UA에서 직접 상대 UA로 가는 구조가 아님

---

## p53 ~ p56 SMTP 상호작용과 메일 형식
- HELO, MAIL FROM, RCPT TO, DATA, QUIT 같은 명령 사용
- 메일 본문 자체는 header + blank line + body 구조

header 예시:
- To
- From
- Subject

### 주의
- 메일 메시지의 `From:` 과 SMTP 명령의 `MAIL FROM:` 은 다름

### 손필기 표현
- **SMTP는 메일을 전달하는 규칙**
- **메일 내용 형식은 또 따로 존재**

### 시험 포인트
- 프로토콜과 메시지 형식을 구분해서 이해

---

## p57 Mail Access Protocol
- **SMTP**: 보냄
- **IMAP**: 서버에 저장된 메일을 가져오고 관리
- **HTTP**: 웹메일 인터페이스(Gmail 등)

### 손필기 표현
- **SMTP = 송신**
- **IMAP = 수신/조회/관리**

### 시험 포인트
- SMTP와 IMAP의 역할 차이를 명확히 설명할 수 있어야 함

---

# 4. DNS (p59 ~ p72)

## p59 DNS란?
- 사람은 이름을 기억
- 네트워크는 IP 주소 사용
- 따라서 이름 <-> IP 변환 필요

### 정의
- **분산된 계층형 데이터베이스**
- 동시에 **응용 계층 프로토콜**

### 손필기 표현
- **DNS는 단순 전화번호부가 아니라, 계층형 분산 전화번호 시스템**

### 시험 포인트
- DNS가 왜 application layer protocol인지 말할 수 있어야 함

---

## p60 DNS를 왜 하나로 모으면 안 되나?
중앙집중형 DNS의 문제:
- SPOF
- 엄청난 트래픽
- 멀리 있는 중앙 DB
- 유지보수 어려움
- 확장성 부족

### DNS 서비스
- hostname -> IP
- host aliasing
- mail server aliasing
- load distribution

### 손필기 표현
- **질의가 너무 많아서 중앙집중형은 스케일이 안 된다**

### 시험 포인트
- “Why not centralize DNS?”는 단골 포인트

---

## p61 ~ p64 DNS 계층 구조
1. **Root DNS**
2. **TLD DNS**
3. **Authoritative DNS**
4. **Local DNS**

### Local DNS
- 학교/회사/ISP의 기본 DNS
- 계층에 엄격히 속하지는 않지만 실제 사용 시작점

||| DNS 계층 구조 그림
`Local -> Root -> TLD -> Authoritative`

### 손필기 표현
- **사용자는 보통 Local DNS부터 시작**

### 시험 포인트
- 각 계층의 역할을 말로 설명할 수 있어야 함

---

## p65 반복적 질의(Iterative Query)
- “나는 모르니 저 서버에 물어봐” 식
- 다음 서버 정보를 알려줌

### 손필기 표현
- **내가 DNS 모름 -> 쟤한테 물어봐**

## p66 재귀적 질의(Recursive Query)
- 요청받은 쪽이 대신 끝까지 찾아줌

### 손필기 표현
- **내가 대신 찾아줄게**

### 시험 포인트
- iterative와 recursive의 부담 주체 차이

---

## p67 캐싱과 TTL
- DNS 결과는 캐시됨
- TTL이 지나면 만료
- 캐시 덕분에 루트 서버 부담 감소
- 단, IP가 바뀌어도 TTL 전까지는 예전 정보가 남을 수 있음

### 손필기 표현
- **TTL = 살아 있는 시간, 끝나면 다시 질의**

### 시험 포인트
- 왜 DNS 결과가 best-effort인지 설명 가능해야 함

---

## p68 DNS 레코드
- **A**: hostname -> IP
- **NS**: authoritative name server
- **CNAME**: alias -> canonical name
- **MX**: mail server

### 손필기 표현
- **A는 주소, MX는 메일, CNAME은 별칭, NS는 권한 서버**

### 시험 포인트
- 레코드 타입을 구분하는 문제 대비

---

## p69 ~ p72 DNS 메시지, 등록, 보안
- query/reply 구조
- question / answer / authority / additional info
- 도메인 등록 시 registrar가 TLD에 NS, A 등을 반영
- 보안 이슈: DDoS, redirect attack, DNS poisoning
- 대응: DNSSEC

### 시험 포인트
- 세부 비트보다 **구조와 개념적 역할** 중심으로 기억
- DNS는 핵심 인프라라 공격 대상이 되기 쉬움

---

# 5. P2P / BitTorrent (p74 ~ p82)

## p74 P2P 구조
- 항상 켜져 있는 중앙 서버 없음
- peers가 직접 통신
- 요청도 하고 서비스도 제공

### 장점
- 참여자 증가 = 자원 증가
- self scalability

### 단점
- IP 변경
- churn
- 관리 복잡

### 손필기 표현
- **요청만 하는 게 아니라, 나도 남에게 제공한다**

### 시험 포인트
- client-server와 가장 크게 다른 지점은 “참여자가 자원도 가져온다”는 점

---

## p75 ~ p78 파일 배포 시간 관점
## Client-Server
- 서버 혼자 N명에게 보내야 하므로 부담 큼
- 사용자 수가 늘수록 선형적으로 무거워짐

## P2P
- 각 peer가 업로드에 참여
- 사용자 수가 늘면 서비스 능력도 같이 증가

### 손필기 표현
- **사람이 늘면 일도 늘지만, 도와줄 손도 늘어난다**

### 시험 포인트
- P2P가 왜 scalable한지 설명할 수 있어야 함

---

## p79 ~ p82 BitTorrent
- 파일을 작은 chunk로 분할
- tracker가 참여 peer 목록 관리
- peer끼리 chunk 교환

### 요청 전략
- **rarest first**
  - 가장 희귀한 조각부터 요청

### 전송 전략
- **tit-for-tat**
  - 나에게 잘 보내는 peer에게 우선 전송
- optimistic unchoke 존재

### 손필기 표현
- **희귀한 조각부터 받아야 전체가 골고루 퍼진다**
- **서로 잘 주는 상대와 우선 교환한다**

### 시험 포인트
- BitTorrent 핵심 키워드: chunk / tracker / rarest first / tit-for-tat

---

# 6. Streaming / DASH / CDN (p84 ~ p98)

## p84 왜 어려운가?
- 비디오 트래픽이 매우 큼
- 사용자 수가 많음
- 네트워크 환경이 제각각

### 결론
- 단일 서버만으로는 감당 어려움
- 분산된 응용 계층 인프라 필요

### 시험 포인트
- scale + heterogeneity 두 문제를 기억

---

## p85 ~ p86 비디오와 압축
- 비디오는 연속된 이미지
- 공간적 중복, 시간적 중복을 제거해 압축

### 개념
- **spatial coding**: 한 프레임 내부 중복 제거
- **temporal coding**: 이전 프레임과의 차이만 전송
- CBR / VBR 구분

### 손필기 표현
- **매번 전체를 다 보내지 않고, 겹치는 부분은 줄여서 보낸다**

---

## p87 ~ p90 스트리밍의 어려움과 버퍼링
문제:
- 대역폭 변화
- 지연/jitter
- 손실
- 재생은 끊기면 안 됨

해결:
- **client-side buffering**
- 조금 미리 받아서 재생

### 손필기 표현
- **미리 좀 받아놔야 네트워크가 흔들려도 영상이 안 끊긴다**

### 시험 포인트
- playout buffering의 목적 설명 가능해야 함

---

## p91 ~ p92 DASH
- 서버는 비디오를 chunk로 나누고 여러 비트레이트 버전 저장
- manifest file 제공
- 클라이언트는 현재 대역폭을 보고 적절한 화질 chunk 선택

### 손필기 표현
- **좋은 망이면 고화질, 안 좋으면 저화질**
- **상황 따라 화질을 바꾸는 적응형 스트리밍**

### 시험 포인트
- DASH = Dynamic Adaptive Streaming over HTTP
- adaptive인 이유를 말로 설명 가능해야 함

---

## p93 ~ p98 CDN
- 여러 지역에 콘텐츠 복사본 저장
- 사용자 가까운 서버가 전송

### 단일 mega-server 문제
- 병목
- 먼 거리
- 단일 장애점
- 확장성 부족

### 손필기 표현
- **콘텐츠를 사용자 가까이에 미리 놔두는 구조**
- **등록/계정 서버와 실제 영상 뿌리는 CDN 서버는 역할이 다를 수 있음**

### 시험 포인트
- CDN의 목적: scale, delay 감소
- DNS/CNAME과 연결되는 흐름도 같이 잡기

||| DASH + CDN 그림
- client
- manifest
- bitrate selection
- nearby CDN node

---

# 7. Socket Programming (p100 ~ p109)

## p100 소켓 프로그래밍 목표
- 클라이언트/서버 앱이 실제로 소켓으로 통신하는 구조 이해

### 시험 포인트
- 개념적으로 “앱이 어떻게 네트워크를 쓰는지”를 concrete하게 보여주는 부분

---

## p101 두 가지 소켓
- UDP socket = unreliable datagram
- TCP socket = reliable byte stream

### 손필기 표현
- **UDP는 편지 묶음, TCP는 연결된 파이프**

---

## p102 ~ p105 UDP 소켓
### 특징
- 연결 설정 없음
- handshake 없음
- 보낼 때마다 목적지 IP/포트 명시
- 순서 보장 X
- 손실 가능

### 응용 계층 관점
- UDP는 **datagram 단위** 전달

### 기억 포인트
- `sendto()`
- `recvfrom()`
- 상대 주소 정보가 같이 다님

||| UDP client/server 그림
- client socket 생성
- sendto()
- server recvfrom()
- server sendto()
- client recvfrom()

### 시험 포인트
- UDP는 connectionless라서 패킷마다 목적지 정보가 필요

---

## p106 ~ p109 TCP 소켓
### 특징
- 먼저 연결 설정 필요
- 서버는 welcoming socket 준비
- 클라이언트가 접속하면 서버는 **새 연결 소켓** 생성
- 신뢰적, 순서 보장된 byte stream

### 기억 포인트
서버:
- bind
- listen
- accept

클라이언트:
- connect

이후:
- send / recv
- close

### 손필기 표현
- **서버는 손님을 기다리는 문 하나와, 손님별 대화방 소켓을 따로 둔다**

||| TCP client/server 그림
- server socket 생성 -> listen -> accept
- client socket 생성 -> connect
- request / response
- close

### 시험 포인트
- TCP 서버는 접속을 받으면 per-client socket이 새로 생긴다는 점
- UDP와 달리 주소를 매번 직접 붙이지 않아도 된다는 점

---

# 8. 헷갈리는 비교 한 번에 정리

## 8-1 TCP vs UDP
| 항목 | TCP | UDP |
|---|---|---|
| 연결 설정 | O | X |
| 신뢰성 | O | X |
| 순서 보장 | O | X |
| flow control | O | X |
| congestion control | O | X |
| 사용 감각 | 정확성 우선 | 지연/단순성 우선 |

---

## 8-2 Non-persistent vs Persistent HTTP
| 항목 | Non-persistent | Persistent |
|---|---|---|
| 연결 수 | 객체마다 별도 | 하나로 여러 객체 |
| RTT 부담 | 큼 | 감소 |
| 효율 | 낮음 | 높음 |

---

## 8-3 HTTP vs SMTP
| 항목 | HTTP | SMTP |
|---|---|---|
| 기본 동작 | pull | push |
| 역할 | 웹 객체 요청/응답 | 메일 전송 |
| 대표 특징 | stateless | 서버 간 전달 |

---

## 8-4 SMTP vs IMAP
| 항목 | SMTP | IMAP |
|---|---|---|
| 역할 | 메일 보냄 | 메일 가져옴/관리 |
| 방향 | 송신 중심 | 수신/조회 중심 |
| 핵심 감각 | push | 서버 저장 메일 접근 |

---

## 8-5 Iterative vs Recursive DNS Query
| 항목 | Iterative | Recursive |
|---|---|---|
| 느낌 | 저 서버에 물어봐 | 내가 대신 찾아줄게 |
| 부담 | 요청자 쪽 | 중간 서버 쪽 |

---

## 8-6 Web Cache vs Cookie
| 항목 | Web Cache | Cookie |
|---|---|---|
| 목적 | 응답 속도/트래픽 최적화 | 상태 유지 |
| 저장 성격 | 자주 쓰는 객체 | 사용자 식별 정보 |
| 핵심 질문 | 더 빨리 줄 수 있나? | 누구인지 기억할 수 있나? |

---

# 9. 문제 풀이용으로 꼭 떠올릴 질문

1. **이 앱은 유실을 허용하는가?**
   - 허용 X -> TCP 후보
   - 허용 O + 지연 중요 -> UDP 후보

2. **왜 IP만으로 부족한가?**
   - 같은 호스트 안 여러 프로세스 때문

3. **왜 HTTP에 쿠키가 필요한가?**
   - HTTP가 stateless라 상태 유지가 안 되기 때문

4. **왜 Non-persistent HTTP가 비효율적인가?**
   - 객체마다 연결, 2RTT/객체, 오버헤드 큼

5. **왜 DNS를 중앙집중형으로 못 만드는가?**
   - SPOF, 트래픽, 거리, 유지보수, 확장성 문제

6. **왜 P2P가 scalable한가?**
   - 사용자 증가가 자원 증가로도 이어짐

7. **왜 DASH가 adaptive인가?**
   - 현재 대역폭에 맞춰 chunk bitrate를 바꾸기 때문

8. **왜 HTTP/2가 체감상 빠른가?**
   - 프레임 단위 전송으로 HOL blocking 완화

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
- 웹 캐시 = 가까운 곳에서 응답
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

---

# 11. 직접 그리면 좋은 그림
1. 소켓 통신 구조
2. Non-persistent vs Persistent HTTP
3. 쿠키 동작 흐름
4. Web cache 흐름
5. DNS 계층 구조
6. Iterative vs Recursive query
7. SMTP 전달 구조
8. BitTorrent chunk 교환
9. DASH + CDN 구조
10. UDP client/server 흐름
11. TCP client/server 흐름
12. HTTP/2 HOL blocking 완화

---

# 12. 마지막 체크리스트
- 왜 IP만으로는 프로세스를 식별할 수 없는가?
- 앱이 요구하는 전송 서비스 4가지는 무엇인가?
- TCP와 UDP를 앱 관점에서 어떻게 구분하는가?
- 왜 HTTP는 stateless인데 쿠키가 필요한가?
- Non-persistent HTTP가 왜 비효율적인가?
- Conditional GET의 목적은 무엇인가?
- SMTP와 IMAP의 역할 차이는?
- DNS를 중앙집중형으로 만들면 왜 안 되는가?
- Iterative query와 Recursive query 차이는?
- P2P가 client-server보다 유리한 이유는?
- BitTorrent의 rarest first와 tit-for-tat은 왜 필요한가?
- DASH가 adaptive streaming이라 불리는 이유는?
- CDN은 왜 필요한가?
- UDP 소켓과 TCP 소켓의 차이는?

