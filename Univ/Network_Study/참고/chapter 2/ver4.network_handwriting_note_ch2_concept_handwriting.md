# 네트워크 손필기 전사용 노트 v4
## Chapter 2(Application Layer) 개념별 정리 + 손필기 반영판
부제: **ver1의 개념별 구조**를 중심으로, **ver2의 손필기 표현과 수업식 강조 포인트**를 합친 전사용 버전

---

## 이 버전의 기준
- **개념별로 묶어서 정리**하되, 각 개념이 **교안 몇 페이지 범위인지** 알 수 있게 구성
- 페이지 순서에 끌려가지 않고, **시험에서 꺼내 쓰기 쉬운 개념 축** 중심
- 손필기 표현, 수업 때 강조된 감각, 시험 문제 풀이용 포인트를 같이 반영
- `|||` 는 **직접 그려 넣을 그림 자리**
- 문장은 짧게, **정의 -> 왜 필요한가 -> 비교/시험 포인트** 순서로 적기

---

# 0. 시험 전에 먼저 외울 핵심 문장 16개

1. **응용 계층 프로토콜은 메시지 타입, 문법, 의미, 송수신 규칙을 정의한다.**
2. **서로 다른 호스트의 프로세스는 메시지 교환으로 통신한다.**
3. **소켓은 응용 계층과 전송 계층 사이의 문이다.**
4. **프로세스 식별은 IP만으로 부족하고 IP + Port가 필요하다.**
5. **앱이 전송 계층에 요구하는 핵심은 무결성, 보안, 시간, 처리량이다.**
6. **TCP는 신뢰적·연결형이고, UDP는 단순·비연결형이다.**
7. **기본 TCP/UDP 소켓은 암호화가 없어서 TLS가 필요하다.**
8. **HTTP는 TCP를 쓰지만 기본적으로 stateless다.**
9. **Non-persistent HTTP는 객체마다 TCP 연결을 새로 만든다.**
10. **Persistent HTTP는 한 연결로 여러 객체를 보내 RTT와 오버헤드를 줄인다.**
11. **쿠키는 stateless한 HTTP 위에서 상태를 유지하는 장치다.**
12. **웹 캐시는 가까운 곳에서 응답해 지연과 트래픽을 줄인다.**
13. **SMTP는 메일을 보내는 프로토콜이고, IMAP은 서버에 저장된 메일을 가져오는 프로토콜이다.**
14. **DNS는 이름을 IP로 바꾸는 분산 계층형 데이터베이스다.**
15. **P2P의 핵심 장점은 참여자가 늘면 자원도 같이 늘어난다는 점이다.**
16. **DASH는 현재 대역폭에 맞춰 화질을 고르는 적응형 스트리밍이다.**

---

# 1. 응용 계층 큰 그림 (p8 ~ p16)

## 1-1. 네트워크 앱이란?
- 서로 다른 **종단 시스템(end system)** 에서 실행되는 프로그램들이
- 네트워크를 통해 **메시지**를 주고받는 것
- 네트워크 코어 장비가 아니라 **end system에서 앱이 실행**됨

### 손필기 표현
- **프로그램은 말을 하고, 네트워크는 그 말을 다른 호스트로 보낸다.**
- **앱은 끝단에서 돌고, 중간 라우터는 앱을 실행하지 않는다.**

### 시험 포인트
- 왜 앱 개발/배포가 빠른가?
  - 코어 장비를 바꾸지 않고 end system만 바꾸면 되기 때문

---

## 1-2. 프로세스와 소켓
- 프로세스 = 호스트 안에서 실행 중인 프로그램
- 서로 다른 호스트의 프로세스는 **메시지 교환**으로 통신
- 소켓 = 응용이 메시지를 넣고 꺼내는 **출입문**

### 손필기 표현
- **소켓은 문, 메시지는 문을 통과하는 짐**
- **프로세스가 말하고, 소켓이 내보내고, 전송 계층이 운반한다**

||| 소켓 그림
`프로세스 -> 소켓 -> 전송/네트워크 -> 소켓 -> 프로세스`

---

## 1-3. 프로세스 식별: IP + Port
- IP 주소만으로는 부족
- 같은 호스트 안에도 여러 프로세스가 동시에 존재 가능
- 따라서 식별자 = **IP 주소 + Port 번호**

### 손필기 표현
- **집 주소(IP)만으로는 부족, 누구 방(Port)인지까지 알아야 한다**
- **메시지를 받은 뒤 누구에게 줄지 분류하려면 포트 번호가 필요하다**

### 예시
- HTTP 서버: `:80`
- Mail 서버: `:25`

### 시험 포인트
- 왜 IP만으로는 프로세스를 식별할 수 없는가?
  - 같은 호스트 안 여러 프로세스를 구분할 수 없기 때문

---

## 1-4. 응용 계층 프로토콜이 정의하는 것
응용 계층 프로토콜은 아래 4가지를 정한다.

1. **메시지 타입**
   - request / response
2. **메시지 문법(syntax)**
   - 필드가 어떻게 생겼는가
3. **메시지 의미(semantics)**
   - 각 필드가 무엇을 뜻하는가
4. **송수신 규칙**
   - 언제 보내고 어떻게 응답하는가

### 프로토콜 종류
- **Open protocol**
  - RFC 공개
  - 예: HTTP, SMTP
- **Proprietary protocol**
  - 특정 기업/서비스 전용

### 손필기 표현
- **프로토콜은 그냥 약속이 아니라, 메시지를 어떻게 생기게 하고 어떻게 주고받을지까지 정하는 규칙**

---

# 2. 앱이 전송 계층에 요구하는 것 (p12 ~ p16)

## 2-1. 네 가지 핵심 요구
1. **data integrity**
2. **security**
3. **timing**
4. **throughput**

### 손필기 표현
- **앱이 보는 건 결국 네 가지: 안 깨지나 / 안전하나 / 늦지 않나 / 충분히 많이 가나**

### 감각적으로 연결
- 파일 전송: integrity 중요
- 로그인/결제: security 중요
- 실시간 통화/게임: timing 중요
- 스트리밍 영상: throughput 중요

---

## 2-2. TCP vs UDP
## TCP
- 신뢰적 전송
- 연결형
- flow control
- congestion control
- 제공하지 않는 것:
  - timing guarantee
  - minimum throughput guarantee
  - 자체 보안

## UDP
- 비연결형
- 신뢰성 보장 X
- 순서 보장 X
- flow control X
- congestion control X
- throughput guarantee X
- 보안 X
- 연결 설정 X

### 손필기 표현
- **TCP = 정확하게, 절차 있게**
- **UDP = 빨리, 단순하게**

### 왜 UDP가 존재하나?
- TCP는 절차가 많고 무거움
- 손실을 조금 허용하고 빠르게 보내야 하는 앱도 있음

### 시험 포인트
- “유실되면 안 됨” -> TCP 후보
- “조금 유실돼도 되지만 지연이 중요” -> UDP 후보

---

## 2-3. TLS
- 기본 TCP/UDP 소켓은 암호화 없음
- 비밀번호도 평문으로 오갈 수 있음
- TLS는 TCP 위에서 보안을 추가
  - 암호화
  - 데이터 무결성
  - 종단 인증

### 손필기 표현
- **HTTPS = HTTP + TLS**
- **TCP가 안전한 것이 아니라, 위에 TLS를 씌우는 것**

### 시험 포인트
- TCP의 reliable과 secure는 다른 개념

---

# 3. Web과 HTTP (p18 ~ p47)

## 3-1. 웹 페이지와 객체 (p18)
- 웹 페이지는 여러 **객체(object)** 로 구성
- 객체는 HTML, 이미지, 오디오 등
- 하나의 페이지 = base HTML + 여러 참조 객체
- 각 객체는 URL로 지정 가능

### 손필기 표현
- **웹 페이지 1개를 여는 건, 실제로 여러 파일을 가져오는 것**

---

## 3-2. HTTP의 본질 (p19 ~ p20)
- HTTP = Web의 응용 계층 프로토콜
- client/server 모델
  - client = browser
  - server = web server
- TCP 사용
- 기본 포트 = 80
- 중요한 특징 = **stateless**

### stateless란?
- 서버가 이전 요청을 기억하지 않음
- 모든 요청이 독립적임

### 손필기 표현
- **뒤로 가기를 눌러도, 예전 상태를 기억하는 게 아니라 다시 요청해서 가져오는 구조**
- **상태 유지가 어렵다 = 프로토콜은 단순하지만 사용자 상태를 따로 저장해야 한다**

### 시험 포인트
- HTTP는 application layer protocol
- transport는 TCP 사용
- stateless의 장단점 설명 가능해야 함

---

## 3-3. Non-persistent HTTP vs Persistent HTTP (p21 ~ p25)

## Non-persistent HTTP
- TCP 연결 열기
- 객체 1개 보내기
- TCP 연결 닫기

## Persistent HTTP
- TCP 연결 1번 열기
- 같은 연결로 여러 객체 전송
- RTT와 연결 오버헤드 감소

### 손필기 표현
- **비연결형 = 객체마다 새 연결**
- **연결형 = 한번 열고 쭉 보냄**

### 비연결형 예시
- HTML 1개 받기 위해 연결 1번
- 이미지 10개면 추가 10번 연결
- 객체마다 생성/종료 반복

### 응답 시간
- 비연결형 HTTP에서 객체 1개당 대략:
- **2 RTT + 파일 전송 시간**

이유:
- 1 RTT = TCP 연결 성립
- 1 RTT = HTTP 요청 + 첫 응답 바이트
- 이후 = 파일 전송 시간

||| Non-persistent vs Persistent 그림
- 왼쪽: HTML 1개 + 이미지 10개 = 연결 반복
- 오른쪽: 하나의 연결로 여러 객체 전송

### 시험 포인트
- 왜 Non-persistent HTTP가 비효율적인가?
  - 객체마다 연결, 2RTT/객체, 연결 오버헤드 큼

---

## 3-4. HTTP 메시지 구조 (p26 ~ p29)
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

### 상태 코드
- **200 OK**
- **301 Moved Permanently**
- **400 Bad Request**
- **404 Not Found**
- **505 HTTP Version Not Supported**

### 손필기 표현
- **HTTP 요청/응답은 헤더가 매우 중요**
- **Date 헤더는 시간 기준을 맞추는 데도 의미가 있음**
- **301 = 영구 이동(리다이렉트), 404 = 서버에 문서 없음**

||| HTTP general format 그림
- request line / headers / blank line / body
- status line / headers / blank line / data

### 시험 포인트
- request와 response의 큰 구조 구분
- POST는 body, GET은 URL에 데이터가 붙을 수 있음

---

## 3-5. 쿠키(Cookie) (p31 ~ p35)
### 왜 필요한가?
- HTTP는 stateless
- 그런데 로그인, 장바구니, 세션 유지에는 상태 정보가 필요

### 쿠키 관련 4요소
1. 응답 메시지의 set-cookie
2. 다음 요청 메시지의 cookie
3. 브라우저의 cookie 파일
4. 서버의 백엔드 DB

### 저장 위치
- 사용자의 웹 브라우저

### 용도
- 권한 유지
- 장바구니
- 유저 세션 정보
- 추천

### 손필기 표현
- **쿠키로 재로그인하는 게 아니라, 쿠키를 통해 이미 인증된 상태를 식별/권한 부여하는 것**
- **쿠키는 stateless HTTP 위에 상태를 얹는 장치**

### 프라이버시 메모
- 추적 문제 존재
- 편리하지만 사생활 측면 주의

||| Cookie 흐름 그림
- 첫 방문 -> 서버가 ID 생성 -> 브라우저 저장
- 다음 요청 -> cookie ID 포함 -> 서버 DB와 대조

### 시험 포인트
- 왜 HTTP에 쿠키가 필요한가?
  - HTTP가 stateless이기 때문

---

## 3-6. 웹 캐시(Web Cache)와 Conditional GET (p36 ~ p42)

## 웹 캐시
- 브라우저가 캐시에 요청
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

## Conditional GET
- 목표: 최신 버전이 이미 있으면 객체를 다시 안 보내기
- 요청 헤더: `If-Modified-Since`
- 응답:
  - 안 바뀜 -> `304 Not Modified`
  - 바뀜 -> `200 OK` + 새 데이터

### 손필기 표현
- **안 바뀐 파일은 안 보내서 시간과 트래픽을 아낀다**

### 시험 포인트
- Web cache의 목적과 Cookie의 목적은 다름
- 304 Not Modified 의미 기억

---

## 3-7. HTTP/2와 HTTP/3 (p43 ~ p47)
## HTTP/1.1 문제
- 요청 순서대로 처리하는 성격(FCFS)
- 앞의 큰 객체 때문에 뒤의 작은 객체가 기다릴 수 있음
- 이것이 **HOL blocking**

## HTTP/2
- 객체를 **프레임(frame)** 단위로 나눔
- 전송 순서를 더 유연하게 조절
- HOL blocking 완화

## HTTP/3
- UDP 기반 방향으로 발전
- 보안과 지연 측면 개선
- QUIC과 연결해서 이해

### 손필기 표현
- **왜 하지? -> 유저 편의성**
- **프레임별로 잘라서 보내 작은 객체가 덜 묶이게 한다**

||| HOL blocking 완화 그림
- HTTP/1.1: 큰 객체 뒤에 작은 객체 대기
- HTTP/2: frame interleaving

### 시험 포인트
- HTTP/2 = frame + HOL 완화
- HTTP/3 = UDP 기반 발전형

---

# 4. E-mail / SMTP / IMAP (p49 ~ p57)

## 4-1. 이메일의 큰 구조
세 가지 핵심 구성:
1. **User Agent**
2. **Mail Server**
3. **SMTP**

### 손필기 표현
- **사람은 UA를 쓰고, 실제 전달은 서버끼리 처리**

---

## 4-2. SMTP (p51 ~ p56)
- 메일은 무결성이 중요해서 TCP 사용
- 포트 번호 = 25
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
- **push 방식**

### Alice -> Bob 흐름
`보내는 사람 -> 내 메일 서버 -> 상대 메일 서버 -> 받는 사람`

### 메일 형식
- To
- From
- Subject
- blank line
- body

### 손필기 표현
- **메일은 사람끼리 바로 가는 게 아니라 각자 메일 서버를 거친다**
- **SMTP는 메일을 전달하는 규칙, 메일 내용 형식은 또 따로 존재**

### 주의
- 메일 메시지의 `From:`과 SMTP 명령의 `MAIL FROM:`은 다름

---

## 4-3. IMAP / HTTP (p57)
- **SMTP** = 보냄
- **IMAP** = 서버에 저장된 메일을 가져오고 관리
- **HTTP** = 웹메일 인터페이스

### 손필기 표현
- **SMTP = 송신**
- **IMAP = 수신/조회/관리**
- **HTTP = 웹에서 메일에 접근하는 창구**

### 시험 포인트
- SMTP와 IMAP 역할 구분은 자주 나오는 비교축

---

# 5. DNS (p59 ~ p72)

## 5-1. DNS란?
- 사람은 이름을 기억
- 네트워크는 IP 주소 사용
- 따라서 이름 <-> IP 변환 필요

### 정의
- **분산된 계층형 데이터베이스**
- 동시에 **응용 계층 프로토콜**

### 손필기 표현
- **DNS는 단순 전화번호부가 아니라, 계층형 분산 전화번호 시스템**

---

## 5-2. DNS를 중앙집중형으로 못 만드는 이유 (p60)
- SPOF
- 엄청난 트래픽
- 멀리 있는 중앙 DB
- 유지보수 어려움
- 확장성 부족

### 제공 서비스
- hostname -> IP
- host aliasing
- mail server aliasing
- load distribution

### 손필기 표현
- **질의가 너무 많아서 중앙집중형은 스케일이 안 된다**

### 시험 포인트
- “Why not centralize DNS?”는 단골 질문

---

## 5-3. DNS 계층 구조 (p61 ~ p64)
1. **Root DNS**
2. **TLD DNS**
3. **Authoritative DNS**
4. **Local DNS**

### Local DNS
- 학교/회사/ISP의 기본 DNS
- 계층에 엄격히 속하지는 않지만 실제 사용 시작점

||| DNS hierarchy 그림
`Local -> Root -> TLD -> Authoritative`

### 손필기 표현
- **사용자는 보통 Local DNS부터 시작한다**

---

## 5-4. Iterative Query vs Recursive Query (p65 ~ p66)

## Iterative Query
- “나는 모르니 저 서버에 물어봐” 식
- 다음 서버 정보를 알려줌

## Recursive Query
- 요청받은 쪽이 대신 끝까지 찾아줌

### 손필기 표현
- **Iterative = 쟤한테 물어봐**
- **Recursive = 내가 대신 찾아줄게**

### 시험 포인트
- 부담이 누구에게 가는지 구분할 수 있어야 함

---

## 5-5. 캐싱과 TTL (p67)
- DNS 결과는 캐시됨
- TTL이 지나면 만료
- 캐시 덕분에 루트 서버 부담 감소
- 단, IP 변경이 즉시 전 세계 반영되지는 않을 수 있음

### 손필기 표현
- **TTL = 살아 있는 시간, 끝나면 다시 질의**
- **캐시 때문에 빠르지만, 약간 오래된 정보가 남을 수도 있다**

---

## 5-6. DNS 레코드와 보안 (p68 ~ p72)
### 레코드
- **A** = hostname -> IP
- **NS** = authoritative name server
- **CNAME** = alias -> canonical name
- **MX** = mail server

### 보안 이슈
- DDoS
- redirect attack
- DNS poisoning
- 대응: DNSSEC

### 손필기 표현
- **A는 주소, MX는 메일, CNAME은 별칭, NS는 권한 서버**
- **DNS는 핵심 인프라라 공격 대상이 되기 쉽다**

### 시험 포인트
- 레코드 타입 구분 문제 대비

---

# 6. P2P / BitTorrent (p74 ~ p82)

## 6-1. P2P 구조
- 항상 켜져 있는 중앙 서버 없음
- peers가 직접 통신
- 요청도 하고, 서비스도 제공

### 장점
- 참여자 증가 = 자원 증가
- self scalability

### 단점
- IP 변경
- churn
- 관리 복잡

### 손필기 표현
- **요청만 하는 게 아니라, 나도 남에게 제공한다**

---

## 6-2. 파일 배포 시간 관점
## Client-Server
- 서버 혼자 N명에게 파일 복사
- N이 커질수록 부담 증가

## P2P
- 각 peer도 업로드에 참여
- 사용자 증가가 자원 증가로 이어짐

### 손필기 표현
- **사람이 늘면 일도 늘지만, 도와줄 손도 늘어난다**

### 시험 포인트
- 왜 P2P가 scalable한지 설명 가능해야 함

---

## 6-3. BitTorrent
- 파일을 작은 chunk로 분할
- tracker가 참여 peer 목록 관리
- peer끼리 chunk 교환

### 요청 전략
- **rarest first**

### 전송 전략
- **tit-for-tat**
- optimistic unchoke 존재

### 손필기 표현
- **희귀한 조각부터 받아야 전체가 골고루 퍼진다**
- **서로 잘 주는 상대와 우선 교환한다**

### 시험 포인트
- BitTorrent 핵심 키워드: chunk / tracker / rarest first / tit-for-tat

---

# 7. Streaming / DASH / CDN (p84 ~ p98)

## 7-1. 스트리밍이 어려운 이유
- 비디오 트래픽 큼
- 사용자 수 많음
- 네트워크 환경이 제각각

### 결론
- 단일 서버만으로는 감당 어려움
- 분산된 응용 계층 인프라 필요

### 시험 포인트
- scale + heterogeneity 두 문제 기억

---

## 7-2. 비디오 압축
- 비디오는 연속된 이미지
- 공간적 중복, 시간적 중복 제거해 압축

### 개념
- **spatial coding**
- **temporal coding**
- CBR / VBR

### 손필기 표현
- **매번 전체를 다 보내지 않고, 겹치는 부분은 줄여서 보낸다**

---

## 7-3. 버퍼링과 DASH
### 버퍼링
- 대역폭 변화, 지연/jitter, 손실에 대비
- 클라이언트가 조금 미리 받아 저장 후 재생

### DASH
- 서버: chunk + 여러 비트레이트 저장
- 클라이언트: 현재 대역폭 보고 적절한 화질 chunk 선택

### 손필기 표현
- **미리 좀 받아놔야 네트워크가 흔들려도 영상이 안 끊긴다**
- **좋은 망이면 고화질, 안 좋으면 저화질**

### 시험 포인트
- 왜 adaptive streaming인가?
  - 현재 대역폭에 따라 bitrate를 바꾸기 때문

---

## 7-4. CDN
- 여러 지역에 콘텐츠 복사본 저장
- 사용자 가까운 서버가 전송

### 단일 mega-server 문제
- 병목
- 먼 거리
- 단일 장애점
- 확장성 부족

### 손필기 표현
- **콘텐츠를 사용자 가까이에 미리 놔두는 구조**
- **등록/계정 서버와 실제 영상 뿌리는 CDN 서버는 역할이 다를 수 있다**

||| DASH + CDN 그림
- client
- manifest
- bitrate selection
- nearby CDN node

### 시험 포인트
- CDN의 목적: scale, delay 감소

---

# 8. Socket Programming (p100 ~ p109)

## 8-1. 소켓 프로그래밍 목표
- 클라이언트/서버 앱이 실제로 소켓으로 통신하는 구조 이해

---

## 8-2. UDP 소켓 (p102 ~ p105)
### 특징
- 연결 설정 없음
- handshake 없음
- 보낼 때마다 목적지 IP/포트 명시
- 순서 보장 X
- 손실 가능
- datagram 단위 전달

### 기억 포인트
- `sendto()`
- `recvfrom()`

### 손필기 표현
- **UDP는 편지 묶음**
- **주소를 매번 붙여서 보낸다**

||| UDP client/server 그림
- client socket 생성
- sendto()
- server recvfrom()
- server sendto()
- client recvfrom()

### 시험 포인트
- 왜 UDP는 connectionless라고 하는가?
  - 연결 상태 없이 패킷마다 독립적으로 처리하기 때문

---

## 8-3. TCP 소켓 (p106 ~ p109)
### 특징
- 먼저 연결 설정 필요
- 서버는 welcoming socket 준비
- 클라이언트가 접속하면 **그 클라이언트 전용 새 소켓** 생성
- reliable, byte stream

### 기억 포인트
서버:
- bind
- listen
- accept

클라이언트:
- connect

이후:
- send / recv / close

### 손필기 표현
- **TCP는 연결된 파이프**
- **서버는 손님을 기다리는 문 하나와, 손님별 대화방 소켓을 따로 둔다**

||| TCP client/server 그림
- server socket 생성 -> listen -> accept
- client socket 생성 -> connect
- request / response
- close

### 시험 포인트
- UDP와 달리 per-client socket 개념이 중요

---

# 9. 헷갈리는 비교축 정리

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

# 10. 문제 풀이용 체크 질문

1. 이 앱은 유실을 허용하는가?
2. 왜 IP만으로는 프로세스를 식별할 수 없는가?
3. 왜 HTTP는 stateless인데 쿠키가 필요한가?
4. 왜 Non-persistent HTTP가 비효율적인가?
5. 왜 DNS를 중앙집중형으로 만들 수 없는가?
6. 왜 P2P가 scalable한가?
7. 왜 DASH가 adaptive streaming이라 불리는가?
8. 왜 HTTP/2가 체감상 더 유리한가?

---

# 11. 손필기 최종 압축본

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
- Cookie = 상태 유지
- Web Cache = 가까운 곳에서 응답
- Conditional GET = 안 바뀌면 다시 안 보냄
- HTTP/2 = frame으로 HOL 완화
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
- buffer로 jitter 완충
- DASH = 상황 따라 화질 선택
- CDN = 가까운 서버에서 전송

## Socket
- UDP = sendto / recvfrom / datagram
- TCP = connect / accept / byte stream

---

# 12. 직접 그리면 좋은 그림
1. 소켓 통신 구조
2. Non-persistent vs Persistent HTTP
3. Cookie 동작 흐름
4. Web Cache 흐름
5. DNS 계층 구조
6. Iterative vs Recursive query
7. SMTP 전달 구조
8. BitTorrent chunk 교환
9. DASH + CDN 구조
10. UDP client/server 흐름
11. TCP client/server 흐름
12. HTTP/2 HOL blocking 완화

---

# 13. 마지막 체크리스트
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

